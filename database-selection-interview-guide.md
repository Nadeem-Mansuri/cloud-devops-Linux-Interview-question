# Database Selection — Interview Reference Guide

> Covers: RDBMS vs NoSQL, ACID properties, joins, NoSQL types, and decision framework.

---

## 1. Two Main Categories

| Category | Structure | Best For |
|---|---|---|
| **Relational (RDBMS)** | Tables with rows & columns | Structured data, complex relationships, strong consistency |
| **Non-Relational (NoSQL)** | Flexible (documents, key-value, wide-column, graph) | Unstructured/semi-structured data, scale, low latency |

**Popular Examples:**
- **RDBMS:** PostgreSQL, MySQL, Oracle Database, SQLite
- **NoSQL:** MongoDB, Cassandra, Redis, Neo4j

---

## 2. Relational Databases (SQL)

### Core Structure
Data is organized into **tables**, similar to spreadsheets:

| Element | Description |
|---|---|
| **Table** | Fundamental building block; holds related data |
| **Column** | A field/attribute of the table (e.g., ID, name, age) |
| **Row** | A single record within the table |

**Example — `customers` table:**

| ID | Name | Age | Email |
|---|---|---|---|
| 1 | John | 30 | john@email.com |
| 2 | ... | ... | ... |

### Key Advantages

#### a) Complex Join Operations
- SQL allows combining **two or more tables** into one via **join operations**.
- Example: A `customers` table + `products` table → joined into an `orders` table, which references `customer_id` and `product_id`.
- This enables normalized, non-redundant data storage across related tables.

#### b) Data Consistency & Integrity (ACID Transactions)
A **transaction** = a sequence of one or more SQL operations executed as a single atomic unit (e.g., a bank transfer).

| ACID Property | Meaning |
|---|---|
| **Atomicity** | The entire transaction either fully succeeds or fully fails — no partial execution |
| **Consistency** | Moves the database from one valid state to another valid state |
| **Isolation** | Concurrent transactions don't interfere with each other |
| **Durability** | Once committed, data persists even after a system/server failure |

> ⚠️ Common typo/mishearing to note: "ACT" acronym → correct term is **ACID**.

---

## 3. Non-Relational Databases (NoSQL)

### The Four Main Types

| Type | Storage Model | Examples | Best Use Case |
|---|---|---|---|
| **Document Store** | JSON-like documents | MongoDB | Complex, nested data in a single record |
| **Wide-Column Store** | Tables with rows & *dynamic* columns | Cassandra, Cosmos DB | Massive scale, heavy write throughput |
| **Graph Database** | Nodes (entities) + edges (relationships) | Neo4j, Amazon Neptune | Relationship-heavy data (e.g., recommendations) |
| **Key-Value Store** | Simple key → value pairs, often in-RAM | Redis, Memcached | Extreme speed, simple lookups |

### Notes per Type
- **Document Stores (MongoDB):** Store data in JSON-like documents, allowing complex nested structures within a single record.
- **Wide-Column Stores (Cassandra/Cosmos DB):** Handle massive scale and are optimized for high write throughput.
- **Graph Databases (Neo4j):** Focus on entities and their relationships. *Example: Amazon uses Neptune (a graph DB) to power product recommendations based on order history.*
- **Key-Value Stores (Redis/Memcached):** Primarily stored in RAM → extremely fast reads/writes. Advantage = simplicity + speed.

### Key Advantage: Denormalized/Embedded Data
Using the same customer/product/order example:
- In MongoDB, you could store **user data, orders, and products all in a single document** — no joins required.
- This lets NoSQL handle **highly dynamic, large datasets** without the rigid structure SQL imposes.
- Optimized for **low latency** and **horizontal scalability**.

---

## 4. Decision Framework: SQL vs NoSQL

| If your application needs... | Choose | Example |
|---|---|---|
| Well-structured data with clear relationships | **SQL** | E-commerce app tracking customers & orders |
| Strong consistency & transactional integrity | **SQL** | Financial / banking system |
| Super low latency for quick responses | **NoSQL** | Real-time apps |
| Unstructured/semi-structured data, relationships not critical | **NoSQL** | Storing JSON objects |
| Flexible, scalable storage for massive data volumes | **NoSQL** | Recommendation engine storing user activity (key-value) |

---

## 5. Quick-Reference Summary

