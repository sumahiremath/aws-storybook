---
description: "ElastiCache keeps reusable data in memory so applications can answer repeated questions quickly without making the durable database repeat the work."
tags:
  - "aws"
  - "databases"
  - "elasticache"
---

# Amazon ElastiCache: The Memory Desk

> ElastiCache keeps reusable data in memory so applications can answer repeated questions quickly without making the durable database repeat the work.

## The Business Goal

The accounting office was healthy.

The warehouse was healthy.

The product page was slow.

Leo refreshed it.

Then ten thousand customers refreshed it.

Every request asked:

> “What is the name, price, description, and image for P100?”

The answer changed once that morning.

The databases rebuilt it ten thousand times.

The systems were not failing.

They were repeating themselves.

---

## The Story

### The receptionist remembers

The architect placed a small desk near the front entrance.

The next customer asked for P100.

The receptionist did not know the answer.

She walked to the authoritative office, retrieved it, and wrote a temporary card:

```text
PRODUCT#P100

Name        Wireless Mouse
Price       29.95
Description Ergonomic wireless mouse
ExpiresAt   11:15
```

The next customer asked the same question.

The receptionist returned the card.

No warehouse walk.

No SQL query.

No reconstruction.

The question crossed the building once.

Then memory answered.

That is the central idea of a cache.

> **Do not make the durable system repeat work whose answer is still reusable.**

---

## Meet Amazon ElastiCache

Amazon ElastiCache is a managed in-memory data service supporting Valkey, Redis OSS, and Memcached.

It offers serverless and node-based deployment options.

ElastiCache manages infrastructure concerns according to the selected engine and deployment, including:

- cache compute and memory
- node replacement
- software maintenance
- monitoring integrations
- replication and failover features where supported
- scaling mechanisms

The application team still owns:

- what is cached
- cache keys
- TTL values
- invalidation
- freshness rules
- miss behavior
- serialization
- failure behavior
- memory and eviction policy choices
- security and network access
- cost

AWS can manage the memory desk.

It cannot decide when the card has become a lie.

---

## Cache-Aside: Check Memory First

The receptionist followed the cache-aside pattern.

```text
request
   |
   v
check cache
   |
   +--> hit ------> return cached value
   |
   +--> miss
          |
          v
    read durable store
          |
          v
      populate cache
          |
          v
      return value
```

The application—not the database—coordinates this pattern.

On a cache hit, the application avoids the durable read.

On a cache miss, it retrieves the value from the source and places a reusable copy in the cache.

Cache-aside works well when:

- reads repeat
- a temporary copy is acceptable
- misses can fall back to the source
- the application can define freshness

It introduces two systems that can disagree.

That disagreement must have a policy.

---

## The Price Changes

Maya changed the product price from `29.95` to `24.95`.

The database committed the new value.

The receptionist’s card still said `29.95`.

The team had several choices.

### Let the card expire

Assign a short TTL and accept bounded staleness.

Simple.

Potentially stale until expiration.

### Remove the card after the write

Update the authoritative store, then invalidate the cache key.

The next read misses and repopulates the new value.

There is still a failure window between the database write and invalidation.

### Update the card after the write

Write the source of truth, then refresh the cache.

This can reduce the next miss but still requires failure handling.

### Write through the cache path

For designs that support it, the application writes through a caching layer that coordinates the downstream update.

This adds coupling and must preserve the durable write’s correctness.

No pattern removes the trade-off.

> **Cache invalidation is deciding how long a fast answer is allowed to be wrong.**

---

## Expiration Is Not Eviction

The owner saw a card disappear before its TTL.

> “It was not expired.”

The cache had run low on memory and evicted an entry according to its engine and configured policy.

Expiration and eviction are different:

| Event | Meaning |
|---|---|
| Expiration | The item’s TTL says it should no longer be served |
| Eviction | The cache removes an item to manage memory pressure |
| Invalidation | The application deliberately removes or replaces an item |

Applications must tolerate a miss at any time.

A cache is an optimization.

If a missing cache entry makes correctness impossible, the cache may have quietly become a primary data dependency.

That decision deserves explicit durability and availability analysis.

---

## The Crowd at the Empty Desk

The most popular card expired.

Ten thousand requests missed at once.

All ten thousand walked toward the database.

