---
description: "A DynamoDB table has a lifecycle: items expire, changes leave the table, capacity is consumed, backups restore into new tables, and distant replicas keep their own copy of the truth."
tags:
  - "aws"
  - "databases"
  - "dynamodb"
---

# Amazon DynamoDB: After Closing Time

> A DynamoDB table has a lifecycle: items expire, changes leave the table, capacity is consumed, backups restore into new tables, and distant replicas keep their own copy of the truth.

## The Business Goal

During the day, the warehouse answered requests.

After closing time, Noah walked the empty aisles.

The lights were still on.

Reservation cards had expired.

Change records waited for handlers.

Capacity meters kept turning.

A backup alarm blinked red.

And on a screen across the room, a second Region asked:

> “If this warehouse disappears, where does the application go?”

The day shift had taught the team how to use the table.

The night shift would teach them how the table lives.

---

## The Reservation That Expired but Did Not Leave

Leo reserved a pair of shoes for fifteen minutes.

The reservation item carried:

```text
ExpiresAt = 1785195000
```

The timestamp passed.

Maya opened the table.

The reservation was still there.

> “It expired,” she said. “Why has it not disappeared?”

The architect pointed to the cleanup cart.

> “Expiration makes it eligible for cleanup. It does not schedule an appointment.”

---

## Time to Live

DynamoDB Time to Live, or TTL, lets the application store a per-item expiration timestamp in a designated attribute.

The value must be:

- a DynamoDB Number
- a Unix epoch timestamp
- expressed in seconds

After the timestamp passes, DynamoDB deletes the item asynchronously, typically within a few days.

The deletion does not consume table write throughput in the Region where the TTL process performs the initial delete.

Until deletion actually occurs, an expired item:

- can still appear in reads
- continues to consume storage
- can still be updated
- may need to be excluded by application logic

```text
expiration timestamp passes
          |
          v
item becomes eligible
          |
          v
background TTL deletion
          |
          v
item removed from table and indexes
```

TTL is excellent for data with a natural lifetime:

- sessions
- temporary reservations
- idempotency records
- event deduplication markers
- short-lived caches
- retention-limited operational data

TTL is not an exact scheduler.

If the business requires an action at precisely 3:00 PM, the TTL deletion process is not the clock that should trigger it.

---

## The Cleanup Cart Rings the Bell

When DynamoDB deleted the reservation, the change appeared in DynamoDB Streams.

The stream record identified it as a service deletion in the Region where TTL performed the delete.

A Lambda function listened for those records and archived selected expired reservations.

```text
TTL deletes item
       |
       v
DynamoDB Streams record
       |
       v
Lambda archive handler
       |
       v
long-term destination
```

Then the handler failed halfway through a batch.

Lambda tried the batch again.

The archive code saw one record twice.

That was not a contradiction.

Lambda event source mappings for DynamoDB Streams process events at least once. Duplicate processing can occur.

The consumer needed idempotent behavior.

---

## The Change Ledger

DynamoDB Streams captures a time-ordered sequence of item-level changes for a table.

Depending on the configured stream view type, a record can contain:

- keys only
- the new image
- the old image
- both old and new images

Stream records remain available for 24 hours.

Lambda polls the stream through an event source mapping and delivers records in batches.

When a batch fails, the team can configure controls such as:

- maximum retry attempts
- maximum record age
- bisecting a failed batch
- partial batch responses
- a destination for discarded records
- event filtering

Increasing parallelization can improve throughput. Lambda still preserves in-order processing for records affecting the same item key during ordinary processing.

The application must not translate that into:

> “Every downstream side effect happens exactly once.”

The safe rule is:

> **The stream preserves the change history it promises. The consumer preserves the business outcome it needs.**

Noah monitored iterator age.

If it kept climbing toward the stream-retention window, the consumer was falling behind.

---

## The Meter on Every Package

At midnight, the owner studied the capacity bill.

> “Why did this six-kilobyte record cost more than the three-kilobyte record?”

DynamoDB capacity rounds items into units.

For provisioned capacity:

- one RCU supports one strongly consistent read per second for an item up to 4 KB
- one RCU supports two eventually consistent reads per second for items up to 4 KB
- one WCU supports one write per second for an item up to 1 KB
- transactional reads and writes consume twice the corresponding units

Read sizes round up to the next 4 KB boundary.

Write sizes round up to the next 1 KB boundary.

### A six-kilobyte read

```text
6 KB rounds to 8 KB

strong read:
8 KB / 4 KB = 2 RCUs

eventual read:
2 RCUs / 2 = 1 RCU

transactional read:
2 RCUs x 2 = 4 RCUs
```

Ten strongly consistent reads per second of that item require:

```text
10 x 2 = 20 RCUs
```

