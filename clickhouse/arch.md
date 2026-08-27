# ClickHouse Architecture — Explained Like You're New to This

*A trainer-to-learner walkthrough. No prior database internals knowledge needed.*

> Based on: ClickHouse Architecture Overview (VLDB 2024 research paper, Schulze et al.)

---

## Before We Start: What IS ClickHouse?

Think of ClickHouse as a **super-fast calculator for huge tables of data**. Where a normal database (like MySQL) is great at "find me this one customer's record," ClickHouse is built for "scan 5 billion rows and give me the average sale price by city, right now."

This type of database is called **OLAP** (Online Analytical Processing) — built for big analytical questions, not tiny individual lookups.

And here's a neat fact: ClickHouse ships as **one single binary file**, written in C++, with no external dependencies. You download one file, and everything — the server, the query engine, the storage system — is inside it. No separate services to install.

---

## The Big Picture: 4 Layers Working Together

Imagine a restaurant. A customer walks in, talks to a host, places an order, the kitchen cooks it, and ingredients come from a supply room. ClickHouse works the same way — a request comes in, gets processed, and touches real stored data. It has **4 layers**:

```
┌─────────────────────────────────────────────┐
│              ACCESS LAYER                    │   <- greets you at the door
│   (native protocol, MySQL, PostgreSQL, HTTP) │
├─────────────────────────────────────────────┤
│         QUERY PROCESSING LAYER               │   <- the kitchen (does the work)
│     (parses, optimizes, executes queries)     │
├─────────────────────────────────────────────┤
│             STORAGE LAYER                     │   <- the pantry (where data lives)
│         (table engines: how/where data sits)  │
├─────────────────────────────────────────────┤
│           INTEGRATION LAYER                   │   <- the delivery service
│  (connects to external DBs, Kafka, S3, etc.)  │
└─────────────────────────────────────────────┘
```

And running *across* all four layers, like the restaurant's shared systems (staffing, security cameras, backup generators), sit some **cross-cutting components**:
- **Threading** — manages workers doing the tasks
- **Caching** — remembers recent answers so it doesn't redo work
- **Role-based access control** — who's allowed to do what
- **Backups** — safety net for your data
- **Continuous monitoring** — keeps an eye on health

Let's walk through each layer one at a time.

---

## Layer 1: The Access Layer (the "front door")

**Job:** Let clients talk to ClickHouse, in whatever language they speak.

ClickHouse doesn't force everyone to use one protocol. It understands:
- Its own **native protocol** (fastest, ClickHouse-specific)
- **MySQL** protocol
- **PostgreSQL** protocol
- **HTTP** (great for simple tools, browsers, curl commands)

**Analogy:** It's like a receptionist who speaks four languages fluently, so no matter which language a visitor walks in speaking, they're understood immediately — no translator needed.

It also manages **user sessions** — keeping track of who's connected and what they're allowed to do.

---

## Layer 2: The Query Processing Layer (the "brain")

**Job:** Take your SQL question, figure out the smartest way to answer it, and actually run it.

This happens in three steps:
1. **Parse** — read your query and understand its structure
2. **Optimize** — decide the *fastest* way to execute it (which indexes to use, what order to do things)
3. **Execute** — actually run it and produce results

You're not limited to plain SQL either — ClickHouse also understands **PRQL** and **Kusto's KQL** as alternative query languages.

**The special trick: vectorized execution.**
Instead of processing data one row at a time (slow, like reading a book one letter at a time), ClickHouse processes data in **batches (vectors)** — similar to a technique used in a database engine called MonetDB/X100. It also does **opportunistic runtime code compilation**, meaning it can generate specialized, faster code on the fly for your specific query.

**Analogy:** Instead of a chef cooking one dumpling at a time, they cook an entire tray of 100 dumplings simultaneously in one motion. Same idea — much faster throughput.

---

## Layer 3: The Storage Layer (the "pantry")

**Job:** Actually store your data on disk, in a format optimized for fast reads.

This is arguably the most important part of ClickHouse's design. There are **three categories of table engines** here:

### 1. MergeTree* family — the primary storage format
This is the default, most-used engine family. Key idea: 
- Data is split into **sorted "parts"**
- These parts are **continuously merged in the background** (like a library periodically reorganizing loose books back onto shelves in order)
- It's inspired by a data structure called an **LSM-tree** (Log-Structured Merge-tree)
- Different MergeTree variants handle rows differently — some aggregate, some replace duplicates, some just keep everything as-is

### 2. Special-purpose engines — for speed or distribution
- Includes in-memory engines for temporary/cache-like use
- Includes the **Distributed engine**, which gives you a single unified view over data that's actually split across many machines (more on this below!)

### 3. Virtual / integration engines — talk to the outside world
- Connect to relational databases (PostgreSQL, MySQL)
- Connect to message queues (Kafka, RabbitMQ)
- Connect to key-value stores (Redis)
- Connect to data lakes (Iceberg, Delta Lake, Hudi)
- Connect to object storage (S3, Google Cloud Storage)

**Analogy:** Think of a big grocery store's warehouse. MergeTree engines are the main shelves where products are neatly sorted and restocked overnight. Special-purpose engines are the "fast checkout" fridges near the front. Integration engines are like a direct pipe to the supplier's truck — no need to unload it onto your own shelves first.

---

## Sharding & Replication — Scaling Out

Now here's where it gets interesting: what happens when your data is *too big* for one machine, or you need it to survive a machine crashing?

### Sharding = splitting the data
- A table gets **partitioned into shards**, usually placed on different machines
- This lets you scale beyond what a single server could hold, and spreads out read/write load
- The **Distributed engine** stitches these shards back together so, from a user's perspective, it still just looks like one table