The cache had protected the source all morning.

Its expiration created a stampede.

Possible controls include:

- adding randomness to TTL values
- allowing one worker to refresh while others wait
- serving a briefly stale value during refresh
- refreshing popular entries before expiration
- limiting concurrent work against the source
- applying backoff after source failures

The application should also use a fallback strategy when the cache itself is unavailable.

Blindly sending the full cached load to the database can turn a cache failure into a database failure.

---

## The Session Coat Check

Leo logged in through one application server.

His next request reached another.

The application needed shared session state.

The memory desk could store:

```text
SESSION#abc123
    |
    +--> user ID
    +--> preferences
    +--> short-lived workflow state
    +--> expiration
```

ElastiCache is commonly used for sessions because access is fast and expiration is natural.

The team still had to decide:

- what happens if the session entry disappears
- whether losing the session merely requires login again
- whether any session field is too important to exist only in memory
- how session IDs are protected
- how logout invalidates the session

A shopping preference and a completed payment do not deserve the same durability contract.

---

## Choosing the Engine

ElastiCache supports several engines.

### Valkey or Redis OSS

Valkey and Redis OSS support rich in-memory data structures and commands.

They can serve patterns such as:

- strings and cached objects
- hashes
- lists and sets
- counters
- leaderboards with sorted sets
- rate limiting
- pub/sub and stream-oriented patterns where appropriate

For node-based designs, replication groups can include a writable primary and read replicas. Multi-AZ with automatic failover can promote a replica when the primary fails.

Replication is normally asynchronous, so the durability behavior of acknowledged data must match the selected configuration.

Cluster mode can partition data across shards for greater scale.

### Memcached

Memcached provides a comparatively simple distributed memory cache.

It is useful for straightforward object caching and horizontal distribution.

It does not provide the same replication, persistence, failover, or rich data-structure model as Valkey or Redis OSS.

Choose the engine by the application behavior required.

Do not choose Redis-compatible features for a workload that needs only a simple disposable cache.

Do not choose Memcached when the design depends on replication or data structures it does not provide.

---

## Primary, Replicas, and Shards

For Valkey or Redis OSS replication groups:

```text
application writes
       |
       v
primary node
       |
       | asynchronous replication
       +----------------+
       |                |
       v                v
read replica A     read replica B
```

Replicas can spread reads and support failover.

Shards divide the keyspace:

```text
cache keys
   |
   +--> shard 1
   +--> shard 2
   +--> shard 3
```

Replication and sharding answer different questions:

- replicas create additional copies within a shard
- shards distribute different keys across primaries

Client compatibility, endpoint selection, failure behavior, and resharding all matter.

ElastiCache Serverless can remove much of the explicit node and shard planning for supported workloads, but the application still owns key design, data lifetime, and failure behavior.

---

## ElastiCache or DAX?

The owner pointed at the DynamoDB warehouse.

> “Did we not already build a memory desk called DAX?”

DynamoDB Accelerator is a DynamoDB-specific caching service.

ElastiCache is a general-purpose managed in-memory service.

| Question | ElastiCache | DAX |
|---|---|---|
| What can it cache? | Application-selected data from many sources | DynamoDB items and query results |
| Which client model? | Valkey, Redis OSS, or Memcached client | DynamoDB-compatible DAX client |
| Who designs cache keys and invalidation? | Primarily the application | DAX manages its DynamoDB-oriented cache behavior |
| Can it accelerate strongly consistent DynamoDB reads? | Application-defined design | DAX passes them through without caching |
| Typical use | General caching, sessions, counters, data structures | Repeated eventually consistent DynamoDB reads |

If the application already uses DynamoDB and wants a compatible cache for suitable repeated eventual reads, DAX may fit.

If the application needs a general memory service or richer data structures, ElastiCache may fit.

---

## The Memory Desk Needs Walls

An ElastiCache deployment belongs behind controlled network access.

Depending on the engine and configuration, the team should consider:

- VPC and security-group paths
- TLS in transit
- encryption at rest where supported
- engine authentication and authorization
- secrets or tokens
- restricted administrative access

Do not expose a cache as if speed excuses access control.

Cached data can contain the same sensitive information as its source.

---

## Watch What Memory Forgets

Noah monitored:

- cache hits and misses
- evictions
- free or used memory
- CPU
- current connections
- replication lag
- engine-specific errors
- latency

