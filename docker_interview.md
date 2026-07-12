# Docker Interview Preparation Guide

A comprehensive, scenario-driven guide covering Docker fundamentals, advanced concepts, and real-world troubleshooting. Each answer includes a **use case**, **examples/commands**, and **interview tips**.

---

## Table of Contents

1. [Part A — Troubleshooting & Common Errors](#part-a--troubleshooting--common-errors)
2. [Part B — Advanced Docker Concepts](#part-b--advanced-docker-concepts)
3. [Part C — Docker Fundamentals](#part-c--docker-fundamentals)
4. [Quick Reference Cheat Sheet](#quick-reference-cheat-sheet)

---

# Part A — Troubleshooting & Common Errors

> These are the errors you *actually* hit in production. Interviewers love these because they reveal whether you have hands-on operational experience.

---

## 🔹 Question 1: What does the 'Container is Unhealthy' status indicate, and how do you troubleshoot it?

**Answer:**
The `unhealthy` status means the container is running, but its **HEALTHCHECK** command has failed the configured number of consecutive times (`--retries`). Docker marks the container as `unhealthy`, and orchestrators like Swarm/Kubernetes may restart or stop routing traffic to it.

**Use case:**
You deploy a web API with a healthcheck hitting `/health`. The app process is alive, but the database connection pool is exhausted, so `/health` returns HTTP 500. Docker flips the container to `unhealthy` → your load balancer stops sending traffic → prevents user-facing errors.

**How to troubleshoot:**
```bash
# 1. Check the health status and last probe output
docker inspect --format='{{json .State.Health}}' my_container | jq

# 2. See the last few healthcheck logs
docker inspect my_container | grep -A 20 "Health"

# 3. Run the healthcheck command manually inside the container
docker exec -it my_container curl -f http://localhost:8080/health

# 4. Check application logs
docker logs --tail 100 my_container
```

**Common root causes:**
- Healthcheck endpoint too strict (checks downstream deps that are temporarily slow).
- `--start-period` too short (app needs more time to boot).
- Wrong port/path in the `HEALTHCHECK` instruction.

**Fix example (tuning the healthcheck):**
```dockerfile
HEALTHCHECK --interval=30s --timeout=5s --start-period=40s --retries=3 \
  CMD curl -f http://localhost:8080/health || exit 1
```

**Interview tip:** Emphasize `--start-period` — it's the #1 cause of false "unhealthy" on slow-starting apps (JVM, migrations).

---

## 🔹 Question 2: What causes the 'Cannot Connect to the Docker Daemon' error, and how do you resolve it?

**Answer:**
This error (`Cannot connect to the Docker daemon at unix:///var/run/docker.sock. Is the docker daemon running?`) means the Docker CLI can't reach the `dockerd` background service.

**Common causes & fixes:**

| Cause | Diagnosis | Fix |
|-------|-----------|-----|
| Daemon not running | `sudo systemctl status docker` | `sudo systemctl start docker` |
| User not in `docker` group | `groups $USER` | `sudo usermod -aG docker $USER` then re-login |
| Wrong `DOCKER_HOST` env var | `echo $DOCKER_HOST` | `unset DOCKER_HOST` |
| Socket permissions | `ls -l /var/run/docker.sock` | `sudo chmod 660 /var/run/docker.sock` |

**Use case:**
A new engineer runs `docker ps` on a fresh Linux VM and gets "permission denied". Adding them to the `docker` group fixes it without needing `sudo` every time.

```bash
sudo usermod -aG docker $USER
newgrp docker   # apply group without full logout
docker ps       # now works
```

**Interview tip:** Mention the security implication — being in the `docker` group is effectively **root access** on the host, so it should be granted carefully.

---

## 🔹 Question 3: How do you fix the 'Image Pull Backoff' error?

**Answer:**
`ImagePullBackOff` (Kubernetes term; Docker equivalent is a failed `docker pull`) means the runtime repeatedly failed to pull an image and is now backing off (waiting longer between retries).

**Common causes & fixes:**
1. **Wrong image name/tag** → verify: `docker pull myrepo/app:v1.2.3`
2. **Private registry auth missing** → create a pull secret / `docker login`.
3. **Rate limiting (Docker Hub)** → authenticate or use a mirror/registry cache.
4. **Network/DNS issue** → `curl -v https://registry-1.docker.io/v2/`

**Use case (Kubernetes private registry):**
```bash
kubectl create secret docker-registry regcred \
  --docker-server=myregistry.example.com \
  --docker-username=deploy \
  --docker-password=$TOKEN
```
```yaml
spec:
  imagePullSecrets:
    - name: regcred
  containers:
    - name: app
      image: myregistry.example.com/app:1.0.0
```

**Diagnosis:**
```bash
kubectl describe pod my-pod   # look at Events section for exact reason
```

**Interview tip:** Distinguish `ErrImagePull` (first failure) from `ImagePullBackOff` (retrying with backoff). Docker Hub anonymous rate limits (100–200 pulls/6h) are a classic real-world cause.

---

## 🔹 Question 4: What causes the 'No space left on device' error, and how do you resolve it?

**Answer:**
The Docker host's disk (or the partition holding `/var/lib/docker`) is full — usually from accumulated images, stopped containers, dangling volumes, build cache, or logs.

**Diagnose:**
```bash
df -h                     # host disk usage
docker system df          # Docker's own usage breakdown
docker system df -v       # verbose: per-image/volume detail
```

**Resolve (least → most aggressive):**
```bash
# Remove dangling images, stopped containers, unused networks & build cache
docker system prune

# Also remove unused volumes (DANGER: deletes data not attached to a container)
docker system prune --volumes

# Target specific resources
docker image prune -a           # all unused images
docker container prune          # stopped containers
docker builder prune            # build cache
```

**Use case (log explosion):**
A misconfigured app logs to stdout at high volume. The JSON log file at `/var/lib/docker/containers/<id>/<id>-json.log` grows to GBs. Fix by capping log size:
```json
// /etc/docker/daemon.json
{
  "log-driver": "json-file",
  "log-opts": { "max-size": "10m", "max-file": "3" }
}
```
Then `sudo systemctl restart docker`.

**Interview tip:** Always mention log rotation — unbounded container logs are a top cause in long-running production hosts. Also note that `prune --volumes` can delete real data.

---

## 🔹 Question 5: How do you resolve a 'Permission Denied' error when accessing files inside a container?

**Answer:**
Usually a mismatch between the **UID/GID** the container process runs as and the ownership/permissions of the file — very common with bind mounts and volumes.

**Common scenarios & fixes:**

1. **Bind mount owned by host root, app runs as non-root user:**
```bash
# Check ownership on host
ls -ln ./data
# Fix: match the container's UID (e.g., 1000)
sudo chown -R 1000:1000 ./data
```

2. **Set the user in Dockerfile and align ownership:**
```dockerfile
RUN addgroup --gid 1000 app && adduser --uid 1000 --gid 1000 app
RUN mkdir /data && chown app:app /data
USER app
```

3. **SELinux on RHEL/CentOS** blocks bind mounts → add `:z` or `:Z`:
```bash
docker run -v $(pwd)/data:/data:Z myimage
```

**Use case:**
A Node app running as user `node` (UID 1000) can't write uploads to a mounted `./uploads` folder owned by root. `chown -R 1000:1000 ./uploads` resolves it.

**Interview tip:** Mention rootless containers and that running as non-root is a security best practice, so you *design* for UID alignment rather than `chmod 777`.

---

## 🔹 Question 6: How do you fix 'Bind for 0.0.0.0:80 failed: port is already allocated'?

**Answer:**
Another process (or another container) is already bound to the host port you're trying to publish.

**Diagnose:**
```bash
# Find what is using port 80
sudo lsof -i :80
sudo ss -tulpn | grep :80

# Check if another container has it
docker ps --format '{{.Names}}\t{{.Ports}}'
```

**Resolve:**
```bash
# Option 1: stop the conflicting container
docker stop old_web

# Option 2: map to a different host port
docker run -p 8080:80 nginx

# Option 3: remove stale container holding the port
docker rm -f $(docker ps -aq --filter "publish=80")
```

**Use case:**
A previous `nginx` container wasn't cleaned up and still holds port 80. `docker ps` reveals it; stopping/removing it frees the port.

**Interview tip:** Note the format is `hostPort:containerPort`. Changing only the host side (`8080:80`) avoids conflicts without changing the app config.

---

## 🔹 Question 7: How do you resolve the 'Cannot Start Service: Mounts Denied' error?

**Answer:**
Seen mostly on **Docker Desktop (macOS/Windows)**. The host path you're bind-mounting isn't in Docker Desktop's list of shared/allowed file-sharing paths.

**Resolve:**
- **Docker Desktop → Settings → Resources → File Sharing** → add the parent directory (e.g., `/Users/me/projects`).
- Ensure the path exists and is spelled correctly (case-sensitive on some setups).
- Use absolute paths in your compose/run command.

**Use case:**
```yaml
services:
  app:
    volumes:
      - /Users/me/projects/app:/usr/src/app   # must be under a shared folder
```
If `/Users/me/projects` isn't shared, add it in Docker Desktop settings and restart.

**Interview tip:** This is a Docker Desktop virtualization quirk (files cross a VM boundary via gRPC-FUSE/virtiofs). On native Linux you rarely see it.

---

## 🔹 Question 8: What causes the 'connection reset by peer' error, and how do you troubleshoot it?

**Answer:**
The remote end abruptly closed the TCP connection (sent an RST). In Docker context, common during image pulls/pushes, or app-to-app networking.

**Common causes:**
- Network instability, proxy/firewall dropping connections.
- Registry timeout or MTU mismatch (common on VPN/overlay networks).
- The target service crashed or restarted mid-request.

**Troubleshoot:**
```bash
# Test raw connectivity to the registry/service
curl -v https://registry-1.docker.io/v2/
docker exec app ping other-service
docker exec app curl -v http://other-service:8080

# MTU mismatch (classic on overlay/VPN) — check and lower MTU
ip link show docker0
```
Fix MTU in `/etc/docker/daemon.json`:
```json
{ "mtu": 1400 }
```

**Use case:**
Containers on an overlay network behind a VPN fail intermittently on large payloads. Lowering the MTU to 1400 fixes fragmentation-related resets.

**Interview tip:** MTU mismatch is the "senior-level" answer here — it separates people who've debugged real overlay networking from those who haven't.

---

## 🔹 Question 9: How do you resolve the 'Layer already exists' message during a docker build/push?

**Answer:**
`Layer already exists` during `docker push` is **not an error** — it's informational. It means that layer's content (identified by its digest) is already present in the registry, so Docker skips re-uploading it. This is content-addressable storage working as intended.

**Use case:**
You push `app:v2` after changing only the top application layer. The base OS and dependency layers are unchanged, so the registry reports "Layer already exists" for those — only the new layer uploads. This makes pushes fast and saves bandwidth/storage.

**If you actually want to force a rebuild (no cache):**
```bash
docker build --no-cache -t app:v2 .
docker push app:v2
```

**Interview tip:** Clarify the misconception — candidates often think this is an error. Explain layer deduplication via SHA256 digests.

---

## 🔹 Question 10: What causes 'Cannot Delete Network: has active endpoints', and how do you fix it?

**Answer:**
You're trying to remove a network that still has containers attached to it.

**Diagnose & fix:**
```bash
# See which containers are attached
docker network inspect my_net --format '{{json .Containers}}' | jq

# Disconnect or stop/remove those containers
docker network disconnect -f my_net container_name

# Then remove the network
docker network rm my_net
```

**Use case:**
A stopped-but-not-removed container (or a stale endpoint after an ungraceful shutdown) keeps the network "in use." Force-disconnecting the endpoint releases it.

**Bulk cleanup:**
```bash
docker network prune   # removes all unused networks
```

**Interview tip:** Mention that a crashed daemon can leave "ghost" endpoints; `docker network disconnect -f` (force) clears them.

---

## 🔹 Question 11: How do you resolve a 'Unknown instruction in Dockerfile' error?

**Answer:**
The Dockerfile parser hit a line it doesn't recognize as a valid instruction — almost always a typo, wrong casing interpreted as a command, or a broken multi-line continuation.

**Common causes:**
- Typo: `RUNN apt-get update` or `FORM ubuntu`.
- A wrapped command missing the `\` line-continuation, so the second line is parsed as an instruction.
- Wrong file being used as a Dockerfile (e.g., a shell script).

**Example (broken):**
```dockerfile
RUN apt-get update
    apt-get install -y curl     # ❌ parsed as instruction "apt-get"
```
**Fixed:**
```dockerfile
RUN apt-get update && \
    apt-get install -y curl
```

**Use case:** A copy-paste from a README dropped the trailing `\`, splitting a `RUN` into two lines. Adding `&& \` fixes it.

**Interview tip:** Instructions are case-insensitive but convention is UPPERCASE. The real gotcha is line continuation in multi-line `RUN`.

---

## 🔹 Question 12: What is the 'Cannot kill container' error, and how do you resolve it?

**Answer:**
`docker kill`/`docker stop` fails because the container is stuck — often a zombie/defunct process, a kernel-level hang, or the containerd-shim lost track of the process.

**Resolve:**
```bash
# Try SIGKILL directly
docker kill -s SIGKILL my_container

# Find and kill the underlying host process
docker inspect --format '{{.State.Pid}}' my_container
sudo kill -9 <PID>

# Last resort: restart the Docker daemon
sudo systemctl restart docker
```

**Use case:**
A container stuck in `D` (uninterruptible sleep) due to a hung NFS mount can't be killed normally. Killing the host PID or clearing the stuck mount resolves it.

**Interview tip:** Mention `docker stop` sends SIGTERM then SIGKILL after `--time` (default 10s). A process ignoring SIGTERM and stuck in kernel I/O is the classic "can't kill" scenario.

---

## 🔹 Question 13: How do you resolve the 'invalid reference format' error?

**Answer:**
The image name/tag you passed doesn't conform to Docker's naming rules. Common triggers: uppercase letters in the repo name, spaces, a trailing/leading space, or malformed `name:tag@digest`.

**Rules:** repository names must be lowercase; tags allow `[A-Za-z0-9_.-]`.

**Examples:**
```bash
docker build -t MyApp:Latest .     # ❌ uppercase repo -> invalid
docker build -t myapp:latest .     # ✅

docker run "ubuntu :20.04"         # ❌ space in reference
docker run ubuntu:20.04            # ✅
```

**Use case:**
A CI pipeline builds `-t $IMAGE_NAME:$TAG` where `$TAG` is empty, producing `myapp:` → invalid. Guard the variable or default it:
```bash
docker build -t "myapp:${TAG:-latest}" .
```

**Interview tip:** Most real-world cases are **empty/uppercase variables in CI**. Show you'd add validation/defaults in the pipeline.

---

## 🔹 Question 14: What causes 'Cannot connect to Docker Daemon at unix:///var/run/docker.sock', and how do you fix it?

**Answer:**
Same root family as Q2, but specifically about the Unix socket. The CLI talks to the daemon over `/var/run/docker.sock`; this fails if the daemon is down, the socket is missing, or you lack permission.

**Checklist:**
```bash
# 1. Is the daemon up?
sudo systemctl status docker

# 2. Does the socket exist and who owns it?
ls -l /var/run/docker.sock     # should be root:docker

# 3. Are you in the docker group?
groups $USER

# 4. Is DOCKER_HOST overriding the socket?
echo $DOCKER_HOST
```

**Fixes:**
```bash
sudo systemctl start docker
sudo usermod -aG docker $USER && newgrp docker
unset DOCKER_HOST   # if it points somewhere wrong
```

**Use case (Docker-in-Docker / CI):**
A CI job mounts the socket to run Docker commands. If the mount path is wrong, you hit this error:
```yaml
volumes:
  - /var/run/docker.sock:/var/run/docker.sock
```

**Interview tip:** Distinguish "daemon down" from "permission denied" — both surface here but have different fixes.

---

## 🔹 Question 15: How do you troubleshoot 'Container Exited with Code 137'?

**Answer:**
Exit code **137 = 128 + 9**, meaning the process received **SIGKILL (signal 9)**. Almost always the **OOM (Out Of Memory) killer** terminated it, or Docker forcibly killed it after a stop timeout.

**Diagnose:**
```bash
# Confirm OOM
docker inspect my_container --format '{{.State.OOMKilled}}'   # true = OOM

docker inspect my_container --format '{{.State.ExitCode}}'
dmesg | grep -i "killed process"     # host kernel OOM log
docker stats                          # live memory usage
```

**Resolve:**
```bash
# Raise the memory limit
docker run -m 1g --memory-swap 1g myimage

# Or fix the app: memory leak, oversized JVM heap, etc.
```
For the JVM specifically, make it container-aware:
```dockerfile
ENV JAVA_OPTS="-XX:MaxRAMPercentage=75.0"
```

**Use case:**
A Java service with `-Xmx2g` runs in a container limited to `1g`. The kernel OOM-kills it → exit 137. Fix by aligning heap to the container limit or raising the limit.

**Interview tip:** Always decode signals: 137→SIGKILL/OOM, 143→SIGTERM (graceful stop), 139→SIGSEGV. Mention checking `.State.OOMKilled` to confirm.

---

# Part B — Advanced Docker Concepts

---

## 🔹 Question 1: What is Docker Swarm, and how does it differ from Docker Compose?

**Answer:**
- **Docker Compose** defines and runs **multi-container applications on a single host** using a YAML file. It's a *development/single-node* tool.
- **Docker Swarm** is Docker's native **container orchestrator** for running services across a **cluster of nodes** with scheduling, scaling, load balancing, rolling updates, and self-healing.

| Aspect | Docker Compose | Docker Swarm |
|--------|---------------|--------------|
| Scope | Single host | Multi-host cluster |
| Purpose | Dev / local | Production orchestration |
| Scaling | Manual, one host | `docker service scale`, across nodes |
| Self-healing | ❌ | ✅ reschedules failed tasks |
| Load balancing | ❌ (external) | ✅ built-in routing mesh |
| File | `docker-compose.yml` | Stack file (`docker stack deploy`) |

**Use case:**
- Compose: spin up `app + postgres + redis` on your laptop for local dev.
- Swarm: run that same stack across 5 nodes in production with 3 replicas of the app, auto-restarting failed tasks.

**Examples:**
```bash
# Compose (single host)
docker compose up -d

# Swarm (cluster)
docker swarm init
docker stack deploy -c docker-compose.yml myapp
docker service scale myapp_web=5
```

**Interview tip:** Note that Swarm can consume Compose files via `docker stack deploy`, but ignores dev-only keys (`build`, `depends_on` ordering).

---

## 🔹 Question 2: Explain Docker networking modes: bridge, host, overlay, and macvlan.

**Answer:**

| Mode | Description | Use case |
|------|-------------|----------|
| **bridge** (default) | Private internal network on a single host; containers get an internal IP, published ports via NAT. | Standard single-host apps. |
| **host** | Container shares the host's network stack directly (no isolation, no NAT). | Max network performance, low latency (e.g., high-throughput proxies). |
| **overlay** | Multi-host network spanning a Swarm cluster; containers on different nodes communicate as if on one L2 network. | Swarm services across nodes. |
| **macvlan** | Assigns containers a real MAC/IP on the physical LAN — they appear as physical devices. | Legacy apps needing a real LAN IP; network appliances. |
| **none** | No networking. | Fully isolated batch jobs. |

**Examples:**
```bash
# User-defined bridge (gives DNS-based service discovery)
docker network create --driver bridge app_net
docker run --network app_net --name api myapi

# Host mode
docker run --network host nginx

# Overlay (Swarm)
docker network create -d overlay --attachable prod_net

# Macvlan
docker network create -d macvlan \
  --subnet=192.168.1.0/24 --gateway=192.168.1.1 \
  -o parent=eth0 pub_net
```

**Interview tip:** Stress that a **user-defined bridge** provides automatic DNS resolution between containers by name, unlike the default bridge (which needs `--link`, now deprecated).

---

## 🔹 Question 3: How do you handle persistent storage in Docker containers?

**Answer:**
Containers are ephemeral — their writable layer is destroyed on removal. For persistence, use:

1. **Volumes** (recommended): managed by Docker under `/var/lib/docker/volumes`. Portable, backup-friendly, work with drivers.
2. **Bind mounts**: map a host directory into the container. Great for dev; tightly coupled to host path.
3. **tmpfs**: in-memory only (not persistent) — for sensitive/temporary data.

**Examples:**
```bash
# Named volume
docker volume create db_data
docker run -v db_data:/var/lib/postgresql/data postgres

# Bind mount (dev)
docker run -v $(pwd)/src:/app/src node:20

# tmpfs
docker run --tmpfs /tmp:size=64m alpine
```
Compose:
```yaml
services:
  db:
    image: postgres:16
    volumes:
      - db_data:/var/lib/postgresql/data
volumes:
  db_data:
```

**Use case:**
A PostgreSQL container stores its data on the `db_data` volume, so `docker rm` + `docker run` a new container keeps all data. For multi-host clusters, use a volume driver (NFS, EBS, Portworx) so any node can mount it.

**Interview tip:** Volumes > bind mounts for production (host-independent, driver support, easier backup via `docker run --rm -v db_data:/data -v $(pwd):/backup alpine tar czf /backup/db.tgz /data`).

---

## 🔹 Question 4: Explain the difference between Docker images and Docker containers.

**Answer:**
- **Image**: a read-only, immutable **template** built from layers — the blueprint (code + dependencies + config). Analogous to a *class*.
- **Container**: a **running (or stopped) instance** of an image with a thin writable layer on top. Analogous to an *object*.

**Analogy:** Image = recipe; Container = the actual dish you cooked from it. You can cook many dishes (containers) from one recipe (image).

**Examples:**
```bash
docker pull nginx:latest          # image
docker run --name web nginx       # container (instance of image)
docker run --name web2 nginx      # another container, same image
docker images                     # list images
docker ps -a                      # list containers
```

**Interview tip:** Mention layers — images share read-only layers across containers (copy-on-write), which is why launching many containers from one image is cheap on disk.

---

## 🔹 Question 5: What are Docker secrets, and how are they used for managing sensitive data?

**Answer:**
Docker **secrets** securely store sensitive data (passwords, TLS certs, API keys) in the **Swarm** cluster. Secrets are:
- Encrypted at rest in the Raft log and in transit.
- Mounted into the container as an in-memory file at `/run/secrets/<name>` (not env vars, not on disk, not in the image).
- Available only to services explicitly granted access.

**Examples:**
```bash
# Create a secret
echo "s3cr3t_pw" | docker secret create db_password -

# Grant it to a service
docker service create --name db \
  --secret db_password \
  -e POSTGRES_PASSWORD_FILE=/run/secrets/db_password \
  postgres:16
```
Compose (stack):
```yaml
services:
  db:
    image: postgres:16
    secrets:
      - db_password
    environment:
      POSTGRES_PASSWORD_FILE: /run/secrets/db_password
secrets:
  db_password:
    external: true
```

**Use case:**
Instead of baking a DB password into the image or passing `-e POSTGRES_PASSWORD=...` (visible via `docker inspect` and shell history), the app reads it from `/run/secrets/db_password`.

**Interview tip:** Secrets are a **Swarm-only** feature. For standalone Docker or Kubernetes, use external secret managers (Vault, AWS Secrets Manager, K8s Secrets). Never use ENV for secrets — they leak via `inspect`, logs, and child processes.

---

## 🔹 Question 6: Explain Docker multi-stage builds and their benefits.

**Answer:**
Multi-stage builds use multiple `FROM` statements in one Dockerfile. You build/compile in a "builder" stage with all the toolchain, then **copy only the final artifacts** into a slim runtime stage — leaving compilers, dev deps, and source out of the final image.

**Benefits:**
- Drastically smaller final images.
- Smaller attack surface (no build tools in production image).
- Single Dockerfile, no external build scripts.

**Example (Go):**
```dockerfile
# Stage 1: build
FROM golang:1.22 AS builder
WORKDIR /src
COPY . .
RUN CGO_ENABLED=0 go build -o /app .

# Stage 2: runtime
FROM alpine:3.20
COPY --from=builder /app /app
ENTRYPOINT ["/app"]
```
The final image is a few MB (Alpine + binary) vs. ~800MB for the full Go image.

**Use case:**
A React app: `node` stage runs `npm ci && npm run build`, then an `nginx` stage serves only the static `dist/` folder — no `node_modules` in production.

**Interview tip:** Mention `--target` to build a specific stage (e.g., a `test` stage in CI) and using distroless/scratch as the final base for minimal size.

---

## 🔹 Question 7: How do you monitor Docker containers and orchestration environments in production?

**Answer:**
Layered monitoring across metrics, logs, and traces:

1. **Metrics**: `cAdvisor` (per-container CPU/mem/net/disk) → **Prometheus** → **Grafana** dashboards.
2. **Node/host**: `node_exporter`.
3. **Logs**: centralize via **ELK/EFK** (Elasticsearch + Fluentd/Filebeat + Kibana) or Loki.
4. **Orchestrator health**: Swarm `docker service ps`, or Kubernetes `kubectl top`, plus `kube-state-metrics`.
5. **Alerting**: Prometheus Alertmanager (CPU throttling, OOM, restart loops, unhealthy).
6. **Tracing (optional)**: OpenTelemetry + Jaeger for request-level insight.

**Examples:**
```bash
docker stats                     # quick live view
docker service ps myapp_web      # Swarm task health
docker logs -f --tail 100 web
```
Prometheus + cAdvisor (compose snippet):
```yaml
cadvisor:
  image: gcr.io/cadvisor/cadvisor:latest
  ports: ["8080:8080"]
  volumes:
    - /:/rootfs:ro
    - /var/run:/var/run:ro
    - /sys:/sys:ro
    - /var/lib/docker/:/var/lib/docker:ro
```

**Interview tip:** Golden signals — **latency, traffic, errors, saturation**. Mention alerting on restart loops and OOMKills, and log rotation to protect disk.

---

## 🔹 Question 8: Explain Docker Healthchecks and how they are used to monitor container health.

**Answer:**
A `HEALTHCHECK` instruction tells Docker how to test that a container is *functionally* healthy (not just running). Docker periodically runs the command; based on exit code (`0`=healthy, `1`=unhealthy) it sets the container's health state to `starting` → `healthy`/`unhealthy`.

**Parameters:** `--interval`, `--timeout`, `--start-period`, `--retries`.

**Example:**
```dockerfile
HEALTHCHECK --interval=30s --timeout=3s --start-period=20s --retries=3 \
  CMD curl -f http://localhost:8080/health || exit 1
```
Or at runtime:
```bash
docker run --health-cmd='curl -f http://localhost/ || exit 1' \
  --health-interval=15s nginx
```

**Use case:**
In Swarm, an `unhealthy` task is stopped and rescheduled; a load balancer only routes to `healthy` tasks — preventing traffic to a deadlocked-but-running app. Also used with `depends_on: condition: service_healthy` in Compose to sequence startup (e.g., app waits for DB to be healthy).

**Interview tip:** `--start-period` gives slow-booting apps grace time; failures during it don't count against `--retries`. Keep the healthcheck lightweight to avoid load.

---

## 🔹 Question 9: What are Docker plugins, and how are they used to extend Docker functionality?

**Answer:**
Plugins extend the Docker Engine with custom behavior via defined APIs. Main types:
- **Volume plugins**: custom storage backends (e.g., `rexray`, `portworx`, NFS, cloud block storage).
- **Network plugins**: custom networking (e.g., Weave, Calico).
- **Log driver plugins**: ship logs to external systems (e.g., Splunk, Fluentd, Loki).
- **Authorization plugins**: enforce access policies on Docker API calls.

**Examples:**
```bash
# Install a managed plugin
docker plugin install vieux/sshfs
docker plugin ls
docker plugin enable vieux/sshfs

# Use it as a volume driver
docker volume create -d vieux/sshfs -o sshcmd=user@host:/data remote_vol
docker run -v remote_vol:/data alpine
```

**Use case:**
A multi-node Swarm needs shared persistent storage. A volume plugin (e.g., cloud EBS/Portworx driver) lets a volume follow a container to whichever node it's rescheduled on.

**Interview tip:** Distinguish **managed plugins** (installed via `docker plugin install`, run in their own rootfs) from legacy plugins. Mention log driver plugins for centralized logging.

---

## 🔹 Question 10: Explain Docker build caching and its impact on image build times.

**Answer:**
Docker builds images layer by layer and **caches each layer**. On rebuild, if an instruction and its inputs are unchanged, Docker reuses the cached layer instead of re-executing it. Once one layer's cache is invalidated, **all subsequent layers rebuild**.

**Key rule:** Order instructions from **least-frequently-changing to most-frequently-changing** to maximize cache hits.

**Bad (cache busts on every code change):**
```dockerfile
COPY . .
RUN npm install      # re-runs even when only source changed
```
**Good (deps cached separately):**
```dockerfile
COPY package*.json ./
RUN npm ci           # cached unless dependencies change
COPY . .             # only this layer rebuilds on code change
```

**Impact:** Proper ordering can cut build times from minutes to seconds because expensive steps (dependency installs) are cached.

**Controls:**
```bash
docker build --no-cache .            # ignore cache entirely
docker build --build-arg X=1 .       # changing ARG busts cache from that point
```
BuildKit adds `--mount=type=cache` for persistent dependency caches:
```dockerfile
RUN --mount=type=cache,target=/root/.npm npm ci
```

**Interview tip:** Mention BuildKit cache mounts and `.dockerignore` (prevents unrelated files from busting the `COPY` cache).

---

## 🔹 Question 11: How do you manage Docker Swarm secrets across multiple environments (dev, staging, prod)?

**Answer:**
Since secrets are **immutable** and scoped per Swarm cluster, manage them per-environment using naming/versioning conventions and external sources:

1. **Separate Swarm clusters** per environment (isolation) — each holds its own secret values under the same logical name.
2. **Environment-prefixed or versioned names**: `db_password_v2` (since you can't update a secret in place — you create a new one).
3. **External source of truth**: store real values in Vault/AWS Secrets Manager and sync them into each Swarm's secrets during deploy (CI/CD step), never in Git.
4. **Stack files reference `external: true`** so the same compose file works across envs, with values injected per cluster.

**Example (CI creates versioned secret, service points to it):**
```bash
# Per environment, pull from Vault and create
vault kv get -field=password secret/staging/db | docker secret create db_password_v3 -

docker service update --secret-rm db_password_v2 \
  --secret-add source=db_password_v3,target=db_password myapp_db
```
```yaml
secrets:
  db_password:
    external: true
    name: db_password_v3   # bump per environment/rotation
```

**Interview tip:** Emphasize secrets are immutable → rotation = create new + update service + remove old. Keep values out of Git; use per-env clusters or namespaced names.

---

## 🔹 Question 12: Explain Docker container networking modes and their use cases.

**Answer:**
(Builds on Part B Q2 — focus here on *when to choose each*.)

- **bridge** → default; isolated single-host apps; use a **user-defined bridge** for automatic DNS between containers (microservices on one host).
- **host** → when you need bare-metal network performance or the app binds many ports dynamically (e.g., a high-throughput reverse proxy, monitoring agent). Trade-off: no port isolation, potential port conflicts.
- **overlay** → multi-host Swarm services that must talk across nodes; includes the built-in **routing mesh** for load balancing published ports.
- **macvlan** → when a container must have a real IP on the physical network (legacy apps, monitoring tools sniffing traffic, DHCP integration).
- **none** → complete network isolation for security-sensitive batch jobs.

**Decision cheat-sheet:**
| Need | Mode |
|------|------|
| Simple isolated app on one host | bridge |
| Service discovery by name on one host | user-defined bridge |
| Lowest network latency | host |
| Cross-node cluster communication | overlay |
| Container as a first-class LAN device | macvlan |
| No network at all | none |

**Interview tip:** Tie each mode to a concrete scenario — interviewers want *why*, not just *what*.

---

## 🔹 Question 13: How do you manage Docker secrets rotation and key rotation in production?

**Answer:**
Because Docker secrets are immutable, rotation is a **create-new → update-service → remove-old** workflow, ideally automated and driven by an external secret manager.

**Rotation steps:**
```bash
# 1. Create the new secret version
echo "new_password" | docker secret create db_password_v2 -

# 2. Update the service to use the new secret (rolling update, no downtime)
docker service update \
  --secret-rm db_password_v1 \
  --secret-add source=db_password_v2,target=db_password \
  myapp_db

# 3. Once all tasks are healthy, remove the old secret
docker secret rm db_password_v1
```

**Best practices:**
- **Automate** via CI/CD + Vault/AWS Secrets Manager (dynamic secrets, TTL-based auto-rotation).
- Ensure the app **re-reads** the mounted secret file (or gets restarted) after rotation.
- For TLS/keys, rotate certs the same way and reload the app gracefully (SIGHUP).
- Keep an **overlap window** (both keys valid briefly) so in-flight sessions don't break.
- Audit and log rotations; alert on failures.

**Use case:**
Quarterly DB credential rotation: Vault generates a new dynamic credential, CI creates `db_password_v2`, `service update` rolls it out task-by-task, old secret removed after verification — zero downtime.

**Interview tip:** Emphasize the overlap/dual-key window and app reloading the secret. Mention Vault dynamic secrets for automatic, short-lived credentials.

---

## 🔹 Question 14: Explain Docker Content Trust (DCT) and how it enhances container security.

**Answer:**
**Docker Content Trust (DCT)** provides **image signing and verification** using **Notary** (based on The Update Framework / TUF). When enabled, Docker only pulls/runs images whose tags are **cryptographically signed** by a trusted publisher, guaranteeing **integrity** (image wasn't tampered with) and **authenticity** (it came from who you expect).

**Enable it:**
```bash
export DOCKER_CONTENT_TRUST=1

# Now pushes are signed automatically...
docker push myrepo/app:1.0.0

# ...and pulls of unsigned images are REJECTED
docker pull myrepo/app:untrusted   # fails if not signed
```

**How it protects you:**
- Prevents running tampered or malicious images (supply-chain attacks).
- Protects against tag-tampering (someone repushing a malicious `:latest`).
- Uses offline root keys + repository/target keys for a chain of trust.

**Use case:**
A production cluster sets `DOCKER_CONTENT_TRUST=1` so it refuses any image not signed by the org's release pipeline — blocking accidental or malicious unsigned deployments.

**Interview tip:** Note DCT is tag-level signing via Notary v1; modern alternatives/complements include **Cosign/Sigstore** (keyless signing) and admission controllers that enforce signatures in Kubernetes. Also mention **image scanning** (Trivy/Clair) as complementary to signing.

---

## 🔹 Question 15: How do you optimize Docker container performance for production workloads?

**Answer:**
Optimize across **image, runtime, resources, and networking**:

**1. Image size & startup:**
- Multi-stage builds + minimal base (`alpine`, `distroless`, `scratch`).
- Fewer, well-ordered layers; use `.dockerignore`.

**2. Resource limits (prevent noisy neighbors & OOM):**
```bash
docker run --cpus="1.5" -m 512m --memory-swap 512m myapp
```
- Set requests/limits so the scheduler places workloads correctly.
- Make the runtime container-aware (JVM `-XX:MaxRAMPercentage`, Node `--max-old-space-size`).

**3. Storage:**
- Use volumes (not the writable layer) for I/O-heavy data.
- Choose an efficient storage driver (`overlay2`).

**4. Networking:**
- Use `host` mode or tuned MTU for latency-sensitive services; user-defined bridges for efficient service discovery.

**5. Logging & observability:**
- Cap logs (`max-size`/`max-file`) to protect disk and I/O.
- Monitor with cAdvisor/Prometheus; watch CPU throttling and OOMKills.

**6. Process hygiene:**
- One primary process per container; use an init (`--init`) to reap zombies.
- Health checks for self-healing.

**Use case:**
A latency-sensitive API: slim distroless image, `--cpus`/`-m` limits tuned from load tests, `--init` for zombie reaping, JVM heap set to 75% of the memory limit, logs capped, and Prometheus alerts on CPU throttle — yielding stable p99 latency under load.

**Interview tip:** Frame optimization as **measure → tune → verify**. Mention CPU throttling (from `--cpus` quota) as a subtle latency killer people miss.

---

# Part C — Docker Fundamentals

---

## 🔹 Question 1: What is Docker?

**Answer:**
Docker is an open-source **containerization platform** that packages an application and all its dependencies (libraries, runtime, config) into a portable, isolated unit called a **container**. Containers share the host OS kernel but run in isolated user spaces, making them lightweight, fast, and consistent across environments ("build once, run anywhere").

**Use case:**
"Works on my machine" problems disappear — a developer builds an image locally; the exact same image runs in CI, staging, and production without dependency drift.

**Example:**
```bash
docker run -d -p 8080:80 nginx    # run nginx in seconds, no local install
```

**Interview tip:** Core benefits — **consistency, isolation, portability, density, fast startup**. Docker uses Linux kernel features: **namespaces** (isolation) and **cgroups** (resource limits).

---

## 🔹 Question 2: Explain the difference between a container and a virtual machine (VM).

**Answer:**

| Aspect | Container | Virtual Machine |
|--------|-----------|-----------------|
| Isolation | OS-level (namespaces/cgroups) | Hardware-level (hypervisor) |
| OS | Shares host kernel | Full guest OS per VM |
| Size | MBs | GBs |
| Startup | Seconds/ms | Minutes |
| Overhead | Low | High |
| Density | Many per host | Fewer per host |

**Analogy:** VMs are like separate houses (each with its own foundation/OS); containers are apartments in one building (shared foundation/kernel, isolated units).

**Use case:**
Run 50 microservice containers on one host efficiently; you'd need far more resources to run 50 VMs. Use VMs when you need **strong isolation** or a **different kernel/OS** (e.g., running Windows and Linux workloads with hard security boundaries).

**Interview tip:** Containers share the kernel → lighter but weaker isolation than VMs. Some setups combine both (containers inside VMs) for security + density.

---

## 🔹 Question 3: What is a Docker image?

**Answer:**
A Docker image is a **read-only, immutable template** composed of stacked **layers** that contains everything needed to run an application: code, runtime, libraries, environment variables, and config. Containers are created from images.

**Key points:**
- Built from a `Dockerfile`.
- Layered and content-addressable (SHA256) → layers are cached and shared.
- Identified by `repository:tag` (e.g., `nginx:1.27`) and a digest.

**Example:**
```bash
docker pull python:3.12-slim
docker images
docker history python:3.12-slim   # see the layers
```

**Interview tip:** Mention layer sharing (copy-on-write) — multiple images/containers reuse identical base layers, saving disk and speeding pulls.

---

## 🔹 Question 4: Explain the role of a Dockerfile.

**Answer:**
A **Dockerfile** is a text file with instructions that Docker executes **in order** to build an image automatically and reproducibly. Each instruction typically creates a layer.

**Common instructions:**
| Instruction | Purpose |
|-------------|---------|
| `FROM` | Base image |
| `WORKDIR` | Set working directory |
| `COPY`/`ADD` | Copy files into the image |
| `RUN` | Execute a command at build time |
| `ENV` | Set environment variables |
| `EXPOSE` | Document the listening port |
| `CMD` | Default command at runtime |
| `ENTRYPOINT` | Fixed executable at runtime |

**Example:**
```dockerfile
FROM python:3.12-slim
WORKDIR /app
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt
COPY . .
EXPOSE 8000
CMD ["python", "app.py"]
```

**Interview tip:** `CMD` provides default args (overridable); `ENTRYPOINT` sets the executable (harder to override). Use them together: `ENTRYPOINT ["python"]` + `CMD ["app.py"]`.

---

## 🔹 Question 5: What is Docker Compose, and how is it used?

**Answer:**
Docker Compose is a tool to **define and run multi-container applications** using a single declarative YAML file (`docker-compose.yml` / `compose.yaml`). It manages the app's services, networks, and volumes together.

**Example:**
```yaml
services:
  web:
    build: .
    ports: ["8000:8000"]
    depends_on: [db]
  db:
    image: postgres:16
    environment:
      POSTGRES_PASSWORD: secret
    volumes:
      - db_data:/var/lib/postgresql/data
volumes:
  db_data:
```
```bash
docker compose up -d      # start whole stack
docker compose logs -f    # tail logs
docker compose down       # stop & remove
```

**Use case:**
Local development of an app that needs a web server, database, and cache — one command spins up the entire environment with networking wired automatically (services reach each other by name, e.g., `db:5432`).

**Interview tip:** Compose is single-host. It creates a default network so services resolve each other by service name via DNS.

---

## 🔹 Question 6: Explain the concept of Docker Swarm.

**Answer:**
Docker Swarm is Docker's **native clustering and orchestration** solution. It turns a pool of Docker hosts into a single virtual cluster with **manager nodes** (orchestration, Raft consensus) and **worker nodes** (run tasks). It provides declarative services, scaling, load balancing (routing mesh), rolling updates, and self-healing.

**Key concepts:** service, task, replica, manager/worker, routing mesh.

**Example:**
```bash
docker swarm init                                  # create cluster (manager)
docker swarm join --token <token> <manager-ip>     # add worker
docker service create --name web --replicas 3 -p 80:80 nginx
docker service scale web=5
docker service update --image nginx:1.27 web        # rolling update
```

**Interview tip:** Swarm is simpler than Kubernetes but less feature-rich; good for smaller clusters or teams wanting native Docker tooling. Managers use Raft (keep an odd number: 3 or 5 for HA).

---

## 🔹 Question 7: What is the difference between Docker Swarm and Kubernetes?

**Answer:**

| Aspect | Docker Swarm | Kubernetes |
|--------|--------------|------------|
| Complexity | Simple, quick setup | Complex, steeper learning curve |
| Scaling | Good, manual/auto basic | Advanced (HPA/VPA, cluster autoscaler) |
| Load balancing | Built-in routing mesh | Services + Ingress controllers |
| Ecosystem | Smaller | Huge (Helm, operators, CNCF) |
| Self-healing | Yes | Yes, more sophisticated |
| Auto-scaling | Limited | Native (metrics-based) |
| Best for | Small/simple clusters | Large, complex, enterprise |

**Use case:**
- Swarm: a small team wanting orchestration fast with familiar Docker CLI.
- Kubernetes: enterprise-scale, multi-team platforms needing rich scheduling, autoscaling, and ecosystem tooling.

**Interview tip:** Kubernetes has effectively won as the industry standard, but Swarm is valid for smaller/simpler needs. Both handle scheduling, scaling, and self-healing.

---

## 🔹 Question 8: How do you share data between Docker containers?

**Answer:**
Several mechanisms:

1. **Shared named volume** (most common):
```bash
docker volume create shared
docker run -v shared:/data --name writer alpine
docker run -v shared:/data --name reader alpine
```
2. **`--volumes-from`** (mount another container's volumes):
```bash
docker run --volumes-from writer alpine
```
3. **Network communication** (share via APIs, not files) — containers on the same user-defined network talk by name.
4. **Bind mount** the same host directory into multiple containers.

**Use case:**
A data-processing container writes files to a shared volume; a separate uploader container reads from that same volume and pushes to cloud storage — decoupled, single-responsibility containers.

**Interview tip:** Prefer named volumes over `--volumes-from` for clarity. For loose coupling, share via the network/APIs rather than files when possible.

---

## 🔹 Question 9: What is Docker Hub, and why is it used?

**Answer:**
Docker Hub is Docker's **cloud-based registry** for storing, sharing, and distributing container images. It hosts **official images** (nginx, postgres, python), verified publisher images, and user/organization repositories (public or private).

**Why used:**
- Central place to `push`/`pull` images.
- Official, maintained base images.
- Automated builds and webhooks.
- Team collaboration via private repos.

**Example:**
```bash
docker login
docker tag myapp:latest myuser/myapp:1.0.0
docker push myuser/myapp:1.0.0
docker pull myuser/myapp:1.0.0
```

**Interview tip:** Mention alternatives (AWS ECR, GCP Artifact Registry, GitHub Container Registry, Harbor) and Docker Hub **rate limits** for anonymous pulls — a real production consideration.

---

## 🔹 Question 10: Explain the concept of Docker networking.

**Answer:**
Docker networking lets containers communicate with each other, the host, and external networks. Docker uses a pluggable **CNM (Container Network Model)** with drivers (bridge, host, overlay, macvlan, none). By default, containers attach to the `bridge` network.

**Key points:**
- **User-defined bridge** networks give automatic **DNS-based service discovery** (containers reach each other by name).
- Ports are published to the host via `-p host:container`.
- Overlay networks span multiple hosts in Swarm.

**Example:**
```bash
docker network create app_net
docker run -d --name db --network app_net postgres
docker run -d --name api --network app_net myapi   # api can reach "db:5432"
```

**Interview tip:** The big win of user-defined bridges over the default bridge is embedded DNS — no need for legacy `--link`.

---

## 🔹 Question 11: What is Docker Swarm mode, and how do you initialize a Swarm?

**Answer:**
Swarm mode is the built-in orchestration capability of the Docker Engine (enabled with `docker swarm init`). It lets you manage a cluster of Docker nodes as a single system and deploy **services** declaratively.

**Initialize:**
```bash
# On the first manager node
docker swarm init --advertise-addr <MANAGER-IP>

# Output gives a join command for workers:
docker swarm join --token SWMTKN-... <MANAGER-IP>:2377

# Get a manager join token to add more managers
docker swarm join-token manager

# Inspect the cluster
docker node ls
```

**Use case:**
Turn three VMs into a cluster: init on one (manager), join the other two (workers), then `docker stack deploy` your app across them with replicas and rolling updates.

**Interview tip:** Port 2377 = cluster management, 7946 = node communication, 4789 = overlay network traffic (VXLAN). Keep an odd number of managers (3/5) for Raft quorum.

---

## 🔹 Question 12: Explain the purpose of Docker volumes.

**Answer:**
Volumes are the **preferred mechanism for persisting data** generated and used by containers, independent of the container lifecycle. Docker manages them under `/var/lib/docker/volumes`, decoupled from the host filesystem layout.

**Advantages over bind mounts:**
- Managed by Docker (portable, host-path independent).
- Easier backup/restore and migration.
- Support volume drivers (NFS, cloud storage) for multi-host use.
- Better performance on Docker Desktop.
- Can be shared safely among containers.

**Example:**
```bash
docker volume create app_data
docker run -v app_data:/var/lib/mysql mysql:8
docker volume ls
docker volume inspect app_data

# Backup a volume
docker run --rm -v app_data:/data -v $(pwd):/backup alpine \
  tar czf /backup/app_data.tgz -C /data .
```

**Interview tip:** Data in the container's writable layer is lost on `docker rm`; volumes survive. Use volumes for databases, uploads, and any stateful data.

---

## 🔹 Question 13: How do you monitor Docker containers and services?

**Answer:**
Combine built-in commands with a proper observability stack:

**Built-in:**
```bash
docker stats                 # live CPU/mem/net/IO per container
docker ps                    # running containers & status/health
docker logs -f --tail 100 web
docker top web               # processes inside a container
docker events                # real-time daemon events
docker service ps myapp      # Swarm task status
```

**Production stack:**
- **cAdvisor + Prometheus + Grafana** for metrics/dashboards.
- **EFK/ELK or Loki** for centralized logs.
- **Alertmanager** for alerting on OOM, restart loops, unhealthy.

**Use case:**
`docker stats` for a quick check; Grafana dashboards + Prometheus alerts for continuous production monitoring with history and paging.

**Interview tip:** Built-in tools are for ad-hoc debugging; production needs centralized, persistent metrics/logs with alerting. (Overlaps with Part B Q7.)

---

## 🔹 Question 14: What are Docker labels, and how are they used?

**Answer:**
Labels are **key-value metadata** attached to Docker objects (images, containers, volumes, networks, services). They enable organizing, filtering, and automation without affecting runtime behavior.

**Set labels:**
```dockerfile
LABEL maintainer="devops@example.com"
LABEL version="1.2.0" environment="production"
```
```bash
docker run -d --label team=payments --label tier=backend nginx
```

**Filter/query by label:**
```bash
docker ps --filter "label=team=payments"
docker images --filter "label=environment=production"
```

**Use case:**
- Group and clean up resources by project/team.
- Reverse proxies like **Traefik** read labels to auto-configure routing:
```yaml
labels:
  - "traefik.http.routers.api.rule=Host(`api.example.com`)"
```
- Prometheus/monitoring uses labels for grouping and cost allocation.

**Interview tip:** Follow reverse-DNS namespacing for keys (`com.example.team`) to avoid collisions. Labels are central to automation tools like Traefik and orchestrators.

---

## 🔹 Question 15: Explain the concept of Docker security.

**Answer:**
Docker security is layered — securing the **image, container runtime, host, and registry/supply chain**:

**Image security:**
- Use minimal, trusted base images; scan with **Trivy/Clair/Grype**.
- Pin versions; keep images patched.
- Don't bake secrets into images.

**Runtime / least privilege:**
- Run as **non-root** (`USER`), drop capabilities (`--cap-drop=ALL`), add only what's needed.
- Read-only filesystem: `--read-only`.
- No `--privileged` unless absolutely required.
- Set resource limits (`--cpus`, `-m`) to limit blast radius.
- Use seccomp/AppArmor/SELinux profiles.

**Host security:**
- Keep Docker/host patched; restrict `docker` group (= root).
- Protect the Docker socket; consider rootless mode.

**Supply chain:**
- **Docker Content Trust / Cosign** for image signing.
- Private registries with auth (ECR, Harbor).

**Secrets:**
- Use Docker secrets / Vault, never ENV or image layers.

**Example:**
```bash
docker run --read-only --cap-drop=ALL --cap-add=NET_BIND_SERVICE \
  -u 1000:1000 --security-opt no-new-privileges myapp
```

**Interview tip:** Summarize as **least privilege + minimal images + scanning + signing + secret management + host hardening**. Mention rootless Docker and CIS Docker Benchmark.

---

# Quick Reference Cheat Sheet

**Lifecycle**
```bash
docker build -t app:1.0 .          # build image
docker run -d --name app -p 80:8080 app:1.0
docker ps / docker ps -a           # list running / all
docker logs -f app                 # follow logs
docker exec -it app sh             # shell into container
docker stop app / docker start app
docker rm -f app                   # force remove
```

**Images**
```bash
docker images
docker pull / docker push
docker tag app:1.0 repo/app:1.0
docker rmi app:1.0
docker history app:1.0
```

**Cleanup**
```bash
docker system df                   # disk usage
docker system prune                # dangling resources
docker system prune -a --volumes   # aggressive (careful!)
docker image/container/volume/network prune
```

**Inspect & Debug**
```bash
docker inspect app
docker stats
docker top app
docker events
docker inspect --format '{{.State.ExitCode}}' app
docker inspect --format '{{.State.OOMKilled}}' app
```

**Common exit codes**
| Code | Meaning |
|------|---------|
| 0 | Success / clean exit |
| 125 | Docker daemon error (bad `run` command) |
| 126 | Command not executable |
| 127 | Command not found |
| 137 | SIGKILL (usually OOM) |
| 139 | SIGSEGV (segfault) |
| 143 | SIGTERM (graceful stop) |

**Swarm**
```bash
docker swarm init
docker service create --name web --replicas 3 -p 80:80 nginx
docker service scale web=5
docker stack deploy -c compose.yml myapp
docker node ls
```

---

*Tip for the interview:* For each answer, lead with a **one-line definition**, follow with a **concrete use case**, then show a **command/example**. This structure signals both conceptual and hands-on knowledge.