### Replication = copying the data
- Each shard can be **copied ("replicated") across multiple nodes**
- If one node dies, the data isn't lost — a replica still has it
- Every MergeTree engine has a matching **"Replicated" version** (e.g., `ReplicatedMergeTree`) that adds this safety net

### Coordination = keeping replicas in sync
- All of this replica-syncing needs a referee. That's **ClickHouse Keeper**
- Keeper is a Raft-consensus-based service, and it's a **drop-in replacement for Apache ZooKeeper**
- It guarantees things like "we always have exactly 2 healthy copies of this shard"

**Example setup from the training deck:** a table split into **2 shards**, each replicated to **2 nodes** = 4 physical copies of data spread across 4 nodes total, all coordinated by Keeper.

**Analogy:** Imagine a company's important documents. *Sharding* is like splitting the filing cabinet across two offices because one office ran out of space. *Replication* is like keeping a photocopy of every document in a second office too, in case one office floods. *Keeper* is the office manager making sure both offices always have matching, up-to-date copies.

---

## Layer 4: The Integration Layer (the "delivery service")

**Job:** Let ClickHouse talk to the outside data world without you needing to manually import/export everything.

This connects to external databases, message queues, and object storage — same idea as the "virtual engines" mentioned in storage, but thought of as its own architectural layer since it's about *connecting out*, not just storing.

---

## Parallelism: Why ClickHouse Feels So Fast

ClickHouse pushes parallel work at **three different zoom levels** simultaneously:

| Level | What's happening | Analogy |
|---|---|---|
| **SIMD** (Single Instruction, Multiple Data) | One CPU core processes *multiple* data elements at the same time, per instruction | One factory worker using a multi-head stamping tool instead of stamping one item at a time |
| **Multi-core** | The query plan is split into independent lanes run across worker threads on one machine | Multiple workers on the same factory floor, each handling their own task lane |
| **Multi-node** | Sharded tables let *multiple machines* scan/filter/aggregate at the same time | Multiple factories in different cities, all working on the same order simultaneously |

**End result:** ClickHouse uses *all* available hardware — you can scale **vertically** (add more cores/bigger machine) or **horizontally** (add more nodes) and it takes advantage of both.

---

## How You Can Actually Run ClickHouse (Deployment Options)

There isn't just one way to use ClickHouse. There are **four**:

1. **Standalone** — A single server, or a full multi-node cluster with sharding/replication. Talk to it via native, MySQL, PostgreSQL, or HTTP protocols. (This is the "classic" server deployment.)
2. **Cloud** — **ClickHouse Cloud**, a fully managed, autoscaling database-as-a-service. Someone else runs the servers for you.
3. **On-premise** — Same idea as standalone, but explicitly deployed on your own company's hardware/infrastructure.
4. **In-process (chDB)** — ClickHouse gets **embedded directly inside another program** (e.g., inside a Python + Jupyter + Pandas session) for zero-copy, interactive analysis. There's also a **command-line utility** for analyzing files directly — think of it as a SQL-powered alternative to classic Unix tools like `cat` and `grep`.

**Analogy:** It's like coffee. You can (1) go to a full café (standalone server), (2) subscribe to a coffee delivery service that handles everything (cloud), (3) install a coffee machine in your own office (on-premise), or (4) carry a tiny personal espresso pod maker in your bag for on-the-spot use (chDB/in-process).

---

## Why This Architecture Actually Matters (The "So What?")

Pulling it all together, here's *why* ClickHouse is fast, in four sentences:

1. **Write-optimized storage** decouples background merging (housekeeping) from parallel inserts (new data coming in), so writes don't get blocked by cleanup work.
2. **Sparse primary indexes, skipping indices, and projections** aggressively prune away data that's irrelevant to your query — so it barely has to look at data that won't matter for the answer.
3. **A vectorized, compiled query engine** parallelizes work across SIMD, cores, and nodes simultaneously (as we covered above).
4. **Integration engines + 90+ supported data formats** mean ClickHouse plugs into almost any existing data ecosystem without painful custom pipelines.

---

## Quick Recap Table

| Layer | Main Job | Key Concept to Remember |
|---|---|---|
| Access | Talk to clients | Multi-protocol (native/MySQL/Postgres/HTTP) |
| Query Processing | Run your queries fast | Vectorized execution + parallelism (SIMD/core/node) |
| Storage | Store data efficiently | MergeTree family, sharding, replication |
| Integration | Connect to the outside world | 90+ formats, external DBs, queues, object storage |
| Keeper | Keep replicas in sync | Raft consensus, ZooKeeper-compatible |

---

## Check Your Understanding

Try answering these before moving on (a trainer would ask these out loud):

1. What's the difference between **sharding** and **replication**?
2. Why does ClickHouse use **vectorized execution** instead of row-by-row processing?
3. Name the three "zoom levels" of parallelism.
4. What is **ClickHouse Keeper**, and what well-known tool does it replace?
5. Which deployment option would you use to analyze a CSV file locally in a Jupyter notebook?

*(Answers: 1. Sharding splits data across nodes for scale; replication copies data across nodes for fault tolerance. 2. It processes data in batches instead of one row at a time, which is far more CPU-efficient. 3. SIMD, multi-core, multi-node. 4. A Raft-based coordination service that's a drop-in replacement for Apache ZooKeeper. 5. chDB / in-process embedding.)*

---

*Source: Schulze et al., "ClickHouse — Lightning Fast Analytics for Everyone," VLDB 2024*
