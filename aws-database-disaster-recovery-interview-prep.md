# AWS Database Disaster Recovery (DR) — Interview Prep Guide

A structured, interview-ready reference for **choosing the right AWS database DR
strategy**. Built from AWS re:Post Live guidance and validated against AWS
documentation. Use it to answer "when do we use which AWS database for DR?"

---

## Table of Contents

**Part A — Fundamentals (know these cold)**
- [1. Foundational Concepts (RTO, RPO, Cost, Complexity, Retention)](#1-foundational-concepts-always-start-here)
- [2. Synchronous vs. Asynchronous Replication (The Core Trade-off)](#2-synchronous-vs-asynchronous-replication-the-core-trade-off)
- [3. The DR Strategy Spectrum (Cost vs. RTO/RPO)](#3-the-dr-strategy-spectrum-cost-vs-rtorpo)

**Part B — Relational DR (RDS / Aurora)**
- [4. Decision Guide — Which AWS Database DR Option to Use](#4-decision-guide--which-aws-database-dr-option-to-use)
  - [4.1 RDS Multi-AZ (in-region HA, auto failover)](#41-rds-multi-az--in-region-high-availability)
  - [4.2 RDS Multi-AZ DB Cluster](#42-rds-multi-az-db-cluster)
  - [4.3 Cross-Region Read Replica](#43-cross-region-read-replica)
  - [4.4 Aurora Global Database (+ headless secondary)](#44-aurora-global-database--preferred-cross-region-dr-for-aurora)
  - [4.5 Cross-Region Automated Backups (cheapest cross-region DR)](#45-cross-region-automated-backups--cheapest-cross-region-dr)
  - [4.6 Manual Snapshots (compliance / long retention)](#46-manual-snapshots--compliance--long-retention)
  - [4.7 AWS DMS for DR (niche)](#47-aws-dms-database-migration-service-for-dr--niche)
- [5. Engine-Specific: RDS for Oracle DR](#5-engine-specific-rds-for-oracle-dr)

**Part C — NoSQL DR (DynamoDB & family)**
- [4B. DynamoDB & Other NoSQL DR](#4b-dynamodb--other-nosql-dr-not-in-the-original-session)
  - [4B.1 DynamoDB Global Tables (active/active)](#4b1-dynamodb-global-tables--multi-region-activeactive)
  - [4B.2 DynamoDB PITR](#4b2-dynamodb-point-in-time-recovery-pitr)
  - [4B.3 DynamoDB On-Demand Backups + AWS Backup](#4b3-dynamodb-on-demand-backups--aws-backup)

**Part D — Backup & Whole-Application DR**
- [5B. Best Backup Strategy — Regional & Cross-Regional Services](#5b-best-backup-strategy--regional--cross-regional-aws-services)
- [6. DR Is a Whole-Application Problem](#6-dont-forget-dr-is-a-whole-application-problem)

**Part E — Interview Quick Reference**
- [7. Rapid-Fire Interview Q&A](#7-rapid-fire-interview-qa)
- [8. One-Line Summary Cheat Sheet](#8-one-line-summary-cheat-sheet)
- [9. Verification & Sources](#9-verification--sources)

---

## 1. Foundational Concepts (Always Start Here)

Before recommending any solution, an architect frames DR around these criteria:

| Term | Meaning | Interview soundbite |
|------|---------|---------------------|
| **RTO** (Recovery Time Objective) | How long to restore service after a disaster (downtime tolerated). | "How fast must we be back online?" |
| **RPO** (Recovery Point Objective) | How much data loss is tolerable, measured in **time**. | "How much data can we afford to lose?" |
| **Cost** | Dollar cost of the strategy. | Lower RTO/RPO ⇒ higher cost. |
| **Complexity** | Operational/engineering effort. | Lower RTO/RPO ⇒ higher complexity. |
| **Backup Retention** | How long backups are kept (driven by RPO + **compliance**). | The optional 5th criterion. |

**Golden rule:** The *lower* the RTO and RPO you require, the *more expensive and
complex* the strategy — and you often have to **layer multiple strategies** to
hit strict targets.

### RPO Example
- Disaster at **5:00 PM**, last good backup at **4:30 PM** ⇒ **RPO = 30 minutes**.
- Zero RPO = **zero data loss** = the exact acknowledged transaction must exist on
  the recovery site at the same millisecond.

---

## 2. Synchronous vs. Asynchronous Replication (The Core Trade-off)

| Aspect | **Synchronous** | **Asynchronous** |
|--------|-----------------|------------------|
| RPO | **Zero** (true zero data loss) | Near-zero (seconds, = replication lag) |
| How it works | Commit is **not acknowledged** to the client until data is replicated to the other site (two-phase commit style). | Client is acknowledged **immediately**; replication happens after (log shipping/streaming). |
| Latency | **High** — every write waits for cross-site round trip. | **Low** — best performance. |
| Cost | Highest | Moderate |
| Cross-region reality | Impractical across regions due to network latency. | The practical choice for cross-region DR. |

**Key insight:** True zero RPO across regions requires synchronous replication,
which introduces **very high latency** — usually unacceptable for OLTP workloads
that need millisecond responses. Hence most cross-region DR uses **asynchronous**
("near-zero RPO").

### Replication Lag = Your RPO
- Any async replication has lag **by design**. That lag **is** your RPO.
- If primary fails at minute 4 of a 5-minute lag window, that data is lost.
- Causes of high lag: bursts of writes, network issues, **under-provisioned
  replica** (too-small instance can't keep up), long-running queries on source.
- **Monitor it:** Put CloudWatch alarms on replication-lag metrics (for cross-region
  read replicas and Aurora Global Database) so you're alerted when lag exceeds your
  acceptable RPO.

---

## 3. The DR Strategy Spectrum (Cost vs. RTO/RPO)

From cheapest/slowest to most expensive/fastest:

```
Backup & Restore  →  Pilot Light  →  Warm Standby  →  Multi-Site Active/Active
   (high RTO/RPO)                                         (near-zero RTO/RPO)
   cheapest                                               most expensive
```

---

## 4. Decision Guide — Which AWS Database DR Option to Use

### Quick-reference matrix

| Solution | Scope | RPO | RTO | Failover | Best for |
|----------|-------|-----|-----|----------|----------|
| **RDS Multi-AZ** | Single region (across AZs) | **0** (synchronous) | **< 60–120 s** (auto) | **Automatic** | In-region HA; survive AZ failure |
| **RDS Multi-AZ DB Cluster** (MySQL/PostgreSQL only) | Single region, 3 AZs | ~0 | Auto | **Automatic** | Higher availability + read scaling in region |
| **Cross-Region Read Replica** | Cross region | Seconds (async) | Minutes (manual promote) | **Manual** | Cross-region DR + optional read offload |
| **Aurora Global Database** | Cross region | ~1 s (async, storage-level) | < 1 min typical | Managed / manual | Low-RPO cross-region DR for Aurora |
| **Cross-Region Automated Backups** | Cross region | Up to **5 min** (PITR) | Higher (restore + log replay) | Manual restore | Cheapest cross-region DR; low-write/reporting DBs |
| **Manual Snapshots** | Any region / archive | Point-in-snapshot only (no PITR) | Restore time | Manual | **Compliance / long retention** (> 35 days) |
| **AWS DMS** | Cross region/engine | Seconds | Manual | Manual | Read/write DR target, version/engine mismatch, niche cases |
| **DynamoDB Global Tables** (NoSQL) | Cross region | **~1 s** | **~0** (active/active) | **None needed** | Multi-region active/active NoSQL DR, low-latency global |

---

### 4.1 RDS Multi-AZ — In-Region High Availability
- **Synchronous** replication to a standby in a **different AZ**, same region.
- **RPO = 0**, **RTO < 60–120 s**, failover is **fully automatic**.
- Single endpoint; the DNS/IP swap is **managed** by AWS transparently.
- **Limitation:** Protects against **AZ** failure, **not** a full **region**
  outage. If the whole region goes down, Multi-AZ alone won't save you.
- **When to use:** Baseline HA for production. Applies to **all engines**.

**Automatic failover triggers:**
1. Hardware / underlying network failure of the primary (detected by RDS health checks).
2. Resource exhaustion (e.g., a runaway query starves memory/CPU so health checks fail).

**Testing failover (for compliance demos):**
- Console: **Actions → Reboot → Reboot with failover** (option only appears for Multi-AZ).
- Verify: run `nslookup <endpoint>` before/after — the resolved **IP changes**, the endpoint stays the same.
- Automate/schedule chaos testing with **AWS FIS (Fault Injection Service)**.

### 4.2 RDS Multi-AZ DB Cluster
- Available for **MySQL and PostgreSQL only** (open-source engines).
- Automatic failover in region; adds readable standbys.

### 4.3 Cross-Region Read Replica
- **Asynchronous** replication to another region.
- Doubles as a **readable** database (offload reads) — but you **cannot write** to it.
- **Failover is MANUAL** — you must **promote** the replica.
- **One-way promotion:** Once promoted it becomes a standalone primary and
  **cannot revert** to a replica. To restore the original topology you must
  snapshot the primary, copy to DR, and rebuild the replica.
- **Automating failover:** Route 53 health check on the primary endpoint +
  **Lambda** that promotes the replica when the health check fails.

### 4.4 Aurora Global Database — Preferred Cross-Region DR for Aurora
- One **primary** (writable) Region + **up to 10 read-only secondary** Regions.
- Replication happens at the **storage layer** (Aurora decouples compute from storage),
  over the **AWS backbone network** ⇒ typically **~1 second** lag.
- **Two recovery operations** (verified against AWS docs):
  - **Switchover** (planned, formerly "managed planned failover"): relocate the
    primary to a healthy secondary Region with **no data loss** — for Region
    rotation/drills.
  - **Failover** (unplanned): recover to a secondary Region **after a primary-Region
    outage** — lower RTO/RPO than traditional replication.
- **Write forwarding:** secondary clusters can forward writes to the primary, so the
  app can connect locally in any Region while writes still go to the single primary.
- **When to use:** Aurora workloads needing low RPO cross-region DR with fast recovery.

#### What is an Aurora Global Database "headless" secondary?
- **Definition:** A secondary region added to an Aurora Global Database that has a
  **storage/cluster volume but ZERO DB instances (no compute)** provisioned. The
  word *"headless"* = **no compute head**, just the data.
- **Why it works:** Aurora **decouples compute from storage**. Replication happens
  at the **storage layer**, so data keeps replicating into the secondary region's
  cluster volume **even though no instance is running** there to serve it.
- **Why use it (the benefit):** You keep the DR region **continuously up to date**
  (~1 s RPO) while **paying only for storage/replication — not for idle compute**.
  This is the cheapest way to keep an Aurora region "warm-ish" for DR.
- **At failover / DR event:** You **add a DB instance** (a compute head) to the
  headless secondary and then promote it. There is a **short delay** to launch that
  instance, so the trade-off is **lower cost vs. slightly higher RTO** compared to a
  secondary that already has running instances.
- **In one line:** *Headless = a DR region with the data replicated but no running
  database instance, so you save compute cost until you actually need to fail over.*

### 4.5 Cross-Region Automated Backups — Cheapest Cross-Region DR
- RDS copies **both snapshots and transaction logs** to another region ⇒ enables
  **Point-in-Time Restore (PITR)** in the DR region.
- **Theoretical RPO ≈ 5 minutes.**
- **RTO is higher** and **variable** — restore = latest copied snapshot + **replay
  of transaction logs** since that snapshot. The **older** the last snapshot, the
  **longer** the replay (e.g., snapshot at 1 AM, region fails at 10 PM ⇒ ~21–22 h
  of logs to replay).
- **Fully isolated:** backups/logs live in an **AWS-managed (service) account** in
  the DR region — you restore even if the **primary region is entirely down**; not
  pulled from the primary.
- **Cheapest** cross-region option: **no standby compute**, only **storage** cost.
- **When to use:** Low-write databases, **reporting/BI** databases refreshed daily/weekly,
  or any workload where a larger RTO is acceptable and cost matters most.

**How to configure (demo steps):**
1. RDS Console → select instance → **Modify**.
2. **Additional configuration** → Backup section.
3. Enable **Backup replication** → choose destination region → set retention → **Continue** → **Modify**.
4. Backups then appear in the destination region's snapshots for full restore.

**Backup basics recap:**
- Automated backups: one snapshot/day in the **backup window**, retention **1–35 days**.
- PITR available up to the **last 5 minutes** within the retention window.

### 4.6 Manual Snapshots — Compliance & Long Retention
- Use when retention must exceed the **35-day** automated maximum (e.g., "keep monthly backup for 70 years").
- **No PITR** — a manual snapshot restores to the **exact moment it was taken** only.
- **Cost tip:** For very long compliance retention, **export data logically** and
  store in **S3 Glacier / Glacier Deep Archive** — far cheaper than snapshots, at
  the cost of higher restore effort (acceptable since old-compliance restores are rare).

### 4.7 AWS DMS (Database Migration Service) for DR — Niche
- Keeps a **read/write** target in sync (unlike read replicas which are read-only).
- Useful when the DR site also needs writable schemas (e.g., a monitoring/BI app
  writing its own schema alongside replicated data), or when there are **engine/version
  mismatches**.
- **Not recommended** as your primary full-DR foundation across different engine
  versions — that creates application compatibility problems later.

> **Note:** The re:Post Live session this guide is based on focused on **relational**
> databases (RDS / Aurora), so it did not cover DynamoDB. For completeness, the
> **NoSQL / DynamoDB** DR story is below — it is one of the *strongest* DR options
> AWS offers.

---

## 4B. DynamoDB & Other NoSQL DR (Not in the original session)

### 4B.1 DynamoDB Global Tables — Multi-Region, Active/Active
- **Fully managed multi-region, multi-active** replication — every region is
  **read AND write** (unlike RDS read replicas which are read-only).
- **Asynchronous** replication over the AWS backbone, typically **~1 second** lag.
- **RPO ≈ 1 second**, **RTO ≈ 0** — no promotion step; the DR region is already
  live and writable. Just point traffic there (Route 53 / app config).
- **Conflict resolution:** **last-writer-wins** on concurrent writes to the same item.
- **Two consistency modes** (verified against AWS docs):
  - **MREC — Multi-Region Eventual Consistency** (default): async, ~1 s lag, RPO ~1 s.
  - **MRSC — Multi-Region Strong Consistency**: strongly consistent reads across
    Regions with **RPO = 0** (same-account only). Choose the mode **at creation**;
    it **cannot be changed** later.
- **Account models:** **same-account** (all replicas in one account) or
  **multi-account** global tables (replicas across accounts for team/security
  isolation). MRSC supports **same-account only**.
- **When to use:** Low-latency global apps, and any workload needing near-zero
  RTO/RPO cross-region DR **without** managing failover/promotion.

### 4B.2 DynamoDB Point-in-Time Recovery (PITR)
- Continuous backups with restore to **any second in the last 35 days**.
- Protects against accidental writes/deletes and logical corruption.
- Restores create a **new table**.

### 4B.3 DynamoDB On-Demand Backups + AWS Backup
- Full on-demand snapshots for **long-term / compliance** retention (beyond 35 days).
- Manage centrally and **copy cross-region / cross-account** via **AWS Backup**.

### DynamoDB DR quick matrix

| Feature | RPO | RTO | Failover | Use case |
|---------|-----|-----|----------|----------|
| **Global Tables (MREC)** | ~1 s | ~0 (already live) | **None needed** (active/active) | Cross-region active/active DR, low-latency global |
| **Global Tables (MRSC)** | **0** (strong consistency) | ~0 (already live) | **None needed** (active/active) | Cross-region DR needing strongly consistent reads (same-account) |
| **PITR** | 1 s (last 35 days) | Restore time (new table) | Manual restore | Accidental delete / corruption |
| **On-demand backup + AWS Backup** | Point-in-backup | Restore time | Manual restore | Long-term / compliance, cross-region/account copy |

### Other AWS databases (same DR pattern family)
| Service | Cross-region DR mechanism |
|---------|---------------------------|
| **Amazon Aurora** | Global Database (see §4.4) |
| **DynamoDB** | Global Tables (active/active) |
| **DocumentDB** | Global Clusters (cross-region, storage-level) |
| **Neptune** | Global Database |
| **ElastiCache (Redis)** | Global Datastore |
| **Amazon Keyspaces / MemoryDB** | Multi-Region replication |

**Interview soundbite:** *"For relational, cross-region DR means Aurora Global
Database or read replicas with manual promotion. For DynamoDB, Global Tables give
you active/active multi-region with ~1s RPO and effectively zero RTO — no
promotion required."*

---

## 5. Engine-Specific: RDS for Oracle DR

Question pattern: *"Most cost-effective Oracle DR with RTO/RPO < 30 minutes?"*

**Engine-agnostic options (apply to all engines):** Multi-AZ, automated/continuous
backup replication, read replicas, cross-region read replicas.

**Oracle-specific caveats — Read Replicas:**

| Replica type | License needed | Serves reads? | Purpose |
|--------------|----------------|---------------|---------|
| **Regular read replica** | Enterprise Edition **+ Active Data Guard** | **Yes** | DR **and** read offload |
| **Mounted read replica** | Enterprise Edition only (**no** Active Data Guard) | **No** (mounted, no connections) | **DR only** (cheaper licensing); promote when needed |

**Recommendation for < 30 min RTO/RPO:** use a **read replica** (regular or
mounted), **not** continuous backup restore — a restore can exceed 30 minutes and
risk the SLA.

**Read replica performance notes:**
- Replicas are **asynchronous**; read workload doesn't directly cause lag —
  **unless** you exhaust CPU/memory/IOPS/throughput/network.
- **Long-running queries on the source add to lag** (target waits for the commit).
- Prefer app-side handling; use engine **timeout settings** as a fail-safe rather
  than manually killing queries.

---

## 5B. Best Backup Strategy — Regional & Cross-Regional AWS Services

Goal: **durable, recoverable, and geographically isolated** backups. A good backup
strategy layers **in-region** (fast, cheap, everyday recovery) with **cross-region**
(disaster isolation) — because a backup stored only in the failed region is useless.

### The 3-2-1 rule (cloud version)
- **3** copies of data, **2** storage types/services, **1** copy in a **different
  region** (and ideally a **different account** for ransomware/blast-radius isolation).

### Recommended layered strategy

| Layer | Purpose | Services to use |
|-------|---------|-----------------|
| **1. In-region automated backups** | Fast day-to-day PITR recovery | RDS/Aurora automated backups (1–35 days), DynamoDB PITR (35 days) |
| **2. Cross-region copy** | Survive a full region outage | RDS **cross-region automated backup replication**, AWS Backup **cross-region copy**, S3 **CRR** |
| **3. Cross-account copy** | Ransomware / accidental-deletion isolation | AWS Backup **cross-account copy** + **Vault Lock** |
| **4. Long-term / compliance archive** | Cheap multi-year retention | Logical export to **S3 → Glacier / Glacier Deep Archive** |

### Key AWS backup services and when to use them

| Service | Scope | Best for |
|---------|-------|----------|
| **AWS Backup** ⭐ | Central, multi-service, **regional + cross-region + cross-account** | The **recommended default** — one policy-driven console for RDS, Aurora, DynamoDB, EFS, EBS, EC2, S3, FSx, DocumentDB, Neptune, VMware, etc. |
| **RDS/Aurora automated backups** | In-region, + optional **cross-region replication** | Native PITR (~5 min RPO); enable backup replication for DR region |
| **RDS/Aurora manual snapshots** | Any region (copyable) | Retention beyond 35 days, compliance |
| **DynamoDB PITR + on-demand backups** | In-region; export/copy cross-region via AWS Backup | NoSQL recovery + long-term backups |
| **S3 Cross-Region Replication (CRR)** | Cross-region object copy | Static assets, exported/logical backups, data lakes |
| **S3 Same-Region Replication (SRR)** | In-region, cross-account/bucket | Compliance, log aggregation, account isolation |
| **EBS Snapshots + Data Lifecycle Manager (DLM)** | Regional, **copyable cross-region** | Self-managed DBs on EC2, volume-level backup automation |
| **AWS Elastic Disaster Recovery (DRS)** | Cross-region/on-prem block-level replication | Self-managed DBs/servers on EC2 needing low RTO/RPO |
| **S3 Glacier / Deep Archive** | Regional (copy for cross-region) | Cheapest long-term compliance archive |

### AWS Backup — why it's the recommended backbone
- **Centralized policy (Backup Plans):** schedule, retention, lifecycle-to-cold-storage all in one place.
- **Cross-region copy** and **cross-account copy** built in — core to DR.
- **Backup Vault Lock (WORM):** immutable backups — protects against deletion/ransomware and meets compliance.
- **Tag-based selection:** automatically back up any resource with a given tag.
- **Audit Manager / Backup Audit:** prove compliance of your backup posture.

### Best-practice checklist
- ✅ Enable **RDS cross-region automated backup replication** for ~5-min cross-region RPO.
- ✅ Use **AWS Backup** with a plan that does **cross-region + cross-account** copies.
- ✅ Turn on **DynamoDB PITR** for all production tables.
- ✅ Apply **Vault Lock (WORM)** on the compliance/DR vault.
- ✅ Move long-term retention to **Glacier Deep Archive** to cut cost.
- ✅ Encrypt with **KMS** (replicate the CMK / use multi-region keys for cross-region restores).
- ✅ **Test restores regularly** — an untested backup is not a backup. Validate restore time against RTO.
- ✅ Tag resources so backup selection is automatic and nothing is missed.

**Interview soundbite:** *"Layer in-region PITR for everyday recovery, AWS Backup
with cross-region and cross-account copies for disaster isolation, Vault Lock for
immutability, and Glacier Deep Archive for cheap long-term compliance — then test
restores regularly against the RTO."*

---

## 6. Don't Forget: DR Is a Whole-Application Problem

Databases are foundational, but full DR must cover **the entire stack**:

| Component | DR mechanism |
|-----------|--------------|
| **Application artifacts (EC2)** | Copy **AMIs** to the DR region |
| **Containers** | Replicate container **images** to DR region |
| **Amazon WorkSpaces apps** | Copy the app **image** to DR region |
| **Static files** | **S3 Cross-Region Replication (CRR)** |
| **Orchestrated backups** | **AWS Backup** (single console for many services) |
| **DNS / traffic routing** | **Route 53 failover** records + health checks (+ Lambda) |
| **Failover readiness & routing control** | **Route 53 Application Recovery Controller (ARC)** — continuous readiness checks + highly reliable routing controls to shift traffic during DR |
| **Load balancers** | No backup needed, but must be **provisioned** in DR region |
| **Whole infrastructure** | **Infrastructure as Code** — CloudFormation (or Terraform) template ready to provision the DR region |

**Critical points:**
- Even if the database fails over, a missing dependency (S3 files, a third-party
  service, load balancer, DNS) can still leave you stuck.
- **Measure provisioning time** against your **RTO** — IaC + Route 53 checks let you
  validate the whole architecture comes up within target.
- **Inject chaos** (kill resources) in a controlled way to prove the DR design
  actually works and meets acceptable interruption limits.

---

## 7. Rapid-Fire Interview Q&A

**Q: What are RTO and RPO?**
RTO = time to restore service after a disaster. RPO = tolerable data loss measured
in time (gap since last recoverable point).

**Q: How do you achieve true zero RPO?**
Synchronous replication (no ack until replicated). In-region: **RDS Multi-AZ**.
Cross-region is impractical due to latency; use **async / near-zero RPO** instead.

**Q: Multi-AZ RPO and RTO?**
RPO = **0** (synchronous), RTO **< ~2 minutes**, failover **automatic**, DNS/IP swap managed.

**Q: Multi-AZ vs. cross-region read replica?**
Multi-AZ = in-region HA, synchronous, automatic failover, survives AZ (not region)
loss. Cross-region read replica = async, manual promotion, survives region loss,
one-way promotion.

**Q: Cheapest cross-region DR for RDS?**
**Cross-region automated backups** (snapshots + logs) — no standby compute, ~5-min
RPO, higher/variable RTO due to log replay.

**Q: What limits RPO in async replication?**
**Replication lag.** Monitor it with CloudWatch alarms.

**Q: Best DR for Aurora across regions?**
**Aurora Global Database** (storage-level replication, ~1 s lag), optionally with a
**headless** secondary to save compute.

**Q: What about DynamoDB / NoSQL DR?**
**Global Tables** = multi-region **active/active** (~1 s RPO, ~0 RTO, no promotion).
Add **PITR** (35-day) for corruption and **on-demand backups + AWS Backup** for
long-term/compliance and cross-region copies.

**Q: When manual snapshots over automated backups?**
When retention must exceed **35 days** (compliance). No PITR; consider Glacier for
long-term cost savings.

**Q: How do you test RDS failover?**
Console **Reboot with failover** (Multi-AZ), verify via `nslookup` IP change; automate
with **AWS FIS**.

---

## 8. One-Line Summary Cheat Sheet

- **In-region HA, zero RPO, auto failover** → **RDS Multi-AZ**
- **Cross-region DR + readable copy, manual failover** → **Cross-Region Read Replica**
- **Cross-region DR for Aurora, ~1 s RPO** → **Aurora Global Database** (headless to save cost)
- **NoSQL cross-region, active/active, ~1 s RPO / ~0 RTO** → **DynamoDB Global Tables**
- **Cheapest cross-region DR, ~5 min RPO** → **Cross-Region Automated Backups**
- **Compliance / long retention (>35 days)** → **Manual Snapshots** (+ Glacier)
- **Writable DR target / engine mismatch** → **AWS DMS** (niche)
- **Oracle DR < 30 min** → **Read Replica** (mounted = DR-only, cheaper licensing)
- **Whole app** → AMIs/images, S3 CRR, AWS Backup, Route 53 failover, IaC, chaos testing
```

---

## 9. Verification & Sources

Key facts in this guide were cross-checked against official AWS documentation:

| Claim | Status | AWS source |
|-------|--------|------------|
| Multi-AZ = synchronous standby; DB **instance** standby doesn't serve reads, DB **cluster** (2 standbys) can serve reads | ✅ Verified | RDS Multi-AZ deployments |
| Automated backups retention up to **35 days**; **100 manual snapshots** per Region; snapshots copyable cross-Region | ✅ Verified | RDS Introduction to backups |
| Aurora Global Database: 1 primary + **up to 10 secondary** Regions, storage-layer replication, **~1 s** latency; **switchover** (no data loss) vs **failover** (unplanned); **write forwarding** | ✅ Verified | Aurora Global Database |
| DynamoDB Global Tables: multi-active, async, last-writer-wins, low/zero RPO; **MREC** (eventual) & **MRSC** (strong, RPO 0, same-account) modes; same/multi-account | ✅ Verified (updated) | DynamoDB Global tables |
| RDS for Oracle **mounted** read replica mode exists (DR-only) | ✅ Verified | RDS Oracle read replicas |

> **Note:** Exact failover timings (e.g., Multi-AZ "60–120 s") and "~5 min" cross-region
> backup RPO are typical/observed values and can vary by workload and engine. Treat
> them as guidance, not SLAs. Always confirm current limits in the AWS docs for your
> specific engine and Region before committing to RTO/RPO targets.

