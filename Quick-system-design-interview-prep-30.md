# System Design — Interview Prep Guide
### 30 Core Concepts with AWS Service Mappings

> Covers the most important system design concepts for senior engineer / big tech interviews, with AWS equivalents for each concept.

---

## Table of Contents

1. [Client-Server Architecture](#1-client-server-architecture)
2. [IP Addresses & DNS](#2-ip-addresses--dns)
3. [Proxy & Reverse Proxy](#3-proxy--reverse-proxy)
4. [Latency & Geographic Distribution](#4-latency--geographic-distribution)
5. [HTTP & HTTPS](#5-http--https)
6. [APIs — REST vs GraphQL](#6-apis--rest-vs-graphql)
7. [Databases — SQL vs NoSQL](#7-databases--sql-vs-nosql)
8. [Vertical Scaling vs Horizontal Scaling](#8-vertical-scaling-vs-horizontal-scaling)
9. [Load Balancing](#9-load-balancing)
10. [Database Indexing](#10-database-indexing)
11. [Database Replication](#11-database-replication)
12. [Database Sharding (Horizontal Partitioning)](#12-database-sharding-horizontal-partitioning)
13. [Vertical Partitioning](#13-vertical-partitioning)
14. [Caching](#14-caching)
15. [Denormalization](#15-denormalization)
16. [CAP Theorem](#16-cap-theorem)
17. [Blob Storage](#17-blob-storage)
18. [CDN — Content Delivery Network](#18-cdn--content-delivery-network)
19. [WebSockets](#19-websockets)
20. [Webhooks](#20-webhooks)
21. [Monolithic vs Microservices Architecture](#21-monolithic-vs-microservices-architecture)
22. [Message Queues](#22-message-queues)
23. [Rate Limiting](#23-rate-limiting)
24. [API Gateway](#24-api-gateway)
25. [Idempotency](#25-idempotency)

---

## 1. Client-Server Architecture

**Concept:** The foundation of almost every web application. A **client** (browser, mobile app, frontend) sends a request to a **server**, which processes it and returns a response.

**Flow:**
```
Client → Request → Server → Process → Response → Client
```

**AWS Services:**
- **Amazon EC2** — Virtual servers to run your backend application
- **AWS Elastic Beanstalk** — Managed platform to deploy and scale web apps without managing servers
- **AWS Lambda** — Serverless compute; server-side logic without provisioning servers
- **Amazon Lightsail** — Simple virtual servers for smaller applications

---

## 2. IP Addresses & DNS

**Concept:** Every server on the internet has a unique **IP address** (like a phone number). Instead of typing raw IPs, we use human-readable **domain names** (e.g., `example.com`). The **Domain Name System (DNS)** maps domain names to IP addresses.

**Flow:**
```
User types domain → DNS lookup → IP address returned → Browser connects to server
```

**AWS Services:**
- **Amazon Route 53** — AWS's scalable DNS service; maps domain names to IPs, supports health checks, routing policies (latency-based, geolocation, weighted, failover)
- **AWS Global Accelerator** — Uses Anycast IPs to route traffic to optimal AWS endpoints globally

> 💡 **Interview tip:** Route 53 supports multiple routing policies — be ready to explain latency-based routing vs. geolocation routing vs. failover routing.

---

## 3. Proxy & Reverse Proxy

**Concept:**
- **Proxy (Forward Proxy):** Sits between the client and internet. Hides the client's identity (IP). Used for anonymity, content filtering, access control.
- **Reverse Proxy:** Sits between the internet and backend servers. Intercepts incoming client requests and forwards them to the appropriate backend server. Hides server details from clients.

**Reverse Proxy capabilities:** SSL termination, load balancing, caching, request routing, authentication offloading.

**AWS Services:**
- **AWS WAF (Web Application Firewall)** — Acts as a reverse proxy layer filtering malicious traffic
- **Amazon CloudFront** — Acts as a reverse proxy + CDN; caches and routes traffic
- **AWS Application Load Balancer (ALB)** — Reverse proxy with Layer 7 routing (HTTP/HTTPS)
- **AWS Network Load Balancer (NLB)** — Layer 4 reverse proxy for high-performance TCP/UDP traffic
- **AWS PrivateLink** — Secure proxy for internal service-to-service communication

---

## 4. Latency & Geographic Distribution

**Concept:** **Latency** is the round-trip delay between a client request and server response. Physical distance is a major cause — a user in India hitting a server in New York experiences high latency. Solution: distribute services across multiple data centers/regions.

**AWS Services:**
- **AWS Regions & Availability Zones** — Deploy workloads globally across 30+ regions
- **Amazon CloudFront** — CDN with 400+ edge locations; serves content from nearest PoP
- **AWS Global Accelerator** — Routes traffic through AWS backbone network, reducing internet latency
- **Amazon Route 53 Latency-Based Routing** — Automatically routes users to the lowest-latency region

> 💡 **Key numbers to remember:** Within an AZ ~1ms latency; cross-AZ ~2ms; cross-region = tens to hundreds of ms.

---

## 5. HTTP & HTTPS

**Concept:**
- **HTTP** is the protocol for client-server communication on the web. It follows a request-response model. Headers carry metadata (content type, cookies, auth tokens). Body carries data.
- **HTTPS** = HTTP + **TLS/SSL encryption**. Prevents eavesdropping and tampering. All modern applications must use HTTPS.

**HTTP Methods:**
| Method | Purpose |
|--------|---------|
| GET | Retrieve data |
| POST | Create new data |
| PUT / PATCH | Update existing data |
| DELETE | Remove data |

**AWS Services:**
- **AWS Certificate Manager (ACM)** — Free SSL/TLS certificate provisioning and renewal for HTTPS
- **Amazon CloudFront** — Enforces HTTPS at the edge; HTTPS offloading
- **Elastic Load Balancing (ALB)** — Handles HTTPS termination, forwards HTTP to backend
- **AWS WAF** — Inspects HTTP/HTTPS traffic for attacks (SQLi, XSS)

---

## 6. APIs — REST vs GraphQL

**Concept:** APIs define how clients and servers communicate. Two dominant styles:

### REST (Representational State Transfer)
- Stateless — every request is independent
- Resources modeled as URLs (`/users`, `/orders`)
- Uses standard HTTP methods (GET, POST, PUT, DELETE)
- Response in JSON or XML
- **Limitation:** Over-fetching (returns more data than needed) or under-fetching (requires multiple requests)

### GraphQL
- Introduced by Facebook in 2015
- Client specifies **exactly** what fields it needs — no over/under-fetching
- Single endpoint; one query can fetch nested/related data in one request
- **Trade-off:** More server-side processing; harder to cache than REST

**AWS Services:**
- **Amazon API Gateway** — Build, deploy, manage REST and WebSocket APIs; supports usage plans and throttling
- **AWS AppSync** — Fully managed **GraphQL** service; connects to DynamoDB, Lambda, RDS, and more
- **AWS Lambda** — Backend logic behind API Gateway or AppSync endpoints

> 💡 **Interview tip:** Choose REST for public APIs with simpler data models. Choose GraphQL when clients need flexible querying of complex, nested data (e.g., mobile apps with varying bandwidth).

---

## 7. Databases — SQL vs NoSQL

**Concept:**

### SQL (Relational Databases)
- Fixed schema, tables with rows and columns
- **ACID** properties: Atomicity, Consistency, Isolation, Durability
- Best for: structured data, complex joins, strong consistency (banking, ERP, e-commerce orders)

### NoSQL Databases
- Flexible/no fixed schema
- Optimized for scale and speed over strict consistency
- Types:
  - **Key-Value** — simple lookups (sessions, caching)
  - **Document** — JSON-like documents (user profiles, catalogs)
  - **Wide-Column** — time-series, event logs
  - **Graph** — relationships (social networks, recommendation engines)

**AWS SQL Services:**
- **Amazon RDS** — Managed relational DB: MySQL, PostgreSQL, MariaDB, Oracle, SQL Server
- **Amazon Aurora** — AWS-optimized MySQL/PostgreSQL; up to 5x faster than MySQL
- **Amazon Redshift** — SQL data warehouse for analytics (OLAP)

**AWS NoSQL Services:**
- **Amazon DynamoDB** — Key-value + document; single-digit millisecond latency at any scale
- **Amazon ElastiCache (Redis / Memcached)** — In-memory key-value store (caching layer)
- **Amazon DocumentDB** — MongoDB-compatible document database
- **Amazon Keyspaces** — Apache Cassandra-compatible wide-column store
- **Amazon Neptune** — Graph database for highly connected datasets
- **Amazon Timestream** — Purpose-built time-series database

> 💡 **When to use both:** Use RDS/Aurora for transactional writes + DynamoDB or ElastiCache for high-speed reads.

---

## 8. Vertical Scaling vs Horizontal Scaling

**Concept:**

| | Vertical Scaling (Scale Up) | Horizontal Scaling (Scale Out) |
|---|---|---|
| What | Add more CPU, RAM, storage to one machine | Add more servers to share the load |
| Pros | Simple, no code changes | Highly scalable, fault tolerant |
| Cons | Hard limit on max capacity; single point of failure; exponentially expensive | Requires load balancer; complexity increases |
| Best for | Quick fix, small apps | Large-scale, production systems |

**AWS Services:**
- **Vertical:** Change EC2 instance type (e.g., `t3.medium` → `r6i.4xlarge`); RDS instance class upgrade
- **Horizontal:**
  - **Amazon EC2 Auto Scaling** — Automatically add/remove EC2 instances based on demand
  - **AWS Application Auto Scaling** — Scale ECS tasks, DynamoDB, Lambda concurrency, etc.
  - **Amazon ECS / EKS** — Container orchestration that scales microservices horizontally

---

## 9. Load Balancing

**Concept:** A **load balancer** sits between clients and servers, distributing incoming requests across multiple backend servers. It improves availability and fault tolerance — if one server goes down, traffic is rerouted to healthy servers.

**Load Balancing Algorithms:**
- **Round Robin** — Distribute requests evenly in rotation
- **Least Connections** — Route to the server with fewest active connections
- **IP Hashing** — Route the same client IP to the same server (session stickiness)

**AWS Services:**
- **Application Load Balancer (ALB)** — Layer 7; HTTP/HTTPS routing by path, headers, host; ideal for microservices and containers
- **Network Load Balancer (NLB)** — Layer 4; ultra-high performance TCP/UDP; millions of requests/second
- **Gateway Load Balancer (GWLB)** — For deploying and scaling third-party network appliances (firewalls, IDS)
- **Classic Load Balancer (CLB)** — Legacy; avoid for new architectures

> 💡 **Interview tip:** For microservices on ECS/EKS, use ALB with path-based routing. For gaming or financial apps requiring ultra-low latency, use NLB.

---

## 10. Database Indexing

**Concept:** An **index** is a data structure that allows the database to find rows quickly without scanning the entire table — like a book index. Created on frequently queried columns (primary keys, foreign keys, WHERE clause columns).

**Trade-off:**
- ✅ Speeds up **READ** queries significantly
- ❌ Slows down **WRITE** operations (index must be updated on every insert/update/delete)
- ❌ Increases storage overhead

**AWS Services:**
- **Amazon RDS / Aurora** — Support standard B-tree and full-text indexes; use Performance Insights to identify slow queries
- **Amazon DynamoDB** — Primary key index + **Global Secondary Indexes (GSI)** and **Local Secondary Indexes (LSI)** for flexible querying
- **Amazon OpenSearch Service** — Full-text search indexing for unstructured data
- **Amazon Aurora Query Plan Management** — Captures and stabilizes query execution plans

> 💡 **Interview tip:** Only index columns you actually query. Too many indexes = degraded write performance. In DynamoDB, design your access patterns upfront to choose the right GSI.

---

## 11. Database Replication

**Concept:** Create copies of the primary database across multiple servers. One **primary** handles all writes; multiple **read replicas** handle read queries. Improves read performance and availability.

**Benefits:**
- Read scalability — distribute read load across replicas
- High availability — if primary fails, a replica can be promoted
- Geographic distribution — replicas in different regions reduce read latency

**AWS Services:**
- **Amazon RDS Read Replicas** — Up to 15 read replicas; supports cross-region replicas
- **Amazon Aurora Read Replicas** — Up to 15 replicas; sub-10ms replica lag; automatic failover
- **Amazon RDS Multi-AZ** — Synchronous standby replica in another AZ; automatic failover (RPO=0, RTO < 2 min)
- **Aurora Global Database** — Cross-region replication via storage layer; typical lag < 1 second
- **Amazon DynamoDB Global Tables** — Multi-region active-active replication for DynamoDB

---

## 12. Database Sharding (Horizontal Partitioning)

**Concept:** Split a large database into smaller pieces called **shards**, distributed across multiple servers. Each shard contains a subset of rows, determined by a **sharding key** (e.g., `user_id`, `region`).

**Benefits:**
- Handles write-heavy workloads at scale
- Reduces query load per server
- Enables storing terabytes/petabytes of data

**Trade-offs:**
- Complex to implement and maintain
- Cross-shard queries are expensive
- Rebalancing shards when data grows unevenly is hard

**AWS Services:**
- **Amazon DynamoDB** — Automatically partitions data internally based on partition key; no manual sharding needed
- **Amazon Aurora Limitless Database** — Horizontal sharding built into Aurora for massive write scale
- **Amazon Keyspaces (Cassandra)** — Natively shards across partitions using partition keys
- **Amazon Redshift** — Distributes data across compute nodes using distribution keys (similar to sharding)

---

## 13. Vertical Partitioning

**Concept:** Split a table **by columns** rather than rows. A large table (e.g., user table with profile, login history, billing) is split into smaller focused tables. Each query only scans relevant columns, reducing disk I/O.

**Example:** `users_profile`, `users_login_history`, `users_billing` — instead of one giant `users` table.

**AWS Services:**
- **Amazon RDS / Aurora** — Standard schema design; split tables and use foreign keys
- **Amazon Redshift** — Columnar storage naturally benefits from vertical partitioning; select only needed columns for analytical queries
- **Amazon DynamoDB** — Store different attribute groups in separate tables or use sparse indexes

---

## 14. Caching

**Concept:** Store frequently accessed data **in memory** to avoid repeated expensive database calls. Dramatically reduces response time and database load.

**Cache-Aside Pattern (most common):**
```
Request → Check cache → HIT: return data
                      → MISS: query DB → store in cache → return data
```

**TTL (Time-to-Live):** Expiry time set on cached entries to prevent serving stale data.

**What to cache:** Session data, user profiles, product catalogs, computed results, API responses.

**AWS Services:**
- **Amazon ElastiCache for Redis** — In-memory data store; supports TTL, pub/sub, sorted sets, persistence; ideal for sessions, leaderboards, real-time counters
- **Amazon ElastiCache for Memcached** — Simple high-performance caching; multi-threaded; no persistence
- **Amazon DynamoDB Accelerator (DAX)** — In-memory cache purpose-built for DynamoDB; microsecond read latency
- **Amazon CloudFront** — Edge caching for HTTP responses, static assets, API responses

> 💡 **Interview tip:** Cache invalidation is hard. Know when to use TTL vs. event-driven invalidation. For user-specific data, use Redis with user_id as the key prefix.

---

## 15. Denormalization

**Concept:** Combine related data into a **single table** to avoid expensive JOIN operations. Trades storage efficiency for read performance. Best for **read-heavy** workloads.

**Example:** Instead of `users` + `orders` tables joined at query time, create a single `user_orders` table with user details + recent orders pre-merged.

**Trade-offs:**
- ✅ Faster reads (no joins)
- ❌ Data duplication
- ❌ More complex updates (must update all copies)

**AWS Services:**
- **Amazon DynamoDB** — Designed around denormalization; single-table design patterns avoid joins
- **Amazon Redshift** — Denormalized fact/dimension tables in star schema for analytics
- **Amazon Aurora / RDS** — Use materialized views to pre-compute and cache join results

---

## 16. CAP Theorem

**Concept:** In a **distributed system**, you can only guarantee **2 out of 3** properties simultaneously:

| Property | Meaning |
|---|---|
| **Consistency (C)** | Every read receives the most recent write |
| **Availability (A)** | Every request receives a response (not always latest data) |
| **Partition Tolerance (P)** | System continues operating even if network partitions occur |

Since **network partitions are inevitable**, you must choose between:
- **CP (Consistency + Partition Tolerance):** May refuse to respond to remain consistent — banking, financial systems
- **AP (Availability + Partition Tolerance):** Always responds but may return stale data — social feeds, DNS, shopping carts

**AWS Services by CAP profile:**
| Service | Type |
|---|---|
| **Amazon RDS Multi-AZ** | CP — strong consistency, may be briefly unavailable during failover |
| **Amazon DynamoDB (default)** | AP — eventually consistent reads by default |
| **Amazon DynamoDB (strong consistency option)** | CP — strongly consistent reads at higher cost |
| **Aurora Global Database** | AP — eventual consistency across regions |
| **Amazon S3** | AP — strong read-after-write consistency (since 2020) |

---

## 17. Blob Storage

**Concept:** Traditional databases aren't designed for large unstructured files (images, videos, PDFs). **Blob (Binary Large Object) storage** stores files in logical containers (buckets), each with a unique URL.

**Advantages:** Scalable, pay-as-you-go, automatic replication, easy global access, built-in CDN integration.

**AWS Services:**
- **Amazon S3 (Simple Storage Service)** — Primary blob/object storage; 11 nines durability; versioning, lifecycle policies, cross-region replication
- **Amazon S3 Glacier / S3 Glacier Deep Archive** — Low-cost archival storage for compliance and long-term retention
- **Amazon EFS (Elastic File System)** — Shared file storage for EC2 (NFS); for shared file use cases
- **Amazon FSx** — Managed file systems (Windows, Lustre, NetApp ONTAP)

**S3 Storage Classes:**
| Class | Use Case |
|---|---|
| S3 Standard | Frequently accessed data |
| S3 Intelligent-Tiering | Unknown/changing access patterns |
| S3 Standard-IA | Infrequent access, rapid retrieval |
| S3 Glacier Instant | Archive with millisecond retrieval |
| S3 Glacier Deep Archive | Lowest cost, hours retrieval |

---

## 18. CDN — Content Delivery Network

**Concept:** A **CDN** is a global network of **edge servers** that cache and serve content (HTML, CSS, JS, images, video) from the location **closest to the user**, reducing latency and improving load times.

**How it works:**
```
User request → Nearest CDN edge → Cache HIT: serve content
                                → Cache MISS: fetch from origin → cache → serve
```

**AWS Services:**
- **Amazon CloudFront** — AWS's global CDN with 400+ Points of Presence (PoPs); integrates natively with S3, ALB, API Gateway, Lambda@Edge
- **Lambda@Edge** — Run serverless code at CloudFront edge locations (auth, redirects, A/B testing)
- **Amazon CloudFront Functions** — Lightweight JS functions at edge for header manipulation, URL rewrites
- **AWS Global Accelerator** — Routes traffic through AWS backbone (not CDN but reduces latency for dynamic content)

> 💡 **Interview tip:** CloudFront is for static/cacheable content. Global Accelerator is for dynamic, non-cacheable API traffic that benefits from AWS backbone routing.

---

## 19. WebSockets

**Concept:** HTTP's request-response model is inefficient for real-time apps (chat, live stock prices, multiplayer games). **WebSockets** establish a **persistent, bidirectional connection** between client and server — both sides can send messages at any time without a new request.

**HTTP Polling vs WebSockets:**
| | HTTP Polling | WebSockets |
|---|---|---|
| Connection | New connection per request | Single persistent connection |
| Direction | Client → Server only | Bidirectional |
| Efficiency | Wasteful (empty responses) | Efficient |
| Use cases | Simple data fetch | Real-time chat, live feeds, games |

**AWS Services:**
- **Amazon API Gateway WebSocket APIs** — Fully managed WebSocket API; routes messages to Lambda functions based on route keys
- **AWS AppSync** — Managed GraphQL with real-time subscriptions over WebSocket
- **Amazon IVS (Interactive Video Service)** — Real-time video streaming with low latency
- **AWS IoT Core** — MQTT-based persistent connections for IoT devices (similar pub/sub pattern)

---

## 20. Webhooks

**Concept:** Instead of polling an API repeatedly to check if an event occurred, **webhooks** let a provider push an HTTP POST to your registered URL the moment an event happens (e.g., payment completed, file uploaded).

**Flow:**
```
Your app registers webhook URL with provider
→ Event occurs (e.g., payment)
→ Provider sends HTTP POST to your URL with event payload
→ Your app processes the event
```

**AWS Services:**
- **Amazon SNS (Simple Notification Service)** — Push notifications to HTTP/HTTPS endpoints; fan-out to multiple subscribers
- **Amazon EventBridge** — Event bus; route events from AWS services or custom apps to targets (Lambda, SQS, API destinations)
- **AWS Lambda + API Gateway** — Build a webhook receiver endpoint that triggers serverless processing
- **Amazon SQS** — Buffer incoming webhook events to prevent overloading your processing backend

---

## 21. Monolithic vs Microservices Architecture

**Concept:**

| | Monolith | Microservices |
|---|---|---|
| Structure | Single codebase, all features together | Small independent services, each with own DB |
| Scaling | Must scale entire app | Scale individual services independently |
| Deployment | Deploy entire app | Deploy services independently |
| Failure | One bug can crash everything | Isolated failures |
| Complexity | Simple to start | Complex orchestration |
| Best for | Early-stage / small apps | Large-scale, high-growth systems |

**AWS Services for Microservices:**
- **Amazon ECS (Elastic Container Service)** — Run Docker containers for each microservice
- **Amazon EKS (Elastic Kubernetes Service)** — Kubernetes-based container orchestration
- **AWS Lambda** — Serverless microservices; each function = one service responsibility
- **AWS App Mesh** — Service mesh for microservices observability and traffic control
- **Amazon API Gateway** — Single entry point routing to microservices
- **AWS Cloud Map** — Service discovery for microservices

---

## 22. Message Queues

**Concept:** In a microservices architecture, direct synchronous API calls create tight coupling and scalability bottlenecks. A **message queue** enables **asynchronous communication** — a producer places a message, and a consumer processes it when ready. This decouples services and prevents overload.

**Flow:**
```
Producer → places message → Queue → Consumer retrieves & processes
```

**Benefits:** Decoupling, load leveling, retry logic, scalability, prevents data loss during spikes.

**AWS Services:**
- **Amazon SQS (Simple Queue Service)** — Fully managed message queue; Standard (at-least-once) and FIFO (exactly-once, ordered); dead-letter queues for failed messages
- **Amazon SNS (Simple Notification Service)** — Pub/sub; fan-out one message to multiple SQS queues or endpoints
- **Amazon MQ** — Managed Apache ActiveMQ and RabbitMQ (for lift-and-shift of existing messaging)
- **Amazon Kinesis Data Streams** — Real-time streaming for high-throughput event data (logs, clickstreams, IoT)
- **Amazon EventBridge** — Serverless event bus for event-driven architectures
- **Amazon MSK (Managed Streaming for Apache Kafka)** — Managed Kafka for high-throughput stream processing

> 💡 **SNS + SQS Fan-out Pattern:** SNS publishes once → multiple SQS queues receive → multiple consumers process independently. Classic pattern for decoupled notifications.

---

## 23. Rate Limiting

**Concept:** Restrict the number of requests a client (by user ID or IP) can make within a time window, protecting backend servers from abuse, DDoS, or runaway clients.

**Common Algorithms:**
| Algorithm | How it works |
|---|---|
| **Fixed Window** | Count requests in a fixed time window (e.g., 100 req/min) |
| **Sliding Window** | Rolling time window; smoother than fixed |
| **Token Bucket** | Tokens added at a rate; each request consumes a token; allows bursts |
| **Leaky Bucket** | Requests processed at a constant rate; excess queued or dropped |

**AWS Services:**
- **Amazon API Gateway** — Built-in throttling per stage, method, and API key; configurable burst and rate limits
- **AWS WAF (Web Application Firewall)** — Rate-based rules to block IPs exceeding request thresholds
- **AWS Shield** — DDoS protection (Standard free; Advanced paid) layered with rate limiting
- **Amazon CloudFront** — Edge-level rate limiting via WAF integration

---

## 24. API Gateway

**Concept:** A single **entry point** for all client requests in a microservices system. Handles cross-cutting concerns centrally:

- **Authentication & Authorization** — Validate tokens before forwarding requests
- **Rate Limiting & Throttling** — Prevent abuse
- **Request Routing** — Forward to the correct microservice
- **SSL Termination** — Handle HTTPS at the gateway
- **Logging & Monitoring** — Centralized observability
- **Request/Response Transformation** — Reformat payloads

**AWS Services:**
- **Amazon API Gateway** — Fully managed; supports REST APIs, HTTP APIs, and WebSocket APIs; integrates with Lambda, ECS, ALB, and any HTTP backend
- **AWS AppSync** — GraphQL API gateway with real-time subscriptions
- **Application Load Balancer (ALB)** — Can act as a lightweight API gateway with path-based and host-based routing
- **Amazon Cognito** — Integrates with API Gateway for JWT-based auth (user pools + identity pools)
- **AWS Lambda Authorizers** — Custom auth logic (API keys, JWTs, OAuth) plugged into API Gateway

> 💡 **API Gateway vs ALB:** Use API Gateway for serverless/Lambda backends with auth, throttling, and transformations. Use ALB for container/EC2 backends needing high throughput at lower cost per request.

---

## 25. Idempotency

**Concept:** An operation is **idempotent** if performing it multiple times produces the same result as doing it once. Critical in distributed systems where network failures can cause retries and duplicate requests.

**Example:** A user refreshes a payment page → server receives 2 payment requests → without idempotency, money is charged twice.

**How it works:**
```
Client generates unique Idempotency-Key per request
→ Server checks if key was already processed
→ If YES: return cached result (no duplicate processing)
→ If NO: process and store result with the key
```

**AWS Services:**
- **Amazon SQS FIFO Queues** — Built-in deduplication using `MessageDeduplicationId`; exactly-once processing
- **Amazon DynamoDB** — Store idempotency keys with TTL; use conditional writes (`ConditionExpression`) to prevent duplicates
- **AWS Lambda** — Idempotent function design by checking for existing records before processing
- **Amazon API Gateway** — Pass idempotency keys through to downstream services
- **Amazon S3** — PUT operations are idempotent by nature (same object, same key = same result)

---

## Quick Reference: AWS Services Map

| Concept | AWS Service(s) |
|---|---|
| Compute / Servers | EC2, Lambda, Elastic Beanstalk, ECS, EKS |
| DNS | Route 53 |
| Load Balancing | ALB, NLB, GWLB |
| CDN | CloudFront, Lambda@Edge |
| Object/Blob Storage | Amazon S3, S3 Glacier |
| SQL Database | RDS (MySQL, PostgreSQL, Oracle, SQL Server), Aurora |
| NoSQL Database | DynamoDB, DocumentDB, Keyspaces, Neptune, Timestream |
| In-Memory Cache | ElastiCache (Redis, Memcached), DAX |
| Data Warehouse | Amazon Redshift |
| Message Queue | SQS, SNS, Amazon MQ, Kinesis, EventBridge, MSK |
| API Gateway | Amazon API Gateway, AppSync, ALB |
| Authentication | Amazon Cognito, IAM, Lambda Authorizers |
| DDoS / WAF / Security | AWS WAF, AWS Shield, AWS ACM |
| Monitoring / Logging | Amazon CloudWatch, AWS X-Ray, CloudTrail |
| Auto Scaling | EC2 Auto Scaling, Application Auto Scaling |
| Global Routing | Route 53, AWS Global Accelerator |
| Real-Time Streaming | Amazon Kinesis, MSK (Managed Kafka) |
| Search | Amazon OpenSearch Service |
| Serverless Orchestration | AWS Step Functions |
| Infrastructure as Code | AWS CloudFormation, AWS CDK |

---

## Key Interview Talking Points

- **Always clarify requirements first** — ask about scale (QPS, data volume), consistency needs, read vs. write ratio, latency requirements.
- **Vertical scaling is a short-term fix** — always design for horizontal scaling with stateless services using EC2 Auto Scaling.
- **DNS + Load Balancer + Auto Scaling** is the standard horizontal scaling stack on AWS (Route 53 + ALB + ASG).
- **CAP theorem** — know which AWS services are CP vs AP; DynamoDB is AP by default but can be CP with strongly consistent reads.
- **Caching with ElastiCache/DAX** reduces DB load dramatically — mention it early in any system design discussion.
- **Async over sync** — prefer SQS/Kinesis/EventBridge for decoupling services; avoid tight synchronous chains.
- **WebSockets for real-time** — API Gateway WebSocket API + Lambda is the serverless pattern on AWS.
- **Idempotency** is a must for payment/order systems — mention SQS FIFO `MessageDeduplicationId` and DynamoDB conditional writes.
- **CloudFront + S3** is the go-to pattern for serving static content globally with minimal latency.
- **Aurora > RDS** for most new relational workloads — better performance, read replicas, global databases.
- **DynamoDB** for serverless, high-scale NoSQL — design access patterns first, then table design; use GSIs wisely.
- **API Gateway is the front door** — authentication (Cognito), throttling, routing all handled centrally.
- **SNS + SQS fan-out** is the canonical AWS decoupling pattern for microservices event propagation.

---

*Source: System Design — 30 Core Concepts (transcript)*
