---
description: "RDS, Aurora, DynamoDB, ElastiCache, DAX, RDS Proxy, and OpenSearch address different kinds of data pressure."
tags:
  - "aws"
  - "databases"
  - "comparison"
---

# AWS Databases: The Right Room for the Question

> RDS, Aurora, DynamoDB, ElastiCache, DAX, RDS Proxy, and OpenSearch address different kinds of data pressure.

## The Business Goal

The DynamoDB warehouse was working.

Its retrieval lanes were direct.

Its collisions had rules.

Its alarms were armed.

So when anyone had a data problem, the owner sent them there.

It was the company’s newest room.

Surely it was the best one.

---

## The Story

### Shreya brings the books to dispatch

Shreya arrived with three years of orders, payments, refunds, fees, and merchant records.

She placed them on the warehouse dispatch counter.

> “Connect every refund to its payment, every payment to its order, and every order to its merchant.”

The dispatcher asked for the partition key.

Shreya did not have one.

She had questions:

```text
Which merchants have delayed payouts?

Which refunds are missing a matching settlement?

How did fee revenue change by region and month?
```

The questions crossed relationships and changed as she investigated.

The warehouse could store the records.

That did not make it the best room for this work.

### Leo returns with the same question

At the next counter, Leo refreshed a product page.

Then he refreshed it again.

Ten thousand shoppers were asking for the same product description.

The authoritative database could answer every request.

But the answer had not changed.

The team was making the system repeat work merely because it could.

### A thousand clerks crowd one office

Then Noah deployed a serverless function.

One invocation opened a connection to the relational database.

Then one thousand invocations arrived.

Each tried to open another.

The database knew how to answer the SQL.

The doorway could not comfortably accept the connection storm.

At the fourth counter, a customer described a product without knowing its product number.

The warehouse had an address.

The customer had words.

Four problems had arrived:

- relational investigation
- repeated reads
- connection pressure
- search without a known key

Calling all three “database performance” did not make them the same problem.

---

## The Architect Reopens the Floor Plan

The owner pointed toward DynamoDB.

> “It scales. Send everything there.”

The architect pointed toward the cache.

> “It is fast. Why not store everything there?”

Then toward Aurora.

> “It supports transactions. Why not send every lookup there?”

Each statement contained a strength.

Each turned that strength into a universal answer.

The architect wrote one question above every doorway:

> **What kind of work is waiting outside?**

---

## The Accounting Office

### Amazon RDS

Shreya moved her connected business records into the accounting office.

Amazon RDS provides managed relational database engines.

It is a strong starting point when the application needs:

- SQL
- joins and relational structures
- constraints
- transactions
- familiar database engines and tooling

RDS manages infrastructure concerns such as provisioning, backups, patching, and recovery features according to the selected configuration.

The team still owns:

- schema design
- indexes and queries
- connection behavior
- permissions and network access
- engine settings within the managed model
- monitoring, cost, and workload performance

The office preserves and recombines relationships.

AWS manages the building.

The company still manages the books.

---

## The Cloud-Built Accounting Office

### Amazon Aurora

Aurora is a cloud-optimized relational database compatible with MySQL or PostgreSQL.

It belongs in the relational family.

Choose it because the workload needs relational behavior and benefits from Aurora’s managed storage, replication, availability, or scaling architecture—not because “Aurora” is automatically the premium answer to every database question.

Useful anchors include:

- SQL and transactions
- MySQL or PostgreSQL compatibility
- writer and reader instances
- replicas and failover
- managed distributed storage
- provisioned and serverless deployment options for supported configurations

Aurora changes how the relational office is built and operated.

It does not turn relational questions into key-value questions.

---

## The Door Coordinator

### Amazon RDS Proxy

Noah’s thousand serverless workers did not need a different data model.

They needed fewer simultaneous database connections.

RDS Proxy sits between compatible applications and supported RDS or Aurora databases, pooling and reusing connections.

```text
Many application workers
          |
          v
       RDS Proxy
          |
          v
managed pool of database connections
          |
          v
       RDS/Aurora
```

RDS Proxy can help absorb connection churn and improve application resilience around database failover.

It does not:

- remove SQL
- redesign a poor schema
- repair inefficient queries
- make a relational model unnecessary

