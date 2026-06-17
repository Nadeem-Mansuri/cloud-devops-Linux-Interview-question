# AWS Database Disaster Recovery (DR) Strategy — Interview Prep Summary

> Based on AWS re:Post Live session with Principal SA, Senior TAM, and Senior Cloud Support DB Engineer from the RDS/Aurora team.

---

## 1. Core DR Concepts: RPO & RTO

### Recovery Point Objective (RPO)
- Measures **how much data loss** your business can tolerate, expressed in time.
- Example: Disaster at 5:00 PM, last good backup at 4:30 PM → **RPO = 30 minutes**.
- Lower RPO = less data loss tolerance = **more expensive and complex** solutions.

### Recovery Time Objective (RTO)
- Measures **how long it takes to restore service** after a disaster.
- Represents the maximum acceptable downtime before the system is back online.

### Design Matrix
When designing a DR strategy, evaluate across **4 dimensions**:
1. **RPO** — acceptable data loss
2. **RTO** — acceptable downtime
3. **Cost** — infrastructure and licensing
4. **Complexity** — operational overhead
5. *(Optional)* **Backup Retention** — compliance-driven retention requirements

> 💡 **Key insight**: Lower RTO/RPO = higher cost and complexity. Always align targets with business needs before choosing a solution.

---

## 2. Zero RPO — What It Really Means

- Zero RPO = **zero data loss** — every committed transaction must be present on the recovery site.
- Only achievable with **synchronous replication**: the database does NOT acknowledge a transaction until it is replicated to the secondary site.

### Challenge: Latency
- Synchronous replication across regions (e.g., US East → US West) introduces **high latency** because the client must wait for a round-trip acknowledgment across regions.
- This conflicts with OLTP requirements (millisecond response times).

### When Zero RPO Is Achievable (In-Region)
- **RDS Multi-AZ**: Synchronous standby in a different Availability Zone.
  - Automatic failover in **60–120 seconds** (RTO < 2 minutes)
  - **RPO = 0** (synchronous replication)
  - DNS/Route 53 endpoint updated automatically — transparent to application
  - ⚠️ Protects against AZ failure, **not entire region failure**

---

## 3. Near-Zero RPO — Asynchronous Replication (Cross-Region)

For cross-region DR, synchronous replication is impractical. Use **asynchronous replication** for near-zero RPO:

| Solution | Engine | Notes |
|---|---|---|
| **Cross-Region Read Replica** | MySQL, PostgreSQL, Oracle, etc. | Online readable replica; failover is **manual** |
| **Aurora Global Database** | Aurora (MySQL/PostgreSQL) | Replication via storage layer on AWS backbone; very low lag |
| **AWS Database Migration Service (DMS)** | Multi-engine | Useful for version mismatches or read/write DR scenarios |

### Replication Lag = Your Actual RPO
- Lag represents the data not yet replicated at the time of failure.
- **Monitor** lag with CloudWatch alarms set to your acceptable RPO threshold.
- Causes of high lag: burst writes, network issues, under-provisioned replica, long-running queries.

### Manual Failover for Cross-Region Read Replica
- **Not automatic** — must promote manually.
- Can automate using: **Route 53 health checks + Lambda** to detect primary failure and promote the replica.
- Once promoted, the replica becomes a standalone DB — cannot revert. A new replica must be created from a snapshot.

### Aurora Global Database — Headless Clusters
- Can add a secondary region **without provisioning a compute instance**.
- Data still replicates via the storage volume (Aurora decouples compute and storage).
- Cost-efficient standby option.

---

## 4. Backup & Restore Strategy

Best for: **high RPO tolerance**, **cost-sensitive** workloads, compliance use cases.

### RDS Automated Backups
- Daily snapshot + continuous transaction log backup.
- **Retention: 1–35 days** (configurable).
- Supports **Point-in-Time Restore (PITR)** — restore to any 5-minute window within retention.

### Cross-Region Backup Replication
- Enable in RDS → Modify → Additional Configuration → **Backup Replication**.
- Choose destination region + retention period.
- Snapshots AND transaction logs are copied → supports PITR in another region.
- **Effective RPO: ~5 minutes** (limited by transaction log replication).
- **RTO: longer** — depends on snapshot restore + log replay time (can be hours if many logs need replaying).
- Fully managed by AWS; customer doesn't manage S3 buckets.

