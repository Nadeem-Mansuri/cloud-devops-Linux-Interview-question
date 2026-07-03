# Troubleshooting Application Latency on AWS — 3-Tier & Serverless Architectures

> Interview + operations guide: how to diagnose and fix latency in an application hosted on AWS, layer by layer, for both a classic **3-tier architecture** and a **serverless architecture**. Includes likely scenarios, root causes, AWS-recommended tools, and step-by-step troubleshooting.

---

## 1. Latency Troubleshooting Methodology (The Framework)

Before diving into layers, follow a structured approach — interviewers love this:

1. **Define & measure** — What is "slow"? Get numbers: p50/p90/**p99** latency, not just averages. Latency is about tail percentiles.
2. **Establish a baseline** — Compare current latency to historical norms (CloudWatch dashboards).
3. **Isolate the layer** — Use **distributed tracing (AWS X-Ray)** to see *where* time is spent (client → edge → LB → app → DB → external).
4. **Form a hypothesis** — CPU? Cold start? DB lock? Network hop? N+1 query?
5. **Test & verify** — Change one variable, re-measure.
6. **Fix & monitor** — Apply fix, set alarms to catch regression.

> **Golden rule:** *Measure first, don't guess.* Use **X-Ray service maps** to find the bottleneck segment before touching any code or config.

### Key AWS Observability Tools

| Tool | What it tells you |
|------|-------------------|
| **CloudWatch Metrics** | Latency, CPU, memory, connections, error rates per service |
| **CloudWatch Logs / Logs Insights** | Application & access logs, query patterns |
| **AWS X-Ray** | End-to-end distributed traces, service map, per-segment latency |
| **CloudWatch ServiceLens** | X-Ray + metrics + logs unified view |
| **CloudWatch Synthetics** | Canaries that measure user-facing latency proactively |
| **CloudWatch RUM** | Real-user monitoring (browser-side latency) |
| **VPC Flow Logs** | Network-level connection/latency issues |
| **CloudWatch Contributor Insights** | Top-N contributors to latency/load |

---

# PART A — 3-Tier Architecture Latency Troubleshooting

```
Client → Route 53 → CloudFront → ALB (Tier 1: Web)
                                   → EC2/ECS (Tier 2: App)
                                       → RDS/Aurora (Tier 3: Data)
```

---

## 2. Layer 0 — Client / DNS / Edge

### Possible Scenarios
- Slow **DNS resolution**.
- Users far from the region → high network **round-trip time (RTT)**.
- Large, uncached static assets.
- TLS handshake overhead.

### How to Troubleshoot
- **CloudWatch RUM / Synthetics** — measure real browser latency by geography.
- Check **Route 53** resolution time; use **latency-based** or **geolocation routing**.
- Inspect **CloudFront cache hit ratio** (CloudWatch metric `CacheHitRate`).

### AWS Recommendations
- Use **CloudFront (CDN)** to cache static + dynamic content at edge locations near users.
- Enable **HTTP/2 / HTTP/3**, **compression (gzip/brotli)**, and **keep-alive**.
- Use **Route 53 latency-based routing** to send users to the nearest region.
- Terminate **TLS at the edge** (CloudFront) and reuse connections.
- Consider **Global Accelerator** for non-HTTP or latency-sensitive TCP/UDP traffic.

---

## 3. Tier 1 — Load Balancer / Web Layer

### Possible Scenarios
- ALB adding latency due to unhealthy or slow targets.
- Uneven load distribution across targets.
- TLS negotiation / connection saturation.
- Cross-AZ traffic hops.

### How to Troubleshoot
- **ALB CloudWatch metrics**:
  - `TargetResponseTime` — time the backend took (isolates app vs LB).
  - `RequestCount`, `HTTPCode_ELB_5XX`, `HTTPCode_Target_5XX`.
  - `RejectedConnectionCount`, `ActiveConnectionCount`.
- Enable **ALB access logs** → analyze in Athena for slow requests.
- Check **target group health** — unhealthy targets shrink capacity.

### AWS Recommendations
- Enable **cross-zone load balancing** for even distribution.
- Tune **health check** thresholds and **deregistration delay**.
- Enable **connection draining** and **keep-alive**.
- Use **sticky sessions** only if needed (can cause imbalance).
- Right-size and **Auto Scale** targets based on `TargetResponseTime` / request count.

---

## 4. Tier 2 — Application / Compute Layer

### Possible Scenarios
- **CPU / memory saturation** on EC2/ECS.
- Inefficient code, blocking I/O, or synchronous external calls.
- **Thread/connection pool exhaustion**.
- Insufficient instances / no Auto Scaling.
- Chatty **N+1 calls** to the DB or downstream services.
- No caching → repeated expensive computation.
- Garbage collection (JVM) pauses.

### How to Troubleshoot
- **CloudWatch**: `CPUUtilization`, `MemoryUtilization` (CloudWatch agent), ECS `CPUReservation`.
- **AWS X-Ray**: trace request through the app; find the slow segment (DB call? external API?).
- **CloudWatch Logs Insights**: parse app logs for slow endpoints, GC pauses, timeouts.
- **Container Insights** (ECS/EKS) for pod/task-level metrics.
- Check **connection pool** metrics (DB connections, HTTP client pools).

### AWS Recommendations
- **Auto Scaling** on CPU / request count / custom metrics; use **target tracking** policies.
- **Right-size** instances (Compute Optimizer recommendations); use Graviton for price/perf.
- Add **caching**:
  - **ElastiCache (Redis/Memcached)** for session/DB query caching.
  - **DAX** for DynamoDB.
- Make external/downstream calls **asynchronous** or parallel; add timeouts + retries with backoff.
- Tune **connection pooling** (e.g., HikariCP) and reuse HTTP clients.
- Offload heavy work to **queues (SQS)** / async processing.
- Use **VPC Endpoints** to reduce hops to AWS services.

---

## 5. Tier 3 — Database / Data Layer (most common latency source)

### Possible Scenarios
- **Missing or inefficient indexes** → full table scans.
- **Slow / expensive queries**, N+1 patterns.
- **Lock contention / blocking transactions**.
- **Connection exhaustion** (too many open connections).
- Under-provisioned instance (CPU, IOPS, memory).
- **Read replica lag** or reads hitting the primary.
- Hot partitions (DynamoDB) / throttling.
- Cold buffer cache after failover.

### How to Troubleshoot
- **RDS/Aurora Performance Insights** — identify top SQL by load, wait events (CPU, I/O, locks).
- **CloudWatch RDS metrics**: `ReadLatency`, `WriteLatency`, `CPUUtilization`, `DatabaseConnections`, `DiskQueueDepth`, `FreeableMemory`, `ReplicaLag`.
- **Enhanced Monitoring** — OS-level metrics (per-process).
- **Slow query logs** → CloudWatch Logs.
- Run `EXPLAIN` / `EXPLAIN ANALYZE` on slow queries.
- **DynamoDB**: check `ThrottledRequests`, `ConsumedRead/WriteCapacity`, `SuccessfulRequestLatency`, hot-key analysis via CloudWatch Contributor Insights.

### AWS Recommendations
- **Add/optimize indexes**; rewrite N+1 into batched queries.
- **Read replicas** to offload reads; route reads to replicas (Aurora reader endpoint).
- **Connection pooling** with **RDS Proxy** — prevents connection storms, reduces failover time.
- Add a **caching layer** — ElastiCache (Redis) / DAX (DynamoDB) to serve hot reads.
- **Scale up/out**: larger instance, Aurora Auto Scaling replicas, or **Aurora Serverless v2**.
- Provision adequate **IOPS** (io1/io2/gp3); monitor `DiskQueueDepth`.
- **DynamoDB**: on-demand mode or auto scaling; design partition keys to avoid hot partitions; use DAX.
- Use **Multi-AZ** to reduce failover-induced latency spikes.

---

# PART B — Serverless Architecture Latency Troubleshooting

```
Client → CloudFront → API Gateway / AppSync → Lambda → DynamoDB / RDS Proxy / S3
                                                     ↘ SQS / SNS / EventBridge / Step Functions
```

---

## 6. Layer 1 — API Gateway / AppSync

### Possible Scenarios
- API Gateway integration latency (backend slow).
- Missing caching → every request hits backend.
- **Throttling** (account/stage rate limits) adding retries.
- Payload/transformation (VTL mapping) overhead.
- Regional vs Edge-optimized endpoint mismatch.

### How to Troubleshoot
- **API Gateway CloudWatch metrics**:
  - `Latency` (total, incl. backend) vs `IntegrationLatency` (backend only) — the difference is API Gateway overhead.
  - `4XXError`, `5XXError`, `ThrottleCount`, `CacheHitCount` / `CacheMissCount`.
- Enable **execution & access logging**; enable **X-Ray tracing** on the API.

### AWS Recommendations
- Enable **API Gateway caching** for cacheable GET responses.
- Use **regional endpoints + CloudFront** for control, or **edge-optimized** for global reach.
- Increase **throttling limits** / request **service quota** increases if throttled.
- Keep **mapping templates** minimal; prefer **proxy integration**.
- For GraphQL, use **AppSync caching** and avoid over-fetching.

---

## 7. Layer 2 — Lambda / Compute (the serverless hot spot)

### Possible Scenarios — the big one: **Cold Starts**
- **Cold start** latency (init of runtime + dependencies + VPC ENI attach).
- Under-provisioned **memory** (memory also scales CPU).
- Large deployment package / heavy init code.
- **VPC-attached Lambda** ENI cold start (historically slow; improved with Hyperplane).
- Downstream call latency (DB, external API) inside the function.
- **Concurrency limits** → throttling (`429`) and retries.
- Inefficient code, synchronous chaining of functions.

### How to Troubleshoot
- **Lambda CloudWatch metrics**: `Duration`, `InitDuration` (cold start), `Throttles`, `ConcurrentExecutions`, `Errors`, `IteratorAge` (for streams).
- **AWS X-Ray** — see `Initialization` segment (cold start) vs handler execution vs downstream calls.
- **CloudWatch Logs** — `REPORT` lines show `Init Duration`, `Duration`, `Max Memory Used`.
- **Lambda Power Tuning** (open-source Step Functions tool) — find optimal memory/cost/speed.

### AWS Recommendations
- **Reduce cold starts**:
  - **Provisioned Concurrency** — keeps functions warm (predictable low latency).
  - **SnapStart** (Java) — snapshot-restore to cut init time.
  - Minimize package size; lazy-load heavy SDK clients; move init outside the handler.
  - Choose faster runtimes / **Graviton (arm64)**.
- **Tune memory** — more memory = more CPU; often *reduces* both latency AND cost. Use Power Tuning.
- **Reuse connections** across invocations (declare DB/HTTP clients outside handler).
- Use **RDS Proxy** if Lambda talks to RDS (avoids connection storms).
- **Avoid synchronous function chaining** — use **Step Functions** or **event-driven (SQS/EventBridge)** patterns.
- Set appropriate **timeouts** and **reserved/provisioned concurrency**.
- Prefer VPC only when needed; use **VPC endpoints** for AWS service calls.

---

## 8. Layer 3 — Data Stores (Serverless)

### DynamoDB
- **Scenarios**: throttling, hot partitions, large item scans, eventually-consistent reads, no caching.
- **Troubleshoot**: `ThrottledRequests`, `SuccessfulRequestLatency`, consumed vs provisioned capacity; Contributor Insights for hot keys.
- **Recommendations**: good partition-key design; **on-demand** or auto scaling; **DAX** cache; use **Query** not **Scan**; project only needed attributes; **Global Tables** for multi-region reads.

### RDS/Aurora (behind Lambda)
- **Scenarios**: connection storms from many concurrent Lambdas.
- **Recommendations**: **RDS Proxy** (pooling), **Aurora Serverless v2** for elastic scaling.

### S3
- **Scenarios**: many small object requests, no CDN.
- **Recommendations**: front with **CloudFront**; use **S3 Transfer Acceleration** for uploads; request parallelism / byte-range fetches for large objects.

---

## 9. Layer 4 — Async / Integration Services

### Possible Scenarios
- **SQS** message backlog / processing delay (`ApproximateAgeOfOldestMessage`).
- **Step Functions** slow states / excessive wait states.
- **EventBridge** rule evaluation / delivery delay.
- **Kinesis** `IteratorAge` growing (consumer falling behind).

### How to Troubleshoot
- **SQS**: `ApproximateAgeOfOldestMessage`, `ApproximateNumberOfMessagesVisible`.
- **Step Functions**: execution history, per-state duration; enable X-Ray.
- **Kinesis**: `GetRecords.IteratorAgeMilliseconds`.

### AWS Recommendations
- Scale consumers (Lambda **batch size**, **parallelization factor**, more shards).
- Use **SQS batch processing** and appropriate **visibility timeout**.
- Optimize **Step Functions** — parallel states, avoid unnecessary waits, use Express workflows for high-volume/short tasks.
- Add **dead-letter queues (DLQ)** to prevent retry storms.

---

## 10. Cross-Cutting — Network & Security-Induced Latency

Applies to **both** architectures:

| Cause | Symptom | Fix |
|-------|---------|-----|
| **Cross-AZ / cross-region hops** | Extra RTT per call | Co-locate resources; cross-zone LB; regional endpoints |
| **NAT Gateway bottleneck** | Latency/packet drops on egress | Use **VPC Endpoints** for AWS services (bypass NAT) |
| **VPC-attached Lambda cold ENI** | High init latency | Keep out of VPC unless required; use endpoints |
| **TLS handshake** | Per-connection overhead | Keep-alive, connection reuse, edge termination |
| **DNS lookups** | First-request delay | Cache DNS, reuse connections |
| **WAF rule evaluation** | Small edge overhead | Optimize/limit rules; use managed rule groups efficiently |
| **Encryption/KMS calls** | Latency on frequent decrypts | Cache data keys (envelope encryption) |

---

## 11. Multi-Layer Caching Strategy

Caching is the single highest-leverage latency fix. Cache **as close to the user as possible** and cascade down:

| Cache Layer | AWS Service | Use For | Notes |
|-------------|-------------|---------|-------|
| **Browser / Client** | Cache-Control headers | Static assets, immutable content | Cheapest — no request at all |
| **Edge / CDN** | **CloudFront** | Static + cacheable dynamic responses | Set TTLs, cache keys carefully |
| **API** | **API Gateway cache**, **AppSync cache** | Repeatable GET/GraphQL responses | Per-stage, keyed on params |
| **Application** | **ElastiCache (Redis/Memcached)** | Sessions, computed results, DB query results | Use TTL + invalidation strategy |
| **Database** | **DAX** (DynamoDB), read replicas | Hot reads | DAX = microsecond reads |
| **Compute reuse** | Lambda execution context | Connections, SDK clients, config | Declare outside the handler |

### Caching Best Practices
- **Cache invalidation** is the hard part — prefer **TTL-based expiry**; use write-through/lazy-loading patterns deliberately.
- Watch **cache hit ratio** — a low ratio means the cache isn't helping (wrong key or TTL too short).
- Beware **thundering herd** on cache expiry — use jittered TTLs, request coalescing, or cache warming.
- Cache **negative results** (e.g., 404s) briefly to protect the backend.

---

## 12. Resilience Patterns That Reduce Tail Latency

Latency spikes often come from waiting on slow/failing dependencies. Apply these:

- **Timeouts everywhere** — never wait indefinitely; set aggressive, per-dependency timeouts (client, HTTP, DB, Lambda).
- **Retries with exponential backoff + jitter** — but cap retries; blind retries amplify load and *worsen* latency (retry storms).
- **Circuit breakers** — fail fast when a dependency is unhealthy instead of piling up slow calls.
- **Bulkheads** — isolate resource pools so one slow dependency doesn't starve others.
- **Load shedding / throttling** — reject excess load early (API Gateway throttling, `429`) to protect p99.
- **Asynchronous decoupling** — offload slow work to **SQS/EventBridge/Step Functions** so the user path stays fast.
- **Request hedging** — for read-heavy critical paths, issue parallel requests and take the first response.
- **Graceful degradation** — serve cached/partial results when a downstream is slow.

> **Interview point:** *A retry storm can turn a small DB blip into a full outage. Always pair retries with backoff, jitter, a retry cap, and a circuit breaker.*

---

## 13. Latency SLOs, Alarming & Proactive Monitoring

Don't wait for users to report slowness — detect it first.

- **Define SLIs/SLOs** — e.g., "p99 API latency < 300 ms for 99.9% of 5-min windows."
- **Alarm on percentiles**, not averages — CloudWatch supports **p90/p99** statistics and **anomaly detection** alarms.
- **CloudWatch Synthetics canaries** — simulate user journeys from multiple regions continuously.
- **CloudWatch RUM** — capture real browser/device latency and geographic breakdown.
- **ServiceLens / X-Ray** dashboards — correlate traces, metrics, and logs in one place.
- **Composite alarms** — reduce noise by combining conditions (e.g., high latency AND high error rate).
- **Contributor Insights** — surface the top offenders (users, keys, endpoints) driving latency.
- **Log-based metrics** — extract latency from logs with Logs Insights + metric filters.

---

## 14. Common Anti-Patterns & Pitfalls

| Anti-Pattern | Why It Hurts Latency | Better Approach |
|--------------|----------------------|-----------------|
| Optimizing **average** latency | Hides the tail; users feel p99 | Track and alarm on **p99** |
| **N+1 queries** | 1 request → hundreds of DB calls | Batch / join / eager-load |
| New DB **connection per request** | Handshake overhead, exhaustion | Pooling (HikariCP) / **RDS Proxy** |
| **Chaining Lambdas synchronously** | Latency stacks, double cold starts | Step Functions / event-driven |
| **VPC Lambda** when not needed | ENI cold-start penalty | Keep out of VPC; use VPC endpoints |
| Heavy work **inside the handler** | Repeated init cost | Move init to execution-context scope |
| **No caching** | Every request hits the backend | Multi-layer caching (Section 11) |
| **Scan** instead of Query (DynamoDB) | Reads entire table | Query on partition key + GSIs |
| **Under-provisioned Lambda memory** | Low CPU → slow execution | Tune with Power Tuning |
| Blind **retries without backoff** | Retry storms, cascading slowdown | Backoff + jitter + circuit breaker |
| **Synchronous** external API calls | Blocks the request path | Async / parallel / queue |

---

## 15. Worked Example — "The App Got Slow After a Traffic Spike"

A realistic walkthrough an interviewer might ask you to reason through:

1. **Symptom** — p99 API latency jumped from 200 ms to 4 s; error rate rising.
2. **Isolate** — X-Ray service map shows time is spent in the **database** segment, not the app or LB.
3. **Metrics** — RDS `DatabaseConnections` is maxed; `CPUUtilization` at 95%; Performance Insights shows one SQL dominating with `Lock:transactionid` waits.
4. **Root cause** — Traffic spike caused (a) connection exhaustion and (b) a missing index forcing full scans under contention.
5. **Immediate fix** — Add **RDS Proxy** to pool connections; add the missing **index**; route reads to a **read replica**.
6. **Prevent recurrence** — Add **ElastiCache** for hot reads; enable **Aurora Auto Scaling** replicas; set a **p99 CloudWatch alarm** and a Synthetics canary.
7. **Verify** — p99 back to ~180 ms; connections stable under load test.

> This demonstrates the full loop: **measure → isolate → hypothesize → fix → prevent → verify.**

---

## 16. Quick Diagnostic Decision Tree

```
Is latency high?
│
├─ Where? → Use X-Ray service map
│
├─ Edge/DNS slow?      → CloudFront cache hit ratio, Route 53, Global Accelerator
├─ ALB TargetResponseTime high? → problem is DOWNSTREAM (app/DB), not the LB
├─ App CPU/mem high?   → Auto Scale, right-size, add caching
├─ API GW Latency ≫ IntegrationLatency? → API Gateway overhead (caching, mapping)
├─ Lambda InitDuration high? → Cold start → Provisioned Concurrency / SnapStart / smaller pkg
├─ DB ReadLatency / Perf Insights hot SQL? → Index, read replica, RDS Proxy, cache
├─ DynamoDB ThrottledRequests? → Capacity/on-demand, partition key, DAX
└─ SQS/Kinesis age growing? → Scale consumers, batch tuning
```

---

## 17. Summary — Where to Look by Layer

| Layer | 3-Tier | Serverless | Primary Metric | First Fix |
|-------|--------|-----------|----------------|-----------|
| Edge | CloudFront/Route 53 | CloudFront/Route 53 | CacheHitRate | CDN caching |
| Entry | ALB | API Gateway/AppSync | `TargetResponseTime` / `Latency` vs `IntegrationLatency` | Caching, health, scaling |
| Compute | EC2/ECS | Lambda | `CPUUtilization` / `Duration`,`InitDuration` | Auto Scale / Provisioned Concurrency |
| Data | RDS/Aurora | DynamoDB/RDS Proxy | `ReadLatency` / `ThrottledRequests` | Index, cache, replica, RDS Proxy |
| Async | SQS/SNS | SQS/Step Functions/Kinesis | Queue age / IteratorAge | Scale consumers, batching |
| Cross-cutting | Network/KMS/TLS | Network/KMS/VPC | Flow Logs / X-Ray | VPC Endpoints, connection reuse |

---

## 18. Interview Talking Points — Detailed Scenarios

Each scenario is structured as **Problem → Detection → Fix → Validation** so you can walk an interviewer through the full troubleshooting loop.

---

### 18.1 "The app is slow" — general triage

- **Problem:** Users report the application feels slow; no clear layer identified yet.
- **Detection:**
  - Pull **CloudWatch** dashboards and look at **p99/p90** latency (never averages — they hide the tail).
  - Open the **AWS X-Ray service map** to see which segment (edge → LB → app → DB → external) owns the time.
  - Correlate with deploy/traffic events (did latency rise after a release or a spike?).
- **Fix:** Depends on the isolated layer — but the *method* is: form one hypothesis, change **one variable**, re-measure.
- **Validation:** Re-run the same measurement window; confirm p99 returned to baseline; run a **load test** to confirm under stress.

---

### 18.2 Is it the Load Balancer or the App?

- **Problem:** High end-to-end latency; unclear whether the ALB/API Gateway or the backend is responsible.
- **Detection:**
  - **ALB:** compare `Latency` (total) vs `TargetResponseTime` (backend). High `TargetResponseTime` ⇒ backend is slow, LB is fine.
  - **API Gateway:** compare `Latency` vs `IntegrationLatency`. A large gap ⇒ API Gateway overhead (mapping templates, no caching); if both are high ⇒ backend.
  - Check target group health and `RejectedConnectionCount`.
- **Fix:**
  - Backend slow → scale/optimize the app or DB (see below).
  - LB overhead → enable **cross-zone LB**, fix health checks, add **API Gateway caching**, simplify mapping templates.
- **Validation:** `TargetResponseTime` / `IntegrationLatency` drops; 5XX and rejected connections return to zero.

---

### 18.3 Serverless Cold Starts (biggest serverless issue)

- **Problem:** Intermittent high latency on the first request / after idle periods; especially on Java/.NET or VPC-attached Lambdas.
- **Detection:**
  - **Lambda** `InitDuration` metric and the `REPORT` line `Init Duration` in CloudWatch Logs.
  - **X-Ray** shows a distinct `Initialization` segment before the handler runs.
  - Correlate spikes with low `ConcurrentExecutions` (scaling from zero).
- **Fix:**
  - **Provisioned Concurrency** to keep functions warm; **SnapStart** for Java.
  - Shrink deployment package; move SDK/DB client init **outside the handler**; lazy-load heavy deps.
  - Prefer **Graviton (arm64)**; keep Lambda **out of the VPC** unless required (use VPC endpoints).
- **Validation:** `InitDuration` occurrences drop sharply; p99 stabilizes; cost check that Provisioned Concurrency is right-sized.

---

### 18.4 Lambda + RDS Connection Storms

- **Problem:** Under load, hundreds of concurrent Lambdas each open a DB connection → RDS connection exhaustion, timeouts, rising latency.
- **Detection:**
  - **RDS** `DatabaseConnections` near the instance max; connection errors in logs.
  - **X-Ray** shows time spent establishing DB connections, not querying.
- **Fix:**
  - Add **RDS Proxy** to pool and reuse connections and smooth failover.
  - Reuse the connection object across invocations (declare outside the handler).
  - Consider **Aurora Serverless v2** for elastic capacity.
- **Validation:** `DatabaseConnections` flattens well below max; query latency stable under a concurrency load test.

---

### 18.5 Database — the most common 3-tier latency source

- **Problem:** App is slow; the bottleneck is the data tier (slow queries, contention, or exhaustion).
- **Detection:**
  - **RDS/Aurora Performance Insights** → top SQL by load and **wait events** (CPU, I/O, `Lock:*`).
  - **CloudWatch** `ReadLatency`, `WriteLatency`, `CPUUtilization`, `DatabaseConnections`, `DiskQueueDepth`, `ReplicaLag`.
  - **Slow query logs**; run `EXPLAIN ANALYZE` on suspect queries.
- **Fix:**
  - **Add/optimize indexes**; rewrite **N+1** into batched/joined queries.
  - Offload reads to **read replicas** (Aurora reader endpoint); add **ElastiCache/DAX** for hot reads.
  - **RDS Proxy** for pooling; scale instance / IOPS (gp3/io2); **Aurora Auto Scaling**.
- **Validation:** Performance Insights shows the hot SQL gone; `ReadLatency` and CPU down; cache hit ratio high; load test passes.

---

### 18.6 Lambda Memory / CPU Under-Provisioning

- **Problem:** Lambda `Duration` is high even without cold starts; function is CPU-bound.
- **Detection:**
  - `REPORT` line shows `Max Memory Used` near the configured limit and long `Duration`.
  - CPU-bound work (parsing, compression, crypto) dominates the X-Ray handler segment.
- **Fix:** Increase **memory** (which proportionally increases CPU); find the sweet spot with **Lambda Power Tuning** (often reduces *both* latency and cost).
- **Validation:** `Duration` drops at the tuned memory setting; cost-per-invocation compared before/after; p99 improves.

---

### 18.7 DynamoDB Throttling / Hot Partitions

- **Problem:** Spiky latency and `429`/throttling errors on DynamoDB reads or writes.
- **Detection:**
  - **CloudWatch** `ThrottledRequests`, `ConsumedRead/WriteCapacityUnits` vs provisioned, `SuccessfulRequestLatency`.
  - **Contributor Insights** to find hot partition keys.
- **Fix:**
  - Switch to **on-demand** capacity or enable **auto scaling**.
  - Redesign the **partition key** to spread load; add **GSIs**; use **Query** not **Scan**.
  - Add **DAX** for microsecond cached reads.
- **Validation:** `ThrottledRequests` at zero; latency flat under load; even key distribution in Contributor Insights.

---

### 18.8 Edge / Network-Induced Latency

- **Problem:** Users in distant regions experience high RTT; low cache utilization; NAT bottlenecks.
- **Detection:**
  - **CloudWatch RUM / Synthetics** show latency varying by geography.
  - **CloudFront** `CacheHitRate` low; **VPC Flow Logs** show NAT egress hops.
- **Fix:**
  - Put **CloudFront** in front; tune cache TTLs and cache keys; enable HTTP/2/3, compression, keep-alive.
  - **Route 53 latency-based routing**; **Global Accelerator** for TCP/UDP.
  - **VPC Endpoints** to bypass NAT for AWS service calls.
- **Validation:** `CacheHitRate` rises; per-region canary latency drops; NAT data-processing charges fall.

---

### 18.9 Retry Storms / Cascading Slowdown

- **Problem:** A brief downstream blip triggers aggressive retries that amplify load and turn a small issue into an outage.
- **Detection:**
  - Sudden multiplication of request counts to a struggling dependency; rising `ConcurrentExecutions`/queue depth; correlated error+latency spikes.
- **Fix:** **Exponential backoff + jitter**, a **retry cap**, and a **circuit breaker** to fail fast; add **load shedding** (throttling / `429`) and **DLQs**.
- **Validation:** Request amplification gone during a fault-injection test; p99 stays bounded when a dependency is degraded.

---

> **Closing statement:** *"Latency troubleshooting on AWS is measure-first: use X-Ray to pinpoint the slow segment, read the right CloudWatch metric for that layer, apply the layer-specific fix — CDN caching at the edge, scaling and caching at compute, indexing/replicas/RDS Proxy at the data layer, Provisioned Concurrency for Lambda cold starts — then validate against baseline p99 and a load test to confirm the fix holds under stress."*
