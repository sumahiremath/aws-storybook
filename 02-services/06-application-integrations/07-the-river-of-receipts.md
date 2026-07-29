---
description: "Preserve a continuous, partitioned stream of records so multiple consumers can process the same activity independently."
tags:
  - "aws"
  - "application-integration"
  - "event-driven"
  - "kinesis"
---

# Amazon Kinesis Data Streams: The River of Receipts

> Preserve a continuous, partitioned stream of records so multiple consumers can process the same activity independently.

## The Business Goal

Nia wanted one operations display showing:

- Orders per minute
- Payment approvals and declines
- Kitchen preparation time
- Drive-through congestion
- Delivery scans

Fraud wanted the payment activity.

Forecasting wanted order history.

Operations wanted current traffic.

Sending each record to one work queue created a new problem: when one consumer took a message, the other consumers did not receive their own position in the same history.

This was not a pile of independent jobs.

It was a continuing flow.

---

## The Story

Nia installed a receipt conveyor.

Every operational record entered a lane according to a routing value such as store ID.

Receipts remained on the conveyor for a retention window. The live dashboard, fraud desk, and forecasting team each followed the same lanes with their own reading position.

One reader falling behind did not make another reader surrender its copy.

But when every receipt used partition key `HEADQUARTERS`, one lane overflowed while the others remained quiet.

The stream could scale only as well as its partitioning.

---

## The Wrong Way

Kinesis is not the dashboard. It is the retained stream feeding dashboard and processing consumers.

It is also not an SQS replacement for every asynchronous task. Queue consumers compete for messages; stream consumers can independently read the same retained records.

A random partition key spreads traffic but destroys per-entity ordering. A constant partition key preserves one sequence by forcing all traffic through one hot path.

---

## Meet the AWS Service

> **Core idea:** Amazon Kinesis Data Streams stores records in shards, maps each record by partition key, and lets multiple consumers process the retained stream.

Producers put records. Consumers read records and track progress.

AWS manages the stream infrastructure. You manage capacity mode, partition-key distribution, producers, consumers, retention, checkpoints, error handling, permissions, encryption, scaling, and cost.

---

## How It Works

### The Complete Conveyor

#### Data Stream

A stream is the named collection of records and shards. Records remain available for the retention period even after consumers read them.

Default retention is 24 hours and can be increased up to 365 days. Retention allows consumers to recover or replay within that window; it is not permanent archival.

### Conveyor Lanes

#### Shards

A shard is a unit of stream capacity and ordering.

Provisioned mode exposes shard management. On-demand mode manages shards according to its capacity behavior. In either mode, traffic distribution still depends on partition keys.

### Choosing a Lane

#### Partition Key

Every record includes a partition key. Kinesis hashes the key to determine a shard.

Records sharing a partition key follow the same shard path and receive sequence numbers. A high-volume key can create a hot shard and throttle writes even while other capacity is idle.

Choose keys that preserve the order the business needs while distributing traffic.

### The Receipt

#### Data Record

A record includes the data blob, partition key, and sequence number assigned after ingestion.

Applications should include event identifiers, timestamps, schema information, and business identifiers in the record data when consumers need them.

### Independent Readers

#### Consumers

Consumers can use the Kinesis Client Library, APIs, Lambda event-source mappings, or supported integrations.

The Kinesis Client Library coordinates workers, shard leases, and checkpoints. A checkpoint records consumer progress; it does not delete stream records.

### A Dedicated Reading Window

#### Enhanced Fan-Out

Shared-throughput consumers share shard read throughput and poll for records.

Enhanced fan-out provides registered consumers dedicated read throughput per shard and pushes records over a subscription connection, reducing contention among consumers at additional cost.

### Lambda at the Conveyor

#### Event Source Mapping

Lambda polls Kinesis shards and invokes functions with batches. Processing for each shard respects the stream position.

A failed batch can block progress while it retries. Configuration can limit record age and retry attempts, bisect failed batches, report partial batch failures, and send exhausted failure metadata to supported destinations.

Lambda event-source mappings process records at least once, so functions should be idempotent.

---

## Architectural Mapping

```text
orders / payments / scans
          |
       producers
          v
Kinesis stream: shard A | shard B | shard C
       |             |             |
 dashboard        fraud         forecasting
 consumer         consumer       consumer
```

Metrics such as write throttling, read throttling, incoming records, and consumer iterator age reveal hot shards and lag.

---

## When to Use It

Use Kinesis Data Streams when:

- Records form a continuous high-volume stream
- Multiple consumers need the same retained activity
- Ordering is required within a partitioning boundary
- Consumers need replay within the retention window
- Near-real-time analytics or stream processing fits

## When Not to Use It

Use SQS when individual work messages should be buffered for competing consumers and removed after successful processing. Use EventBridge when discrete facts need pattern-based routing to service targets.

---

## Painkiller

> **Problem:** Several teams need the same continuing flow of operational activity.  
> **Pain:** A work queue divides messages among consumers and direct fan-out creates one delivery path per reader.  
> **AWS solution:** Kinesis retains partitioned records so independent consumers can follow and replay the same stream.

---

## Knife Cut

> **A queue distributes jobs among workers. A stream preserves activity for independent readers.**

---

## The Masthead

### What Actually Just Happened

|In the story|In Kinesis|What it actually means|
|---|---|---|
|Receipt conveyor|Data stream|Named retained record stream|
|Conveyor lane|Shard|Capacity and ordering unit|
|Lane selector|Partition key|Value hashed to choose a shard|
|Receipt number|Sequence number|Position information assigned in a shard|
|Reader's bookmark|Checkpoint|Consumer progress|
|Dedicated window|Enhanced fan-out|Dedicated per-consumer shard throughput|

---

## A Note From the Author

A physical conveyor implies every observer sees every receipt at the same instant. Consumers have independent lag, failure, checkpoint, and retry behavior.

Ordering is scoped to the stream's shard and partitioning behavior, not a magical global business timeline. Producer retries and multi-record APIs also require careful handling when strict application order matters.

- [Kinesis Data Streams concepts](https://docs.aws.amazon.com/streams/latest/dev/key-concepts.html)
- [Enhanced fan-out](https://docs.aws.amazon.com/streams/latest/dev/enhanced-consumers.html)

---

## The Last Bite

The dashboard was never the river.

It was one observer standing beside it.

> **Keep the activity flowing so every consumer can read the history it needs.**

---

**Next chapter:** *[AWS AppSync: The Order in Your Pocket](08-the-order-in-your-pocket.md)*

Management can now see Byte Burger's activity. The customer still needs a menu, a way to place an order, and a live answer to one question: “Is mine ready?”