The room was correct.

The doorway needed a coordinator.

---

## The Dispatch Warehouse

### Amazon DynamoDB

Maya’s operational inventory lookups stayed in the warehouse.

DynamoDB is a strong candidate when:

- important access patterns are known
- key-value or document access dominates
- horizontal scale and managed distribution matter
- predictable low-latency access matters
- joins are not central to the hot path
- deliberate denormalization is acceptable

DynamoDB was not rejected because it could not store Shreya’s records.

It was rejected for that specific job because her investigation depended on flexible relationships and broad questions.

The service choice followed the work.

---

## The Memory Desk

### Amazon ElastiCache

Leo’s product description was authoritative in a durable data store.

But customers requested it millions of times.

The company placed a temporary copy at a memory desk near the entrance.

```text
Application checks cache
       |
       +--> hit -> return copy
       |
       +--> miss
              |
              v
        authoritative store
              |
              v
          fill cache
```

Amazon ElastiCache provides managed in-memory caching through supported engines.

Common uses include:

- cache-aside
- sessions and temporary state
- repeated expensive results
- leaderboards and counters
- rate-limiting patterns

The application still needs to decide:

- when entries expire
- how entries are invalidated
- how stale a value may be
- what happens on a cache miss
- what happens when the cache is unavailable

A cache can be highly available.

That does not automatically make it the durable owner of the business truth.

---

## The DynamoDB Express Desk

### DynamoDB Accelerator

The warehouse had one especially repetitive, read-heavy lane.

The data already lived in DynamoDB.

The application could tolerate eventually consistent cached results.

DynamoDB Accelerator, or DAX, provides a DynamoDB-compatible in-memory cache for suitable read-heavy workloads.

DAX is a candidate when:

- DynamoDB is already the database
- reads repeat enough to benefit from caching
- microsecond response time is useful
- eventually consistent cached reads are acceptable

DAX does not help accelerate:

- relational queries
- strongly consistent reads, which DAX passes through to DynamoDB without caching
- non-DynamoDB data
- workloads with little cache reuse

ElastiCache is a general-purpose managed cache that applications address through the chosen cache engine.

DAX is specialized for DynamoDB access.

---

## The Extra Accounting Desk

### Read replicas

The owner confused a relational read replica with a cache.

Both could reduce pressure on the writer.

They did it differently.

```text
Read replica
SQL query -> replicated database instance

Cache
key lookup -> in-memory copy
```

A read replica still executes database queries against a replicated relational database.

A cache can avoid repeating the database query while the cached result remains usable.

Read replicas help scale relational reads and can support read-oriented workloads.

Caches help avoid repeated work and reduce latency.

Related problem.

Different room.

---

## The Research Wing

Shreya still wanted to scan and aggregate years of history.

Even a well-designed operational database can be the wrong place to run large analytical work beside customer transactions.

The company copied historical data into an analytical platform.

```text
Operational systems
small, frequent reads and writes
          |
          | controlled data movement
          v
Analytical platform
large scans, grouping, and aggregation
```

The operational systems served today’s business.

The research wing studied the past.

The copy introduced synchronization, security, lineage, freshness, and recovery responsibilities.

It also prevented a monthly investigation from standing in the checkout line.

---

## The Search Catalog

Leo knew the words:

```text
quiet wireless mouse with blue wheel
```

He did not know the product key.

OpenSearch maintained a purpose-built document index for:

- full-text search
- relevance
- exact filters
- aggregations

The catalog helped Leo discover the product.

The authoritative operational system still confirmed price and availability before checkout.

Search was not another exact lookup.

It was another kind of question.

---

## The Decision at the Door

| Work waiting outside | Likely starting point |
|---|---|
| SQL, joins, constraints, relational transactions | Amazon RDS or Amazon Aurora |
| Relational application with connection pressure | RDS Proxy in front of supported RDS/Aurora |
| Known key-based access at managed horizontal scale | DynamoDB |
| General-purpose in-memory caching | ElastiCache |
| Repeated eventually consistent DynamoDB reads needing very low latency | DAX |
| Scaling relational reads | Read replicas |
| Full-text discovery, relevance, filters, and search analytics | OpenSearch |
| Broad historical scans and aggregation | An analytical service |