### Manual Snapshots
- No expiry — suitable for **long-term compliance retention** (e.g., 7 years).
- ⚠️ No PITR support — restores only to exact snapshot moment.
- For very old backups, consider exporting to **Amazon S3 Glacier** for lower storage cost.

### Backup RTO Consideration
- Restore time = snapshot restore time + transaction log replay time.
- If region fails 21 hours after last snapshot, **21 hours of logs** must be replayed — plan for this in RTO commitments.

---

## 5. Engine-Specific: Oracle on RDS

For **RPO/RTO < 30 minutes**, recommendation is **Read Replica** (not backup/restore).

| Feature | Read Replica | Mounted Read Replica |
|---|---|---|
| Serves read traffic | ✅ Yes | ❌ No |
| Oracle Enterprise Edition required | ✅ Yes | ✅ Yes |
| Active Data Guard license required | ✅ Yes | ❌ No (cost saving) |
| Use case | Offload reads + DR | Exclusive DR |

> Use **mounted read replica** to save on Active Data Guard licensing when DR-only is the goal.

---

## 6. Multi-AZ Failover — Deep Dive

- RDS Multi-AZ maintains a **synchronous standby** in a different AZ.
- Single endpoint — automatically updated via DNS/Route 53 after failover.
- **Failover triggers**:
  - Hardware/network failure
  - Health check failure (e.g., runaway query consuming all memory → health check fails → automatic failover)
- **Failover time**: typically **60–120 seconds** (< 2 minutes).
- Verify via NS lookup: IP changes after failover.

### Testing Failover
1. **RDS Console**: Actions → Reboot → Reboot with Failover (multi-AZ only)
2. **AWS Fault Injection Service (FIS)**: Simulate RDS failover in controlled/scheduled chaos engineering exercises (recommended for compliance demos and production resilience testing)

> 💡 **RDS Multi-AZ**: RPO = 0, RTO < 2 minutes.

---

## 7. Long-Running Queries & Replication Lag

- Long-running queries on the **source instance** can block replication commits on the replica (replication waits for query to complete).
- Increases replication lag → increases effective RPO.
- Mitigation: Set **query timeout settings** at the database engine level as a failsafe.

---

## 8. Holistic Application DR (Beyond the Database)

DR must cover the **entire application stack**, not just the database:

| Component | DR Approach |
|---|---|
| **EC2 instances** | AMI backups stored in DR region |
| **Containers** | Container image copies in DR region |
| **Static files** | S3 **Cross-Region Replication (CRR)** |
| **Infrastructure** | IaC templates (CloudFormation/Terraform) ready to provision in DR region |
| **Load balancers** | Provision via IaC (no backup needed) |
| **DNS / routing** | **Route 53 failover endpoints** + health checks |
| **Consolidated backup** | **AWS Backup** for centralized management |

> 💡 Always account for third-party service dependencies — even with a complete failover, external dependencies can still cause failures.

---

## 9. DR Strategy Summary Table

| Strategy | RPO | RTO | Cost | Use Case |
|---|---|---|---|---|
| Multi-AZ | ~0 | < 2 min | Medium | In-region HA, all engines |
| Aurora Global DB | Seconds | Minutes | Medium-High | Cross-region, low latency |
| Cross-Region Read Replica | Seconds–minutes | Minutes (manual) | Medium | Cross-region near-zero RPO |
| Cross-Region Backup (PITR) | ~5 min | 30 min – hours | Low | Cost-effective, tolerable RTO |
| Manual Snapshot | Hours–days | Hours | Very Low | Compliance, long retention |

---

## 10. Key Interview Talking Points

- **Always start with RPO/RTO requirements** — drive the entire architecture decision.
- **Zero RPO = synchronous replication = latency trade-off** — know when to push back.
- **Asynchronous replication lag = actual RPO** — monitor it with CloudWatch.
- **Cross-region read replica failover is manual** — automate with Route 53 + Lambda.
- **Aurora Global DB replicates at storage layer** — extremely fast, low overhead.
- **Multi-AZ is not cross-region DR** — know the boundary clearly.
- **PITR with cross-region backup = ~5 min RPO** but RTO depends on log replay volume.
- **Chaos engineering** (AWS FIS) is essential for validating DR plans.
- **Think holistically** — DR is not just databases; include S3, EC2/AMIs, DNS, IaC.
- **Mounted read replica** saves Oracle Active Data Guard licensing costs.

---

*Source: AWS re:Post Live — Database Disaster Recovery Strategy session*