```
RDBMS (SQL)                          NoSQL
─────────────────                    ─────────────────
Tables, rows, columns                Documents / Key-Value / Wide-Column / Graph
Fixed schema                         Flexible / dynamic schema
Joins across tables                  Embedded/denormalized data
ACID transactions                    Eventual consistency (typically)
Vertical scaling (traditionally)     Horizontal scaling
Best: structured + relational data   Best: unstructured, high-scale, low-latency data
```

### One-Line Litmus Test
- **Need joins + strong consistency?** → SQL
- **Need flexible schema + massive scale + speed?** → NoSQL

---

## 6. Likely Interview Questions & Answers

**1. What is the difference between SQL and NoSQL databases?**
SQL (relational) databases store data in structured tables with fixed schemas, rows, and columns, and use SQL to query it. They excel at complex joins and strong transactional consistency (ACID). NoSQL (non-relational) databases store data in flexible formats — documents, key-value pairs, wide-columns, or graphs — with dynamic schemas. They're optimized for horizontal scaling, low latency, and handling unstructured/semi-structured data at large volume.

**2. Explain ACID properties with a real-world example.**
ACID describes the guarantees a database transaction provides. Take a bank transfer from Account A to Account B:
- **Atomicity** — the debit from A and credit to B happen as one unit; if either fails, both are rolled back (money isn't lost mid-transfer).
- **Consistency** — the total balance across both accounts before and after the transfer stays mathematically valid.
- **Isolation** — if two transfers happen at the same time, they don't see each other's incomplete intermediate state.
- **Durability** — once the transfer is confirmed, it survives even if the server crashes a second later.

**3. What is a join in SQL? Why is it useful?**
A join combines rows from two or more tables based on a related column (e.g., a `customer_id` shared between `customers` and `orders`). It's useful because it lets you keep data normalized (no duplication) — you can store customers and products separately and only link them when needed, e.g., to produce a report of "which customer ordered which product."

**4. Name the four types of NoSQL databases with an example of each.**
- **Document store** — MongoDB (JSON-like documents)
- **Wide-column store** — Cassandra / Cosmos DB (tables with dynamic columns)
- **Key-value store** — Redis / Memcached (simple key → value pairs)
- **Graph database** — Neo4j / Amazon Neptune (nodes + relationships)

**5. When would you choose a document store over a wide-column store?**
Choose a **document store** (e.g., MongoDB) when your data is naturally hierarchical or varies in structure between records — like user profiles, product catalogs, or content management, where each "document" can have a different shape. Choose a **wide-column store** (e.g., Cassandra) when you need to handle massive write throughput and huge volumes of similarly structured time-series or log-style data across distributed clusters.

**6. Why are key-value stores so fast?**
Because they're typically held **in-memory (RAM)** rather than on disk, and the lookup is a simple direct key → value fetch with no query parsing, joins, or schema validation overhead. This simplicity is what makes stores like Redis and Memcached extremely fast for reads and writes.

**7. Give a real-world example of when you'd use a graph database.**
Amazon uses its graph database, **Neptune**, to power product recommendations — modeling customers, products, and purchase relationships as a graph makes it efficient to answer questions like "customers who bought X also bought Y" by traversing relationships rather than running expensive multi-table joins.

**8. How does MongoDB avoid the need for joins?**
MongoDB stores related data **embedded within a single JSON-like document** instead of splitting it across separate tables. For example, a single document could contain the customer's info plus their orders and the ordered products nested inside it — so a single read retrieves everything, with no join operation required.

**9. In an e-commerce system, would you use SQL or NoSQL for the orders table? Why?**
Generally **SQL**, because order data has clear structured relationships (customers → orders → products) and typically needs strong transactional integrity — e.g., you don't want an order to be marked "placed" if the payment or inventory deduction fails halfway through. ACID guarantees make SQL well-suited to this. (In practice, large-scale systems sometimes use a hybrid: SQL for the transactional order/payment core, and NoSQL for things like product catalogs, search, or activity logs where flexibility and scale matter more.)

**10. What trade-offs does NoSQL make to achieve scalability and low latency?**
NoSQL typically relaxes strict consistency in favor of **eventual consistency**, meaning that after a write, not all nodes may immediately reflect the latest value. It also generally sacrifices the ability to do complex, ACID-guaranteed multi-table joins and transactions in favor of denormalized, embedded data structures. In exchange, it gains horizontal scalability (adding more machines rather than a bigger single machine), higher write throughput, and lower latency for simple access patterns.
