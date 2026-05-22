# AWS Landing Zone — Complete Summary

> **Source:** [YouTube — Building Landing Zones (AWS)](https://www.youtube.com/watch?v=C14YrvV5hwI)
> **Presenter:** Alexi — AWS Ambassador, Community Builder & Leader of AWS User Group in Kyiv
> **Company:** ASOM — AWS Premier Tier Partner (founded 11 years ago in Israel, 8 locations, ~160 employees, 300+ AWS certifications, helped 500+ startups)

---

## Table of Contents

1. [History & Problem Statement](#1-history--problem-statement)
2. [What is a Landing Zone?](#2-what-is-a-landing-zone)
3. [AWS Control Tower — Setup](#3-aws-control-tower--setup)
4. [Account Structure](#4-account-structure)
5. [Security Configuration & Controls](#5-security-configuration--controls)
6. [Centralized Logging & SIEM](#6-centralized-logging--siem)
7. [Audit Account & Security Hub](#7-audit-account--security-hub)
8. [Infrastructure Organizational Unit](#8-infrastructure-organizational-unit)
9. [Centralized Networking](#9-centralized-networking)
10. [Landing Zone Customizations (CfCT)](#10-landing-zone-customizations-cfct)
11. [Full Architecture Diagram](#11-full-architecture-diagram)
12. [Key Takeaways](#12-key-takeaways)

---

## 1. History & Problem Statement

### The Old Way (Single Account)
- A team opens one AWS account and logs in via **IAM users**.
- They create resources: VMs, Lambda functions, databases, storage.
- The company grows → new teams join → **all teams share the same account**.
- Problems that emerge:
  - How do you distinguish workloads from different teams?
  - How do you efficiently manage permissions?
  - How do you deal with **service limits** — one team impacting another?

### The 2017 Turning Point — AWS Organizations
- AWS launched **AWS Organizations** to solve this.
- Each team gets its own **isolated AWS account**.
- All accounts are controlled by a single **Management Account**.
- Benefits introduced:
  - **Consolidated Billing** — see costs per team; get volume discounts (e.g., S3 charges cheaper as aggregate usage grows).
  - **Single Sign-On (SSO)** — centrally manage access to all accounts.
  - **Organizational Policies** — Service Control Policies (SCPs), tag policies, backup policies applied across all accounts.
  - **Centralized Security Aggregation** — security findings from all accounts in one place.
- AWS Organizations became the **baseline for the Landing Zone**.

---

## 2. What is a Landing Zone?

> A **Landing Zone** is a **multi-account AWS environment** built according to the AWS Well-Architected Framework — secure, scalable, and governed by best practices.

### Who Needs It?
If your company answers **YES** to any of the following, a Landing Zone is likely for you:

| Need | Description |
|---|---|
| Administrative isolation | Different teams access only what they need |
| Limited visibility | Workloads and environments isolated from each other |
| Minimize blast radius | A compromised account doesn't affect the whole org |
| Audit data isolation | Backups and logs stored securely and separately |

### Problems a Landing Zone Solves
- Apply **different security policies** for different workloads.
- Enforce **account-level isolation**.
- Limit access to **highly private data**.
- Manage teams with **different responsibilities and resource needs**.
- Separate **business units** with different processes.
- **Convenient billing** and spending visibility.
- Efficiently manage **service limits** — prevent one workload from starving another.

---

## 3. AWS Control Tower — Setup

**AWS Control Tower** is the AWS service that provisions and governs a Landing Zone.

### Setup Steps
1. Open the Control Tower console → click the single **"Set up"** button.
2. Fill in a few fields:
   - Email addresses for new accounts.
   - **Home region** for the Landing Zone.
   - **KMS key** for encrypting logs.
3. Click and **wait ~30 minutes**.

### What Happens During Setup
Control Tower automatically:
- Uses **AWS Organizations** as the baseline.
- Configures **IAM Identity Center (SSO)** with basic defaults.
- Uses **CloudFormation StackSets** and **Service Catalog** to create new accounts.
- Creates **2 new Organizational Units**:
  - **Security OU** → contains the `Log Archive` and `Audit` core accounts.
  - **Custom OU** → for your workloads.

### Result After Setup
- ✅ 2 new Organizational Units
- ✅ 3 new accounts (Management, Log Archive, Audit)
- ✅ Basic SSO configured
- ✅ Set of preventive & detective controls applied

---

## 4. Account Structure

### Core Accounts (Created by Control Tower)

```
Root
├── Management Account
│   ├── Consolidated Billing
│   ├── SSO / IAM Identity Center
│   ├── SCPs & Org Policies
│   └── Account Factory (for new accounts)
│
├── Security OU
│   ├── Log Archive Account
│   │   ├── S3 bucket: CloudTrail logs (all accounts)
│   │   └── S3 bucket: AWS Config logs (all accounts)
│   └── Audit Account
│       ├── AWS Config Aggregator
│       ├── SNS Topics (security notifications)
│       └── Delegated Admin for Security Hub, GuardDuty, Inspector, Macie
│
└── Custom OU (your workloads go here)
```

### Extended Account Structure (Common Pattern)

| OU | Accounts | Purpose |
|---|---|---|
| **Infrastructure OU** | Networking Account | Transit Gateway, Firewall, VPN, DNS |
| | Shared Services Account | Monitoring, Grafana, Prometheus, OpenSearch |
| | DevOps Account | Jenkins, ECR, Artifactory, CI/CD tools |
| | Backup Account | AWS Backup with cross-account backup |
| **Workload OU** | Production, Staging, Dev | Per-environment accounts per application |
| **Sandbox OU** | Per-developer accounts | Isolated experimentation; limited spend |
| **Suspended OU** | Blocked accounts | Accounts to be closed, hacked, or over-budget accounts |

---

## 5. Security Configuration & Controls

Control Tower offers **500+ controls** across 3 types:

### Type 1 — Detective Controls
- Built on **AWS Config Rules**.
- **200+ built-in rules** covering ~50 AWS services.
- Enable **Security Hub** to add **300+ more rules** + integrations.

**Example Security Hub Integrations:**

| Service | Function |
|---|---|
| **Amazon GuardDuty** | ML-based threat detection; analyzes activity patterns |
| **Amazon Inspector** | Scans OS and containers for vulnerabilities |
| **AWS Systems Manager Patch Manager** | Remediates vulnerabilities found by Inspector |
| **Amazon Macie** | Scans S3 buckets for sensitive data (credit card numbers, PII) |

**Example Detective Control:** Detects EBS volumes that are **not encrypted**. The console shows Volume ID, Region, Account, OU, and marks the account non-compliant.

---

### Type 2 — Proactive Controls
- Built on **CloudFormation Hooks**.
- **200+ built-in rules** — only relevant if you deploy via CloudFormation.
- Evaluates CloudFormation templates **before deployment**.
- Reduces human error and prevents compliance violations before they occur.

**Example Proactive Control:** Deploying an S3 bucket without enforced encryption-in-transit → the deployment **fails before creation**.

---

### Type 3 — Preventive Controls
- Built on **Service Control Policies (SCPs)**.
- **~50 built-in rules**.
- Defines **maximum permitted permissions** for member accounts.
- Applies even to **administrators** in member accounts.

**Examples:**
- Restrict deployments to specific **AWS Regions**.
- Limit permissions of users/roles even if they have Admin access.
- Restrict specific **service permissions** org-wide.

**Example Preventive Control:** An administrator tries to create an **unencrypted public snapshot** → receives `"You are not authorized to perform this operation"` error regardless of their IAM role.

---

## 6. Centralized Logging & SIEM

### Log Archive Account Flow

```
All Member Accounts
     │
     ├── AWS CloudTrail logs ──────────────────────────┐
     └── AWS Config logs ──────────────────────────────┤
                                                        ▼
                                             Log Archive Account
                                               (Central S3 Bucket)
                                                        │
                                             (Optional) SIEM Solution
                                             (e.g., OpenSearch-based)
```

### SIEM Solution (AWS Open Source on GitHub)
An AWS-developed SIEM built entirely on AWS services:

| Component | Role |
|---|---|
| **Amazon OpenSearch** | Core log storage and search engine |
| **AWS Lambda** | Normalizes/converts logs from various formats into OpenSearch format |
| **Built-in Dashboards** | Pre-built visualizations for security analysis |

### SIEM Dashboard Examples

**CloudTrail Log Analysis:**
- Summary per AWS account and region
- API calls by country, IP address, geo map
- Failed login attempts, root login attempts
- Unauthorized API calls
- Security group changes, Network ACL changes

**Load Balancer Access Logs:**
- Protocols, HTTP methods, IP addresses
- Request counts, URL paths, user agents
- Geographic distribution of web requests

**RDS Database Monitoring:**
- Source IP addresses, authentication failures
- Slow query detection
- Access patterns per account and region

---

## 7. Audit Account & Security Hub

The **Audit Account** is the central hub for all security services.

### Default Baseline (Created by Control Tower)
- Dedicated roles for **Administrators** and **Auditors**.
- Alert email address for notifications.
- **AWS Config Aggregator** (collects Config data from all accounts).
- CloudWatch alarms.

### As Delegated Administrator
The Audit Account acts as **Delegated Administrator** for:
- **AWS Security Hub** — aggregates all findings from all accounts/regions.
- **Amazon GuardDuty**
- **Amazon Inspector**
- **Amazon Macie**

### Security Hub Dashboard
- Multiple **security standards** enabled (CIS, AWS Foundational, PCI DSS, etc.).
- All findings from **all accounts and all regions** in one place.
- Filter by: severity, resource type, account, region.

---

## 8. Infrastructure Organizational Unit

This OU is **not provisioned by default** but is commonly used.

### Backup Account
- Uses **AWS Backup** service for centralized backup management.
- Management Account applies **Backup Policies**.
- Backup plans automatically appear in **member accounts** — cannot be deleted or modified by member account users.
- Backup plan includes:
  - Schedule for backup creation.
  - Resources to back up (EC2, RDS, S3, etc. — 16 AWS services supported).
  - Rotation/retention policies.
  - Cross-region and cross-account copy options.

### DevOps Account
- Hosts CI/CD tools: **Jenkins** or other build servers.
- Artifact storage: **Amazon ECR**, **Artifactory**, etc.
- Centralized: deploys artifacts to member accounts.

### Shared Services Account
- **Amazon OpenSearch** — centralized log storage for all accounts.
- **Prometheus** — centralized metrics collection.
- **Grafana** — unified monitoring dashboards.
- Provides access to the **support team** without granting production access.

---

## 9. Centralized Networking

All networking resources live in a **dedicated Networking Account**.

### Core Components

| Component | Purpose |
|---|---|
| **AWS Transit Gateway** | Shared hub connecting all member VPCs |
| **VPN (Site-to-Site)** | Connect on-premises data centers and remote offices |
| **IP Address Management (IPAM)** | Allocate CIDR blocks; prevent overlapping ranges |
| **AWS Network Firewall / 3rd Party** | Central traffic filtering (Palo Alto, Check Point) |
| **Central VPN** | Remote engineers access private resources |
| **Amazon Route 53** | Centralized DNS management |

### Centralized Egress Architecture Options

**Option A — Simple Egress:**
```
Spoke VPCs → Transit Gateway → Central Ingress/Egress VPC → Internet
(No NAT Gateways needed in each member VPC — saves cost)
```

**Option B — Egress with Firewall (Advanced):**
```
Spoke VPCs → Transit Gateway → Firewall → Central Egress VPC → Internet
(Traffic is filtered before reaching the internet)
```

### Network Orchestration Automation (AWS Solution)
AWS provides an open-source solution for **automatic Transit Gateway attachment**:

**Hub (deployed in Networking Account):**
- Transit Gateway
- Step Functions + Lambda automation
- DynamoDB (state storage)
- SNS (notifications)
- Optional Web UI for networking team to approve/reject attachment requests

**Spoke (deployed in all member accounts):**
- EventBridge rule monitors VPC tags
- When a VPC/subnet is tagged with the appropriate tag → event fires → hub automation runs → VPC is automatically attached to Transit Gateway with correct routes

---

## 10. Landing Zone Customizations (CfCT)

**Customizations for Control Tower (CfCT)** is an AWS solution available on GitHub for automating Landing Zone configuration.

### How It Works

```
Git Repository (Source of Truth)
├── CloudFormation Templates    → Infrastructure definitions
├── Service Control Policies    → JSON-formatted SCPs
└── manifest.yaml               → Defines WHERE and WHAT to deploy
         │
         ▼
    CfCT Pipeline (CodePipeline)
         │
         ▼
    Deployed to Accounts/OUs    → via CloudFormation StackSets
```

- Configuration is stored in a **Git repository** as code (GitOps approach).
- The **manifest file** defines: which templates deploy to which accounts/OUs.
- Fully automated — no manual intervention required per account.

---

## 11. Full Architecture Diagram

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                           AWS ORGANIZATION (ROOT)                           │
│                                                                             │
│  ┌─────────────────────────────────────────────────────────────────────┐   │
│  │                      MANAGEMENT ACCOUNT                             │   │
│  │  • Consolidated Billing   • SSO / IAM Identity Center              │   │
│  │  • SCPs & Backup Policies • Account Factory (Control Tower)        │   │
│  │  • CfCT Pipeline (Git → CloudFormation StackSets)                  │   │
│  └─────────────────────────────────────────────────────────────────────┘   │
│                                                                             │
│  ┌───────────────────────┐   ┌────────────────────────────────────────┐   │
│  │      SECURITY OU      │   │           INFRASTRUCTURE OU            │   │
│  │                       │   │                                        │   │
│  │  ┌─────────────────┐  │   │  ┌──────────┐  ┌──────────────────┐  │   │
│  │  │  Log Archive    │  │   │  │ Networking│  │  Shared Services │  │   │
│  │  │  • CloudTrail   │  │   │  │ • TGW    │  │  • OpenSearch    │  │   │
│  │  │  • Config Logs  │  │   │  │ • VPN    │  │  • Prometheus    │  │   │
│  │  │  • SIEM         │  │   │  │ • IPAM   │  │  • Grafana       │  │   │
│  │  │  (OpenSearch)   │  │   │  │ • Firewall│  └──────────────────┘  │   │
│  │  └─────────────────┘  │   │  │ • Route53│  ┌──────────────────┐  │   │
│  │                       │   │  └──────────┘  │    DevOps        │  │   │
│  │  ┌─────────────────┐  │   │                │  • Jenkins / ECR │  │   │
│  │  │  Audit Account  │  │   │  ┌──────────┐  └──────────────────┘  │   │
│  │  │  • Config Agg.  │  │   │  │  Backup  │  ┌──────────────────┐  │   │
│  │  │  • Security Hub │  │   │  │  Account │  │     (others)     │  │   │
│  │  │  • GuardDuty    │  │   │  └──────────┘  └──────────────────┘  │   │
│  │  │  • Inspector    │  │   └────────────────────────────────────────┘   │
│  │  │  • Macie        │  │                                                 │
│  │  └─────────────────┘  │   ┌────────────────────────────────────────┐   │
│  └───────────────────────┘   │             WORKLOAD OU                │   │
│                              │  ┌──────────┐  ┌──────────┐            │   │
│                              │  │Production│  │ Staging  │  (Dev...)  │   │
│                              │  │ Account  │  │ Account  │            │   │
│                              │  └──────────┘  └──────────┘            │   │
│                              └────────────────────────────────────────┘   │
│                                                                             │
│  ┌──────────────────────┐    ┌────────────────────────────────────────┐   │
│  │      SANDBOX OU      │    │              SUSPENDED OU              │   │
│  │  • Dev sandbox accts │    │  • SCP: Deny All                      │   │
│  │  • Spend limits      │    │  • Accounts pending closure           │   │
│  │  • No network access │    │  • Hacked / over-budget accounts      │   │
│  └──────────────────────┘    └────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Control Types Summary

```
┌─────────────────────────────────────────────────────────────────┐
│                    CONTROL TOWER CONTROLS                        │
├───────────────────┬──────────────────────┬──────────────────────┤
│  DETECTIVE        │  PROACTIVE           │  PREVENTIVE          │
│  (AWS Config)     │  (CloudFormation     │  (Service Control    │
│                   │   Hooks)             │   Policies)          │
├───────────────────┼──────────────────────┼──────────────────────┤
│ 200+ built-in     │ 200+ built-in        │ ~50 built-in         │
│ + 300 via Sec Hub │ (requires CFN use)   │                      │
├───────────────────┼──────────────────────┼──────────────────────┤
│ Detects after     │ Blocks before        │ Prevents always      │
│ the fact          │ deployment           │ (even for admins)    │
├───────────────────┼──────────────────────┼──────────────────────┤
│ Ex: EBS volume    │ Ex: S3 bucket        │ Ex: Block unencrypted│
│ not encrypted     │ without TLS policy   │ public snapshots     │
└───────────────────┴──────────────────────┴──────────────────────┘
```

---

## 12. Key Takeaways

1. **Start with Control Tower** — it automates the heavy lifting of Landing Zone setup in ~30 minutes.
2. **3 core accounts are mandatory**: Management, Log Archive, Audit — everything else is optional but recommended.
3. **Use all 3 control types together** — Detective + Proactive + Preventive for a defense-in-depth strategy.
4. **Centralize early**: logging, networking, security, and devops tooling are easier to centralize from the start than to migrate later.
5. **Automate with CfCT** — treat your Landing Zone config as code (GitOps) to ensure consistency and auditability.
6. **Sandbox and Suspended OUs** are underutilized but critical — Sandbox for safe experimentation, Suspended for incident response.
7. **The Audit Account is your security nerve center** — delegate all security services there for a single pane of glass.
8. **Transit Gateway + Network Orchestration** removes per-account VPC networking complexity.
9. **SSO/IAM Identity Center** connects to your existing identity provider (Azure AD, Okta, JumpCloud, Google Workspace) — no need to start from scratch.
10. **Landing Zone scales with you** — start small, add accounts/OUs/controls as your organization grows.