### A 1.6-kilobyte write

```text
1.6 KB rounds to 2 KB

standard write:
2 WCUs

transactional write:
4 WCUs
```

Names count toward item size.

Indexes add their own storage and write work.

Filters and projection expressions do not make DynamoDB avoid reading the evaluated items for capacity calculation.

The team could choose on-demand capacity instead of provisioning RCUs and WCUs in advance.

The unit sizes and request economics still mattered.

On-demand changed how capacity was managed and billed.

It did not make item size, transaction cost, or poor access patterns irrelevant.

---

## The Alarm Board

Noah did not want to discover pressure through customer complaints.

DynamoDB publishes metrics to Amazon CloudWatch.

He built alarms and dashboards around signals such as:

| Signal | What it can reveal |
|---|---|
| `ConsumedReadCapacityUnits` | Read demand |
| `ConsumedWriteCapacityUnits` | Write demand |
| `ThrottledRequests` and throttle-reason metrics | Capacity, quota, or hot-key pressure |
| `SuccessfulRequestLatency` | Service-side request latency |
| `SystemErrors` | Server-side failures |
| `UserErrors` | Invalid or rejected requests |
| `ConditionalCheckFailedRequests` | Failed business preconditions or contention |
| `TransactionConflict` | Transaction contention |
| `TimeToLiveDeletedItemCount` | TTL cleanup activity |
| Lambda errors and iterator age | Stream consumer health and lag |

A high conditional-failure count was not automatically an infrastructure outage.

It could mean the application was correctly refusing oversells.

Metrics needed interpretation in the context of the business event.

---

## The Ledger Is Damaged

A deployment bug overwrote product records.

The owner opened the live table and asked:

> “Can we rewind it?”

DynamoDB offers two main backup approaches.

### On-demand backup

An on-demand backup captures the table at a chosen point for retention or archival needs.

### Point-in-time recovery

Point-in-time recovery, or PITR, continuously protects a table after it is enabled.

The configurable recovery period can span from 1 to 35 days.

Within the available recovery window, the team can select a point in time at second-level granularity.

The restore creates a new table.

```text
damaged live table
        |
        v
choose recovery point
        |
        v
restore operation
        |
        v
new DynamoDB table
```

The live table does not roll backward in place.

The team still needs a recovery procedure:

- validate the restored data
- restore or recreate relevant settings
- redirect or migrate application traffic
- reconcile changes made after the selected recovery point
- verify IAM, alarms, integrations, tags, and downstream consumers

A backup is a recovery input.

It is not a recovery plan.

---

## The Lock Around Every Ledger

DynamoDB encrypts table data at rest.

The encryption also covers indexes, streams, and backups according to the table’s encryption configuration and service behavior.

The team can use supported key choices, including:

- an AWS owned key
- an AWS managed key
- a customer managed AWS KMS key

A customer managed key gives the team more control over key policy, lifecycle, and auditing.

It also gives the team more responsibility.

If the application requires protection before the data reaches DynamoDB, that is a separate client-side encryption decision.

Server-side encryption protects stored data.

It does not decide who is authorized to call `GetItem`.

IAM authorization, transport encryption, and data-at-rest encryption solve different parts of the security story.

---

## The Warehouse Across the Ocean

The company opened stores near customers in another AWS Region.

Sending every request across the ocean increased latency and made one Region a larger dependency.

DynamoDB global tables replicate table data across selected Regions and allow applications to use regional replicas.

```text
Application in Region A       Application in Region B
          |                             |
          v                             v
     Replica A <------ replication ----> Replica B
```

Global tables support more than one consistency model.

### Multi-Region eventual consistency

With multi-Region eventual consistency, or MREC, changes replicate asynchronously between Regions.

Applications must tolerate a window in which replicas differ.

Concurrent updates and conflict behavior must be part of the design.

### Multi-Region strong consistency

For supported global-table configurations and Regions, multi-Region strong consistency, or MRSC, coordinates writes across its required Regional topology and supports strongly consistent reads from replicas.

MRSC has different availability, topology, latency, and operational considerations from MREC.

The team should choose based on current documented support and business requirements—not merely because “strong” sounds safer.

Global tables do not turn a single-Region transaction into one cross-Region ACID transaction. Transaction guarantees apply in the Region where the transaction is performed; replication to other Regions follows the global table’s behavior.

Replication also consumes resources and produces cost.

TTL deletes replicated to other Regions consume replicated write units there.

The second warehouse provided regional access and resilience.

It did not eliminate distributed-systems trade-offs.

---

## The Wrong Way

The wrong way is to treat lifecycle features as switches:

```text
TTL enabled
Streams enabled
PITR enabled
Global table enabled
```

The switches create mechanisms.

The application still needs policies:

- which timestamp makes an item expired
- how reads treat expired-but-not-yet-deleted data
- how stream consumers handle duplicates and poison records
- which recovery point the business can accept
- how a restored table returns to service
- which consistency model the Regional application expects
- which metrics wake the team before customers do

Another wrong way is to use current limits as timeless architecture.

Quotas and service capabilities can change.

Verify volatile details against current documentation.

---

## Architectural Mapping

```text
live DynamoDB table
        |
        +--> TTL cleanup
        |         |
        |         v
        |      stream record
        |
        +--> DynamoDB Streams --> Lambda consumer
        |
        +--> CloudWatch metrics --> alarms
        |
        +--> on-demand backup / PITR --> restored new table
        |
        +--> global-table replication --> Regional replica
```

The live table is only the center.

Lifecycle, observation, recovery, and replication determine how the table behaves over time.

---

## When to Use Each Lifecycle Tool

| Need | Starting tool |
|---|---|
| Remove items after a natural lifetime | TTL |
| React to item-level changes | DynamoDB Streams |
| Process table changes with code | Lambda event source mapping |
| Retain a deliberate snapshot | On-demand backup |
| Recover from an accidental change within a rolling window | PITR |
| Serve and replicate table data across Regions | Global tables |
| Observe traffic, latency, errors, and throttling | CloudWatch metrics and alarms |
| Control the encryption key lifecycle | Customer managed KMS key where justified |

Each tool solves a different failure or lifecycle requirement.

---

## Painkiller

> **Problem:** A DynamoDB table continues changing after the application’s successful request returns.  
> **Pain:** Expired data lingers, stream handlers retry, capacity costs surprise the team, damage requires restoration, and distant replicas introduce consistency trade-offs.  
> **AWS solution:** Design TTL, Streams, capacity, monitoring, encryption, backups, recovery, and global replication as one explicit table lifecycle.

---

## Knife Cuts

> **TTL marks an item for asynchronous deletion. It does not schedule exact-time work.**

> **A backup preserves data. A recovery plan returns the application to service.**

> **A Regional replica brings the table closer. It also brings distributed consistency decisions.**

---

## The Night Shift Board

### What Actually Just Happened

| In the story | In DynamoDB | What it actually means |
|---|---|---|
| Expired reservation card | TTL attribute | Per-item epoch timestamp for asynchronous deletion |
| Cleanup cart | DynamoDB TTL process | Background service deletion |
| Change ledger | DynamoDB Streams | Retained item-level change records |
| Handler sees a record twice | At-least-once processing | Consumer code must be idempotent |
| Package meter | Read and write capacity units | Item size and consistency determine consumption |
| Alarm board | CloudWatch metrics | Operational visibility into demand and failure |
| Rebuilt ledger | Backup or PITR restore | Recovery creates a new table |
| Lock around the ledger | Encryption at rest | DynamoDB and KMS protect stored data |
| Distant warehouse | Global table replica | Multi-Region access and replication |

The warehouse did not stop operating after the doors closed.

It expired, copied, measured, protected, and repaired truth through the night.

---

## A Note From the Author

Lifecycle details are especially sensitive to service changes.

TTL deletion is asynchronous, and expired items can remain visible until the background process deletes them. Applications that cannot use expired data must enforce that rule themselves.

DynamoDB Streams retains records for 24 hours. Lambda event source mappings can retry and duplicate processing, so consumer idempotency and failure destinations matter.

PITR restores into a new table. Restored-table settings and application cutover need explicit verification.

Global tables now support both MREC and MRSC configurations. Availability, topology, consistency, latency, and limitations differ, so a production choice must use current Regional documentation.

Technical references:

- [Using TTL in DynamoDB](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/TTL.html)
- [Using Lambda with DynamoDB Streams](https://docs.aws.amazon.com/lambda/latest/dg/with-ddb.html)
- [DynamoDB read and write capacity consumption](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/read-write-operations.html)
- [Backup and restore for DynamoDB](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/Backup-and-Restore.html)
- [DynamoDB read consistency and global tables](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/HowItWorks.ReadConsistency.html)
- [Monitoring DynamoDB with CloudWatch](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/Monitoring-metrics-with-Amazon-CloudWatch.html)

---

## The Last Bite

The request returned.

The table kept living.

Items expired.

Changes traveled.

Capacity burned.

Backups waited.

And distant replicas negotiated what “current” meant.

Design the night shift before the lights go out.

---

**Next chapter:** *[Amazon RDS and Amazon Aurora: The Accounting Office](07-rds-aurora-accounting-office.md)*

The DynamoDB warehouse is now complete—from application code to recovery.

Then Shreya arrives with a refund connected to an order, payment, merchant, and settlement—and asks the warehouse to follow every relationship.