A low hit rate could mean:

- keys are poorly chosen
- TTLs are too short
- the workload does not repeat
- the cache is too small
- the application is bypassing the cache

More cache is not always the answer.

Measure whether the cache avoids meaningful work.

---

## The Wrong Way

The wrong way is:

> “Put a cache in front of it.”

Before adding a cache, ask:

- Which repeated work disappears?
- How stale may the answer be?
- What happens on a miss?
- What happens during a stampede?
- What happens if the cache is unavailable?
- Which system owns the durable fact?

Another wrong way is to treat every cache entry as recoverable while storing the only copy of important business state there.

Memory is fast.

That does not make every memory configuration durable.

---

## Architectural Mapping

```text
application request
        |
        v
     ElastiCache
        |
        +--> hit -----> response
        |
        +--> miss
               |
               v
       authoritative store
               |
               v
          refresh cache
```

The cache shortens a path.

The authoritative store preserves the fact.

The application defines how those roles reconnect after change or failure.

---

## When to Use ElastiCache

Use ElastiCache when:

- reads or computations repeat
- lower latency materially helps
- the application can define freshness
- sessions or temporary state need shared low-latency access
- in-memory data structures fit the workload
- the source needs protection from repeated demand

Consider another approach when:

- the workload has little cache reuse
- every read must reflect the latest durable write
- the cache would become an unjustified second system
- the application cannot tolerate eviction or failover behavior

---

## Painkiller

> **Problem:** Applications repeatedly ask durable systems to rebuild answers that have not changed.  
> **Pain:** Database load, latency, and cost rise even though most work is repetition.  
> **AWS solution:** ElastiCache keeps reusable data in managed memory while the application defines misses, expiration, invalidation, eviction, and failure behavior.

---

## Knife Cuts

> **Expiration follows time. Eviction follows memory pressure. Invalidation follows application intent.**

> **A cache hit avoids work. A cache miss must still be a safe application path.**

> **DAX remembers DynamoDB. ElastiCache remembers whatever the application deliberately places there.**

---

## The Memory Desk

### What Actually Just Happened

| In the story | In caching | What it actually means |
|---|---|---|
| Temporary product card | Cache entry | Reusable in-memory copy |
| Receptionist remembers | Cache hit | Return without reading the source |
| Receptionist walks to the office | Cache miss | Read source and populate cache |
| Card expiration time | TTL | Bounded lifetime |
| Card removed early | Eviction | Memory-pressure removal |
| Price card removed after change | Invalidation | Application-driven freshness |
| Crowd after expiration | Cache stampede | Many misses overload the source |
| Session coat check | Shared session store | Temporary state available across workers |

The memory desk was valuable because everyone knew it could forget.

---

## A Note From the Author

ElastiCache capabilities vary by engine, deployment option, and version. Valkey, Redis OSS, and Memcached do not offer identical replication, persistence, failover, scaling, security, or data-structure behavior.

The story treats a cache as disposable because that is the safest default mental model. Some ElastiCache configurations offer stronger durability and recovery features, but the application must deliberately choose and validate those guarantees.

Multi-AZ failover and asynchronous replication can involve interruption or data loss depending on the engine and configuration. Do not infer durable exactly-once state from the existence of replicas.

Technical references:

- [How ElastiCache works](https://docs.aws.amazon.com/AmazonElastiCache/latest/dg/WhatIs.corecomponents.html)
- [ElastiCache components and replication](https://docs.aws.amazon.com/AmazonElastiCache/latest/dg/WhatIs.Components.html)
- [High availability using replication groups](https://docs.aws.amazon.com/AmazonElastiCache/latest/dg/Replication.html)
- [ElastiCache Multi-AZ and automatic failover](https://docs.aws.amazon.com/AmazonElastiCache/latest/dg/AutoFailover.html)

---

## The Last Bite

The database remembers the truth.

The cache remembers the question everyone keeps asking.

Speed comes from knowing which memory is allowed to forget.

---

**Next chapter:** *[Amazon OpenSearch Service: The Search Catalog](09-opensearch-search-catalog.md)*

The memory desk can return P100 instantly when the application knows the key.

Then a customer asks for “the quiet mouse with the blue wheel,” and nobody knows which product number to request.