This table is not an automatic selector.

Consistency, availability, latency, scale, team skills, security, recovery, regional design, and cost can change the final choice.

It is a way to begin with the work instead of the logo.

---

## The Wrong Way

The wrong way is to ask:

> “Which one is fastest?”

Fast at what?

A key lookup?
A join?
A repeated cached read?
A historical aggregation?
A connection storm?
A relevance-ranked text search?

Another wrong way is to solve every overloaded system with a cache.

A cache can hide repeated work.

It can also create stale answers, invalidation bugs, cold starts, and another dependency to operate.

The room should solve the actual pressure.

---

## Architectural Mapping

```text
                         +----------------------+
                         | Relational truth     |
                         | RDS / Aurora         |
                         +----------+-----------+
                                    |
                         connections coordinated
                                    |
                              +-----v-----+
                              | RDS Proxy |
                              +-----------+

     +----------------------+             +----------------------+
     | Key-based operations |             | Repeated reads       |
     | DynamoDB             |-----------> | DAX or ElastiCache   |
     +----------------------+             +----------------------+

                 operational changes copied deliberately
                                    |
                         +----------+----------+
                         |                     |
                         v                     v
              +------------------+   +------------------+
              | Search view      |   | Analytics        |
              | OpenSearch       |   | history          |
              +------------------+   +------------------+
```

This is a menu of roles, not a required architecture.

A system may use fewer services.

Every additional room must justify its boundary.

---

## Painkiller

> **Problem:** Several AWS data services appear to solve “database performance.”  
> **Pain:** Choosing by product reputation instead of workload contract puts relationships, key lookups, repeated reads, connection pressure, and text discovery behind the wrong door.  
> **AWS solution:** Match relational work to RDS or Aurora, key-based scale to DynamoDB, connection pressure to RDS Proxy, repeated reads to ElastiCache or DAX, and search-oriented access to OpenSearch.

---

## Knife Cut

> **A read replica is another database reader. A cache is memory that may avoid the database read.**

---

## The Headquarters Directory

### What Actually Just Happened

| In the story | AWS service or pattern | What it actually means |
|---|---|---|
| Accounting office | RDS or Aurora | Relational truth, SQL, joins, and transactions |
| Door coordinator | RDS Proxy | Managed pooling and reuse of relational connections |
| Dispatch warehouse | DynamoDB | Known key-based operational access |
| Memory desk | ElastiCache | General-purpose in-memory caching |
| DynamoDB express desk | DAX | DynamoDB-specific cache for suitable eventual reads |
| Extra accounting desk | Read replica | Relational database read scaling |
| Search catalog | OpenSearch | Full-text discovery, relevance, filters, and aggregations |
| Research wing | Analytical platform | Broad historical queries away from operational traffic |

The company did not choose the newest room.

It sent each kind of work to the room built to perform it.

---

## A Note From the Author

AWS data services overlap more than the headquarters metaphor suggests.

RDS and Aurora can serve many high-scale operational workloads. DynamoDB supports transactions and multiple consistency options. ElastiCache can hold data structures used for more than simple cache-aside. OpenSearch can store documents as well as search them. Service capabilities and supported configurations also evolve.

The architectural decision must use current service documentation and the workload’s actual requirements.

This chapter intentionally treats analytics as a role rather than selecting one product. The correct analytical service depends on the data shape, latency, query engine, governance, and operational model.

The diagram is illustrative. It is not a recommendation to place every service in one application.

Technical references:

- [Amazon RDS Proxy for Aurora](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/rds-proxy.html)
- [DAX and DynamoDB consistency models](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/DAX.consistency.html)
- [Amazon OpenSearch Service documentation](https://docs.aws.amazon.com/opensearch-service/)

---

## The Last Bite

Do not ask which database is fastest.

Ask:

> Fast at which job, under which promise, and at whose operational cost?

The right room begins with the question waiting outside.

---

**Next chapter:** *[AWS Databases: Choose the Home for the Truth](11-databases-epilogue.md)*

Every room is finally open.

The architect now walks one order through the entire headquarters to see where the truth lives, where its copies travel, and what happens when a door fails.
