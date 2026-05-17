# Sr. AWS DevOps Engineer — Complete Interview Q&A
### Combined from Two Real Mock Interviews | Tech with Ajit

---

> **Interviewer:** Ajit (14 years experience, Technology Head)
>
> **Interview 1 — Hussein Lokhandwala** | 4.5 years experience | AWS, Kubernetes, Terraform, Azure certified
>
> **Interview 2 — Hershel Kung** | 5 years experience | AWS, GCP, Kubernetes, Terraform, GitLab, Argo CD

---

## 1. 🏗️ Architecting a Highly Available & Fault-Tolerant Application on AWS

**Q: How would you architect a fully available and fault-tolerant application on AWS?**

---

### Hussein's Approach *(Data Analytics Application)*

- **EKS** for compute with clusters in the primary and a secondary DR region
- **VPC** with public and private subnets — compute and databases in private subnets; ALB in public subnets; NACLs and security groups configured accordingly
- **S3** with **cross-region replication** for storage
- **RDS** with **cross-region read replicas**; prefer **DynamoDB** (global tables) where applicable
- **CloudWatch Logs** integrated with **Prometheus and Grafana** for monitoring
- **Route 53** with failover routing — if primary region fails, traffic automatically routes to DR region
- **AWS Secrets Manager** for secrets
- Fully automated with **Terraform pipelines** and **GitOps using Argo CD**

---

### Hershel's Approach *(Global E-Commerce Application)*

Hershel asked a clarifying question first — *"Do you have specific conditions or requirements?"* — before designing, which the interviewer noted positively.

Key priorities identified: **low latency globally** + **security**

Architecture design:
- **Three-tier VPC** (public subnet for ALB, private for compute and databases) — skipped basics, went straight to interesting parts
- **CloudFront CDN** — first priority for low latency delivery of static assets globally
- **DynamoDB Global Tables** — low latency reads/writes across all regions natively
- **EKS with node groups across multiple regions** — e.g., one instance in US, one in Mumbai — managed centrally via Helm and Argo CD
- **Microservices on EKS** — e-commerce naturally splits into services: cart, catalog, payments, reviews, customer service — each independently deployable
- **HPA and VPA** (Horizontal/Vertical Pod Autoscalers) + **Cluster Autoscaler** for traffic spike handling
- **AWS Secrets Manager** for sensitive data (API keys, payment gateway credentials); **Parameter Store** for non-sensitive config (URLs, endpoints)
- **S3** for backups of databases and application state
- **CloudWatch + CloudTrail** for logging and audit trails
- **VPC Flow Logs** for tracking all incoming/outgoing traffic
- **IAM least privilege** enforced across all services
- Multi-account setup considered — separate accounts per region or business unit, with inter-account connectivity enabled

---

**Interviewer's key follow-up — database caching for infrequently updated data:**

**Hershel's Answer:**
- For **NoSQL** → use **DynamoDB Global Tables** — it caches and replicates across regions automatically, fully managed
- For **MySQL/PostgreSQL** → use **RDS with read replicas** across regions (near real-time sync); good for data updated once a month
- Add **ElastiCache** (Redis) in front of the database for frequently accessed queries — but filter out very stale data (e.g., month-old data) from cache to control costs
- Decision depends on **how critical the response time is** — if users can wait 1–2 seconds for month-old data, replicas suffice without the expense of ElastiCache

---

## 2. ⚖️ EKS vs. ECS — When to Use Each?

**Q: Why EKS over ECS? When is ECS the better choice?**

---

### Hussein's Answer

- **EKS preferred** for complex, security-heavy architectures — provides RBAC rules, network policies, and GitOps-friendly deployments via Argo CD
- **ECS is simpler** — sufficient for straightforward deployments where everything is managed within AWS using IAM roles and ALB routing; no need for ingress or service mesh
- **ECS when:** small scale, fewer services, AWS-native tooling is sufficient, no need for Kubernetes constructs
- **Cost:** ECS is cheaper — you pay only for what you use; EKS charges ~$0.10/hr for the control plane even before nodes run

### Hershel's Answer

- **ECS is like a managed container runner** — spin up containers by specifying task definitions; AWS handles everything. Perfect for batch jobs, small-scale, simple request/response workloads
- **EKS gives more control** — manage scaling, configuration, and cross-region deployments more precisely
- For a **global e-commerce app** needing deployments across 10 regions — ECS would require duplicating the same resources in each region independently, creating config drift over time; EKS lets you manage all of it centrally through Helm and Argo CD
- **Additional EKS advantage:** can manage **on-premises workloads** as well through EKS Anywhere — centralized control over hybrid infrastructure

**Interviewer's summary:** Both are valid — choose based on scale, complexity, and team familiarity.

---

## 3. 🌐 Networking — Public vs. Private Subnets & Hybrid Connectivity

**Q: What configuration makes a subnet public or private? How do you connect on-premises to AWS?**

### Hussein's Answer

- **Public subnet** = route table points traffic through an **Internet Gateway (IGW)**
- **Private subnet** = route table points outbound traffic through a **NAT Gateway** (which itself sits in the public subnet)
- The **IGW is attached to the VPC**, not the subnet
- Outbound internet access for a private subnet is configured in the **private subnet's own route table**, pointing to the NAT Gateway

**On-prem to AWS connectivity options:**
1. **AWS Direct Connect** — for large-scale data transfer; dedicated physical link; most reliable
2. **Site-to-Site VPN** — creates a Customer Gateway + AWS Gateway; cheaper, good for moderate workloads
3. **Transit Gateway** — can also bridge on-prem to AWS
4. *(Hussein mentioned an "AWS Hub" service he couldn't recall by name — likely AWS Cloud WAN)*

---

## 4. 🔗 VPC Peering vs. Transit Gateway

**Q: Why is Transit Gateway needed when VPC Peering exists?**

### Hussein's Answer

- **VPC Peering** = direct non-transitive link between two VPCs; cheaper for simple setups
- **Non-transitive problem** — A↔B and B↔C doesn't allow A↔C; you'd need separate peering for every pair
- **Transit Gateway** = hub-and-spoke model — one TGW connects all VPCs; all can communicate through it
- **Cost** — VPC Peering data transfer is cheaper than Transit Gateway; use TGW when managing 5+ VPCs or needing transitive routing

---

## 5. 🛡️ Disaster Recovery Strategies

**Q: Design a DR solution. How do you balance cost vs. uptime?**

---

### Hussein's Three-Tier DR Model *(for an e-commerce site)*

| Scenario | Setup | Downtime | Cost |
|---|---|---|---|
| **Tier 1 — Backup & Restore** | Backup EKS, app, RDS; spin everything up in DR region on failure | Minutes to hours | Lowest |
| **Tier 2 — Warm Standby** | Read replica in DR region; EKS cluster with nodes scaled to zero | Minimal | Medium |
| **Tier 3 — Hot Standby** | Fully scaled EKS + RDS read replica always running in DR; Route 53 failover auto-diverts traffic when ALB health check fails | Zero | Highest |

**For global multi-region support (US + India):** CloudFront for static content, global load balancer, read replicas per region, ElastiCache per region for caching. (*Hussein missed mentioning DynamoDB Global Tables for active-active writes — Ajit flagged this.*)

---

### Hershel's DR Approach *(cost-sensitive e-commerce)*

Hershel asked the cost and criticality questions upfront before proposing a solution:

- **Scenario 1 — Cold DR (low cost, some downtime and data loss):** Spin up a brand new application when the primary goes down; cheapest but not suitable for critical apps
- **Scenario 2 — Moderately Critical:** Leverage AWS global services already in use (DynamoDB Global Tables, RDS replicas, EKS spread across regions); Kubernetes auto-scaling will spin up new nodes if a region fails — some delay but manageable
- **Scenario 3 — Super Critical:** Warm replica of the **entire infrastructure** in another region, always running; Route 53 routes traffic to the standby region the moment the primary fails; no data loss, no downtime

**Hershel's smart cost-saving insight:**
> Not all microservices in an e-commerce app are equally critical. *Creating orders, processing payments, managing users* → **warm standby DR**. *Customer reviews, complaint forms, feedback* → **cold DR**. Splitting DR tiers per service drastically reduces cost while keeping the critical path always available.

---

## 6. 🔐 Security Best Practices

**Q: What security measures do you implement? What's the bare minimum for a budget-constrained customer?**

---

### Hussein's Answer

**Enterprise-grade:**
- **AWS Shield + WAF** — request filtering and DDoS protection
- **VPC Flow Logs** — trace all incoming/outgoing traffic
- **Encrypt** all RDS and EBS volumes with KMS
- **CloudTrail** for full audit logging
- **AWS Config rules** + Lambda auto-remediation for misconfigurations
- **CloudWatch Alarms** for database failures, CPU spikes, etc.

**Budget-constrained (bare minimum):**
- **Security Groups** — restrict to minimum required ports; never open to 0.0.0.0/0
- **NACLs** — subnet-level filtering
- **IAM Least Privilege** — granular permission sets in IAM Identity Center
- **No manual access** — all infrastructure changes through Terraform pipelines only; full automation enforces traceability

### Hershel's Security Additions

- **IAM Least Access** across all services
- **NAT Gateways and Security Groups** as the baseline
- **CloudTrail** for all API actions; VPC Flow Logs for network activity
- **No manual access to the cloud** — require Terraform for all provisioning; temporary access (1 hour) must have a documented business justification and is fully logged
- **Secrets Manager** for all sensitive credentials; **Parameter Store** for non-sensitive config values

---

## 7. 🗄️ Database Access Without Exposing Subnets

**Q: A third party needs to access your RDS. How do you grant access securely without a public subnet?**

### Hussein's Answer

- Use **IAM Database Authentication** — IAM role is the only entity that can authenticate; no username/password needed
- Restrict the **Security Group** to allow only the specific third-party IP — not open to the internet
- Enforce **TLS/SSL** — only SSL-certified connections accepted by the database

**Bonus:** AWS Inspector scans EC2 instances and all resources for vulnerabilities and CVEs.

---

## 8. 🏛️ The 6 Pillars of the AWS Well-Architected Framework

**Q: List and summarize each pillar.**

### Hussein's Answer

| Pillar | Summary |
|---|---|
| **Cost Optimization** | Choose the right instance types and services; avoid over-provisioning |
| **Security** | Confidentiality, integrity, threat detection; enable AWS security services |
| **Operational Excellence** | CI/CD, automated backups, runbooks; systematic deployment and operations |
| **Performance Efficiency** | Right-sizing instances and services for the workload |
| **Reliability** | Multi-AZ, DR, backups and recovery |
| **Sustainability** | Minimize environmental impact; AWS manages at the infrastructure level |

*Hussein correctly noted that Sustainability was added later, bringing the total from 5 to 6 pillars.*

### Hershel's Addition

Hershel recommended using the **AWS Well-Architected Tool** in the console to run formal WAF reviews — it asks structured questions across all pillars and identifies specific areas for improvement in both architecture and cost.

---

## 9. 🆔 Cross-Account Access & IAM

**Q: How do you give an EC2 in Account A access to an S3 bucket in Account B?**

### Hussein's Answer

1. In Account A's IAM role → add policy for **sts:AssumeRole** with Account B's role ARN
2. In Account B's IAM role → **Trust Policy** allows Account A's role ARN to assume it
3. In Account B's **S3 bucket policy** → explicitly grant Account A's role get/put permissions
4. EC2 in Account A assumes Account B's role via **STS**, gets temporary credentials, accesses S3

**Common pitfall:** S3 bucket policies must explicitly allow the cross-account role — IAM role permissions alone are not sufficient. For **KMS keys** — additional key policy read access must also be granted in Account B.

---

## 10. 💰 Cloud Cost Optimization (FinOps)

**Q: How do you achieve significant cost reductions (e.g., 30%) without impacting compliance?**

---

### Hussein's Strategy

1. Never compromise **compliance and security** while cutting cost
2. Identify the **highest-cost service** and find alternatives
3. **Reserved Instances** (upfront payment) for stable, long-running workloads — up to 72% savings
4. **Spot Instances** for batch jobs (with retry logic)
5. Upgrade storage — **GP2 → GP3** (better performance, cheaper price)
6. Right-size **instance families** (GPU-intensive, memory-intensive, IOPS-intensive)
7. Switch instance types within family — e.g., T3 → T3a (same config, lower cost)
8. Reduce **RDS/EBS backup retention** periods
9. Move cold S3 data to **Glacier or Archive** storage class
10. Use **Cost Explorer** and **Trusted Advisor** (note: Trusted Advisor is not free — requires a paid support plan)

**On Spot Instances for scheduled batch jobs:** Yes — use Spot if retry logic exists; switch instance families if preferred type is unavailable; always compare cost before deciding.

---

### Hershel's Strategy *(Healthcare Context)*

- **Reserved Instances** for critical, long-running workloads (1–3 year commitment)
- **Spot Instances** for non-critical or fault-tolerant services
- Reduce **backup retention periods** to the minimum required by compliance
- Delete old, unnecessary **CloudWatch log groups** — these silently accumulate and drive up bills
- **Clean up infrastructure regularly** — old snapshots, unattached EBS volumes, orphaned resources
- Use **AWS Trusted Advisor** for spend recommendations
- Perform a **Well-Architected Framework Review** — answers to WAF questions reveal specific efficiency and cost gaps
- **Honesty point:** Hershel noted that committing to a 30% cost reduction without impacting the application is unrealistic — cost reduction always carries trade-offs and requires careful analysis

---

## 11. 🛠️ Terraform vs. CloudFormation

**Q: When do you choose Terraform over CloudFormation? Would you still use Terraform for AWS-only infrastructure?**

---

### Hussein's Answer

**Terraform advantages:**
- Multi-cloud support (AWS + Azure + GCP)
- Modular, reusable code — write once, deploy to multiple environments and accounts
- Remote state in S3 + DynamoDB locking
- Massive community — easy to find solutions

**CloudFormation advantages:**
- New AWS services appear in CloudFormation first — Terraform providers lag behind
- Simpler for certain AWS-native resources (e.g., Lambda function creation is less verbose)

**Still choose Terraform for AWS-only?** Yes — modularization, community, multi-account/environment management, and remote state are reason enough.

---

### Hershel's Addition

- Also prefers Terraform for the same reasons
- Specifically called out **Helm** as *"Terraform for Kubernetes"* — manages K8s deployments the same way Terraform manages AWS resources
- Highlighted that when disaster strikes (e.g., someone deletes AWS resources), having Terraform + Helm + Argo CD means you can **rebuild the entire infrastructure from scratch** quickly — impossible if you used manual provisioning

---

## 12. 🔄 CI/CD Pipeline Design

**Q: Walk through the complete CI/CD journey for a large e-commerce application.**

### Hershel's Detailed Answer

**Step 1 — Choose the right tool based on where the code lives:**
- Code on GitHub → **GitHub Actions**
- Code on GitLab → **GitLab CI/CD**
- Code on AWS CodeCommit → **AWS native CI/CD** (CodePipeline, CodeBuild, CodeDeploy)
- *(Note: AWS CodeCommit was deprecated — Ajit flagged this during the interview)*

**Step 2 — Pipeline stages (using GitLab CI/CD as example):**
1. **Lint/Format check** — syntax validation, indentation, typo detection
2. **Unit tests / Terraform validate** — verify code correctness before building
3. **Docker image build** — containerize each microservice
4. **Push to ECR** — store the image in AWS Elastic Container Registry
5. **GitOps trigger** — Argo CD detects the new image, updates the deployment in Kubernetes automatically
6. **Manual approval gate** — for Terraform infrastructure changes: plan runs automatically, apply requires human review and approval
7. **Rollback jobs** — available in pipeline for quick revert if needed

**Authenticating a GitLab pipeline with AWS:**
- Use **OIDC-based trust relationship** (IRSA pattern) — GitLab's SaaS URL is trusted to assume a specific IAM role without storing long-lived access keys
- Secrets stored in GitLab's native secrets store — never printed in logs, inaccessible to pipeline runners

**Code/repo structure considered:**
- Monorepo vs. multi-repo for microservices
- Separate folders for each application + one parent folder for Terraform code
- Helm charts as the deployment unit — values.yaml per environment (dev/staging/prod)

---

## 13. 🔭 GitOps with Argo CD & Helm

**Q: Why do you need GitOps/Argo CD when you can just use plain Kubernetes YAML and Helm directly?**

### Hershel's Answer

- Helm is to Kubernetes what Terraform is to AWS — it's **IaC for Kubernetes**
- Helm charts define all K8s resources (deployments, services, config maps, secrets) in one package; `values.yaml` changes per environment without rewriting the chart
- **Argo CD + GitOps** makes deployments **declarative and auditable** — the Git repo is the single source of truth
- When you need to spin up a new region or recover from disaster — you don't need to manually recreate 50 YAML files; Argo CD pulls the Helm charts and deploys everything automatically
- **Security threat scenario:** If someone deletes AWS resources, Terraform + Argo CD + Helm means you rebuild the entire stack with minimal manual intervention

---

## 14. ⚖️ Load Balancers & API Gateway

**Q: ALB vs. NLB vs. Gateway LB. When do you use API Gateway instead of ALB?**

### Hussein's Answer

| Load Balancer | Layer | Use Case |
|---|---|---|
| **ALB** | Layer 7 | Path/host-based routing, web apps, microservices |
| **NLB** | Layer 4 | Ultra-low latency (trading platforms, real-time systems); static Elastic IP attachment |
| **Gateway LB** | Layer 3 | Security-first architectures; integrates with third-party network appliances; common in financial sector for request authentication before forwarding |
| **Classic LB** | — | Deprecated — no longer used |

**API Gateway vs. ALB:**
- **ALB** — for internal VPC traffic; simple path/host routing is sufficient
- **API Gateway** — when APIs need: third-party authentication, DDoS protection, TLS by default, rate limiting, Lambda integration; more security controls than ALB

### Hershel's Addition

- **ALB Ingress Controller** (AWS Load Balancer Controller) used in EKS for Kubernetes Ingress resources
- **NLB for Ingress** — when very specific network-level or latency requirements exist
- **Ingress Controller** eliminates the need for one ALB per service — one ALB routes to all services via Ingress rules (massive cost saving)

---

## 15. 🔍 Kubernetes Best Practices, Probes & Config Management

**Q: What are Kubernetes best practices? How do probes work? How do you manage configuration securely?**

### Hussein's Answer

**Best practices:**
1. Always set **resource requests and limits** for every pod
2. Use **ConfigMaps and Secrets** — never hardcode config in environment variables or application code
3. **Prometheus + Grafana** (via Helm) for custom metrics and dashboards
4. Set **alerts** in Prometheus/Grafana — e.g., node usage > 80%
5. Use **RBAC** — restrict access at the pod/namespace level
6. Use **IRSA** — give specific pods access to specific AWS resources, not the entire cluster

**Probes:**

| Probe | Trigger | Action on Failure |
|---|---|---|
| **Liveness** | Is the pod alive? | Kubernetes **restarts** the pod |
| **Readiness** | Is the pod ready for traffic? | Pod is **removed from Service endpoints** (no traffic), not restarted |

*Real example:* Pod is running but database is down — pod is "live" but not "ready." Readiness probe removes it from the load balancer until the database recovers.

**Kubernetes Secrets security:**
- Secrets are only base64 encoded — anyone with cluster access can decode them
- **RBAC** — restrict secret access to specific DevOps engineers or tech leads
- Best practice: use **External Secrets Operator** or **AWS Secrets Manager + CSI driver** to keep secrets outside the cluster entirely

**ConfigMap auto-update without pod restart:**
- If ConfigMap is mounted as an **environment variable** → pod reads it only at startup; restart required to pick up changes
- If ConfigMap is mounted as a **volume** → Kubernetes propagates updates automatically without restart
- Caveat: if the application itself only reads config at boot time, a code change is required regardless

### Hershel's Addition

**Observability layered approach:**
- **CloudWatch** — natively integrates with most AWS services; auto-logs available in the console for each service
- **Sidecar container with logging agent** — for compliance-heavy use cases (healthcare, finance) where every transaction must be logged; sidecar pulls deep application logs and pushes them to Grafana
- **CloudWatch detailed monitoring** — costs more but provides granular metrics; useful for dashboards and custom alarms
- **Request tracing** — Hershel acknowledged he'd need to look up the specific tool for in-depth pod-level request tracing (likely **AWS X-Ray** or **Jaeger**)

**Service Discovery (Istio/Service Mesh):**
- If microservices call each other directly → **Istio** or **service mesh** may be needed for service discovery, mutual TLS, traffic management
- For simpler apps where services call each other via Ingress or ClusterIP — service mesh is not necessary
- Decision: how interconnected are the services and how much of a latency/cost overhead is the service mesh worth?

---

## 16. 🚨 Incident Management — Production Application Down

**Q: A customer reports the application is not working. Walk through investigation and remediation.**

### Hussein's Answer

1. **Check for recent changes** — any deployments or Terraform changes just before the failure?
2. **Check infrastructure** — are EKS pods running? Is the database active? Which health check is failing?
3. **Stabilize first** — revert any recent deployment immediately; don't investigate root cause while users are impacted
4. **Root cause analysis** — review logs once stable; reproduce in dev/staging; fix and redeploy through environments

---

## 17. 🐳 Docker Best Practices & Image Security

**Q: What Docker best practices do you follow? How do you manage image size and security?**

### Hussein's Answer

**Image creation:**
1. **Multi-stage builds** — build in stage 1, copy only the artifact into the lightweight runtime image
2. **Lightweight base images** — Alpine or similar minimal images
3. **Pin base image versions** — never use `alpine:latest`; always a specific tag
4. **Non-root user** — create a dedicated user; run container as that user with access only to required folders
5. **Document every Dockerfile line** — comments for every instruction

**ECR image sprawl control:**
- **Lambda functions** on daily schedule — keep latest 5–7 images, auto-delete older ones
- Apply same approach to other resources — scale down EC2/RDS at night

**"Works on local, fails on EKS" debugging:**
- Port conflicts — port used locally but already taken on EKS
- Security Group — port may not be open in the SG
- Image tag mismatch — pushed one tag, deployment pulls another

**Security scanning pipeline:**
1. Developer pushes to GitHub
2. **SonarQube** — code quality check
3. **Docker build**
4. **Trivy** — image vulnerability scan; generates report; image only pushed to ECR if scan passes
5. Developers encouraged to run Trivy locally first to avoid burning pipeline minutes

---

## 18. ☸️ EKS Cluster Upgrades — Real-World Scenarios

**Q: How do you upgrade EKS clusters with zero downtime? What if the upgrade fails?**

---

### Hussein's Answer *(Standard Process)*

**Prerequisites:**
- Create a detailed runbook of upgrade steps
- Review K8s release notes — check for deprecated APIs
- Verify application compatibility with the target version
- All deployments fully pipeline-managed for easy rollback
- Back up application using Velero
- Test in lower environments first (dev → staging → prod)

**Upgrade process:**
1. EKS upgrades the **control plane** (managed by AWS)
2. **Managed node groups** — rolling update
3. **Self-managed node groups** — cordon + drain each node, upgrade, uncordon
4. Upgrade all **EKS add-ons** after nodes

**If upgrade fails:**
- **You cannot downgrade EKS** — once upgraded, no rollback
- Troubleshoot first — version issue? Deprecated API? Small fix possible?
- If not resolvable: spin up a **new cluster at the older version**, redeploy application, restore service, then investigate root cause

---

### Hershel's Real-World Story *(8 Clusters, 3 Versions Behind)*

One of the most impressive answers in either interview.

**The challenge:** Upgrade 8 EKS clusters, each with ~10 nodes and 50–60 pods per node, from a version that was 3 minor versions behind the target — with **as little downtime as possible**.

**Key insight most people don't know:**
> The **control plane must be upgraded one minor version at a time** (e.g., 1.24 → 1.25 → 1.26). However, **node groups can jump versions** — e.g., node group on v1.24 can jump directly to v1.26 (as long as it's within the ±2 version skew that the control plane supports). This saves significant time.

**The actual upgrade approach:**
1. Upgraded the control plane step by step (one version at a time per cluster)
2. Once control plane was at the target version, **added a new node group** at the target version alongside the existing old node group
3. **Tainted all old nodes** — prevented Kubernetes from scheduling new pods there
4. With autoscaling and multiple replicas per app, pods began **gradually moving to the new node group** — at any moment, traffic was split between old and new
5. Once all pods migrated to the new node group — removed old nodes completely
6. **Zero downtime** — traffic was always served from at least one of the two node groups
7. Entire process was **automated after the first cluster** — remaining 7 clusters were upgraded with few manual clicks + pipeline automation

**Rollback strategy:**
- Full backup of the cluster before starting
- All Helm charts and deployment files manually verified to be in sync with cluster state
- Snapshots taken of all resources
- If things go wrong: either spin up from scratch or restore from backup
- *(Acknowledged: downtime is unavoidable if a cluster upgrade fully fails — the goal is to minimize it)*

**Time taken:** First cluster ~4–5 hours (manual); subsequent clusters ~1–2 hours each (automated), run in parallel.

---

## 19. 📀 Stateful vs. Stateless Apps on Kubernetes

**Q: How do stateful and stateless applications differ in Kubernetes? Real-world example?**

### Hussein's Answer

- Deployed **MongoDB on EKS** using a **StatefulSet** with Persistent Volumes — for a client POC requiring everything on Kubernetes
- Interviewer's note: AWS has **DocumentDB** (MongoDB-compatible managed service) — preferring managed services over stateful K8s workloads is usually better
- Hershel's shared view: **EKS/Kubernetes should be mostly stateless** — stateful workloads belong in managed services (RDS, DocumentDB, ElastiCache, MSK)
- Exception: cost — managed services are more expensive than self-managed on K8s

**Blue-Green Deployments in Kubernetes:**
- Deploy the new version as a separate deployment in the same namespace
- Update **Ingress annotations** to shift 100% traffic from old to new version via labels/selectors
- Alternatively, use **Argo Rollouts** — a CRD that natively manages blue/green and canary deployments with full configuration control

---

## 20. 🏗️ Enterprise Terraform Directory Structure

**Q: How would you structure Terraform for an enterprise with 15–20 accounts, multiple environments, and multiple regions?**

---

### Hussein's Structure

```
terraform/
├── dev/
│   ├── account-111111111/
│   └── account-222222222/
├── staging/
│   └── account-333333333/
├── prod/
│   └── account-444444444/
├── shared-services/       # Centralized logging, S3, monitoring account
└── modules/               # Reusable modules per service
    ├── network/
    ├── eks/
    ├── iam/
    └── rds/
```

- Modules called from each account directory — no code duplication
- **Remote backend in S3** + **DynamoDB locking** for state management

---

### Hershel's Structure *(Landing Zone / Control Tower scale)*

```
terraform/
├── org-management/        # AWS Organizations, Control Tower config
├── modules/               # Reusable modules (EKS, network, IAM, etc.)
│   ├── kubernetes/
│   ├── network/
│   └── rds/
└── environments/
    ├── dev/
    │   └── main.tf        # Calls modules, passes dev tfvars
    ├── staging/
    └── prod/
    
tfvars/
├── dev.tfvars             # Dev-specific values (T2 micro, smaller instances)
├── staging.tfvars
└── prod.tfvars            # Prod values (large instances, multi-AZ)
```

**Hershel's promotion model:**
- Same Terraform code is promoted across environments — only the `.tfvars` file changes
- CI/CD pipeline automatically switches the active `.tfvars` file based on the target environment variable
- Dev uses `t2.micro`, prod uses large instances — same module, different values
- All account numbers, URLs, and access details passed through tfvars files

---

## 21. 🔒 Terraform State — Collaboration & Secret Management

**Q: How do you prevent 20 DevOps engineers from overwriting each other's Terraform state? How do you handle secrets in Terraform?**

---

### Hussein's Answer

- **S3 remote state + DynamoDB locking** — state is locked when one engineer is applying
- For parallel development: **Terraform workspaces locally** — each engineer works in their own workspace without touching shared state
- On PR merge → main pipeline runs against shared S3 state and deploys

**Secrets:** Use Terraform's `random` provider to generate passwords → store directly in **AWS Secrets Manager** via Terraform code. *Risk: the generated secret still appears in the Terraform state file — for regulated industries, secrets are created manually and passed as ARNs.*

---

### Hershel's Answer *(More Advanced)*

- **Remote state in S3** is the baseline — but state locking means 20 engineers wait in a queue
- **Terraform Workspaces (open source)** — each engineer creates their own workspace; Terraform creates a **separate state file in S3** for each workspace (not stored locally — always remote)
- When the engineer is done, the workspace state is cleaned up — either via pipeline automation (workspace created on MR, deleted on merge) or manually
- **Benefit:** 20 engineers can work simultaneously without any conflicts

**Hershel's important clarification:** When asked "is this the cloud/enterprise version?", Hershel confirmed this is **open-source Terraform workspace** — Terraform pushes the workspace state file to the same S3 remote backend. Each workspace gets its own prefix in S3, keeping state isolated.

**Drift management:**
- No one should have manual write access to AWS — **read-only access only**; any provisioning goes through Terraform pipelines
- Temporary manual access (1 hour) must have documented business justification and is fully logged
- If drift occurs: use **terraform state import** to sync state with actual resource, or delete and let Terraform recreate it
- All Terraform code is version-controlled in GitLab — can always revert to a known working commit

**Emergency manual change vs. IaC:**
- Prefer IaC always; but if a CEO-level P0 situation demands a manual change right now — make the change manually, **but** document it, notify the team, and schedule a Terraform import or pipeline fix for the next release to bring IaC back in sync with reality

**One-time infrastructure (won't be updated for 5 years) — IaC or console?**
- **Always IaC** — after 5 years nobody will remember what was created, why, or whether it's still needed. Terraform code + documentation is the only reliable record

---

## 22. 🚀 AI in DevOps & Career Advice

**Q: How is AI impacting your day-to-day work? What's your 5-year plan?**

---

### Hussein's Answer

- Uses AI primarily for **documentation** — reliable and fast; AI generates well-structured docs
- Avoids AI for actual infrastructure work — prefers **official docs** (Terraform, Kubernetes) to avoid hallucinations
- Doesn't currently use tools like Cursor — *Ajit recommended Cursor specifically*

**Career goal:** 3 years → hardcode DevOps engineer. 5 years → Solution Architect who still stays hands-on with technology.

**Interviewer's advice (Ajit):**
> "I worked as an architect for 6 years and was hands-on the entire time. An architect who only does solutioning and diagrams without hands-on experience is incomplete. Stay technical regardless of your title."

---

### Hershel's Answer *(More Detailed on AI)*

- AI is **not a replacement** — it's a productivity multiplier for those who already have skills
- GenAI outputs are only as good as your prompt — and prompts have limits; outputs are never 100% reliable; unskilled users will create more problems than they solve
- **If you already have skills:** tasks that took 2–3 days now take 1–2 hours
- **Future of DevOps with AI:**
  - AI agents integrated into CI/CD pipelines — agent reads logs, identifies root cause, suggests exact fix; what took 2 hours now takes 10 minutes
  - AI agents managing Kubernetes — automatically checking pod health, restarting failed pods, managing autoscaling
  - The trend: Cloud → DevOps → DevSecOps → GitOps → **MLOps/AIOps** is next
  - Teams will get smaller but individual engineers will be far more valuable and highly paid

**Hershel's personal AI policy:**
> "I'm comfortable giving LLMs read access to my AWS infrastructure for cost optimization suggestions. But write access — terminating or modifying resources — I'm not convinced yet."

**Ajit's perspective:**
> "A year ago you weren't comfortable giving read access. Now you are. In 8 months you might be comfortable with write access too. These are interesting times."

---

## 📌 Combined Scorecard — What Interviewers Look For

| Topic | Hussein | Hershel |
|---|---|---|
| Architecture Design | ✅ Solid | ✅ Excellent — asked clarifying questions first |
| EKS vs ECS | ✅ Good | ✅ Very good — added hybrid/on-prem EKS point |
| Disaster Recovery | ✅ All 3 tiers | ✅ Smart split by service criticality |
| DynamoDB Global Tables | ⚠️ Mentioned but missed for active-active DR | ✅ Proactively mentioned |
| Security | ✅ Good | ✅ Strong — no manual access policy |
| Load Balancers | ✅ Complete | ✅ Added ingress controller context |
| CI/CD & GitOps | ✅ Argo CD + Terraform | ✅ End-to-end pipeline design with OIDC auth |
| Kubernetes Upgrade | ✅ Standard process | ✅ Outstanding — real 8-cluster story with version skew insight |
| Terraform Structure | ✅ Good | ✅ More detailed — tfvars promotion model |
| Terraform Workspaces | ⚠️ Local only | ✅ Advanced — remote state per workspace in S3 |
| Terraform Drift | ✅ Covered | ✅ In-depth — state import, policy enforcement |
| Observability | ✅ CloudWatch + Grafana | ✅ Added sidecar logging agent pattern |
| AI in DevOps | ⚠️ Limited adoption | ✅ Thoughtful — future MLOps/AIOps vision |
| Cost Optimization | ✅ Good | ✅ Added WAF reviews, healthcare compliance context |
| Trusted Advisor cost | ⚠️ Thought it was free | ✅ Knew it's paid |
| Service Discovery/Mesh | Not covered | ✅ Mentioned Istio/service mesh |

---

*This document is based on the actual transcripts of two mock interviews. Answers are paraphrased from each candidate's real responses with additional context from Ajit's feedback and corrections.*
