---
description: "A DynamoDB design is complete only when it explains collisions, retries, partial results, concentrated traffic, and failure."
tags:
  - "aws"
  - "databases"
  - "dynamodb"
---

# Amazon DynamoDB: The Warehouse Under Pressure

> A DynamoDB design is complete only when it explains collisions, retries, partial results, concentrated traffic, and failure.

## The Business Goal

The warehouse passed every rehearsal.

One request entered.

One worker followed the lane.

One correct answer came back.

Then launch day arrived.

The happy path lasted eleven seconds.

---

## The Story

### Two buyers, one pair of shoes

Leo reached checkout.

At another store, Akhila reached checkout too.

Both wanted the final pair of shoes.

The inventory card said:

```text
Quantity = 1
```

Leo’s clerk read the card.

Akhila’s clerk read the same card.

Both saw one.

Both wrote zero.

```text
Leo reads 1
Akhila reads 1
Leo writes 0
Akhila writes 0
```

The number on the card looked reasonable.

Two customers had been promised one pair.

The problem was not the final value.

The problem was that the decision had been split into an unsafe read and write.

### The gatekeeper checks the condition

The architect replaced the two-step procedure with one guarded instruction:

```text
Decrease Quantity by 1
only if Quantity > 0
```

Now the update and its condition were evaluated together.

One checkout changed one to zero.

The other found that the condition no longer held.

One succeeded.

One failed honestly.

> **Correctness is not what the warehouse says when requests arrive alone. It is what remains true when they collide.**

---

## Conditional Writes

DynamoDB condition expressions allow a write to succeed only when an item still satisfies an expected state.

They can protect:

- existence or nonexistence
- value comparisons
- expected versions
- legal state transitions
- idempotency markers

For the final pair:

```text
Update Quantity = Quantity - 1
Condition Quantity > 0
```

The losing request receives a conditional-check failure instead of silently violating inventory.

The application must treat that failure as a business outcome:

> “The item just sold out.”

It should not blindly retry until the condition changes.

---

## The Stale Clipboard

Two warehouse supervisors opened the same inventory record:

```text
Version = 7
```

Noah corrected the reorder level.

Another supervisor corrected the product status.

If both saved without checking, the second save could overwrite the first worker’s newer state.

The team required each update to present the version it had read:

```text
Write only if Version = 7
Then set Version = 8
```

```text
Worker A expects 7 -> succeeds -> version 8
Worker B expects 7 -> finds 8 -> fails
```

This is optimistic locking.

It assumes collisions are possible and detects them instead of holding a lock across the worker’s entire task.

The losing application can reload, reconcile, or ask a person to decide.

---

## One Order, Several Promises

Leo’s checkout needed more than an inventory decrement.

The business wanted to:

1. reserve the item
2. create the order
3. record the payment authorization
4. mark the request as processed

If only the first two succeeded, the company could reserve stock without a usable order.

These changes represented one business promise.

A DynamoDB transaction can coordinate multiple item actions as one all-or-nothing operation.

```text
TransactWriteItems
        |
        +--> inventory condition and update
        +--> order creation
        +--> reservation creation
        +--> idempotency marker
        |
        +--> all commit
        |
        +--> all fail
```

Transactions provide ACID behavior for supported actions.

They also perform additional coordinated work and consume more read or write capacity than comparable nontransactional operations.

Use them to protect business invariants.

Do not use them merely to reconstruct an unbounded relational graph.

---

## The Reply That Never Arrived

Leo pressed **Place order**.

The transaction committed.

Then the network dropped the reply.

Leo’s screen still showed a spinner.

The application tried again.

Without an idempotency design, the second request could create another order for the same intent.

The team gave the checkout request a stable token:

```text
IDEMPOTENCY#request-123
```

The first operation recorded the business result:

```text
Status  = COMPLETED
OrderId = O900
```

A repeated request with the same identity could return the original result instead of performing the business action twice.

> **Retry answers “Can I try again?” Idempotency answers “Will trying again duplicate success?”**

DynamoDB transactional writes can also accept a client request token for idempotent transaction requests within the service’s documented token behavior. Applications still need a business-level strategy when deduplication must outlive that window or cross service boundaries.

---

## The Crowd Returns Together

The promotion overwhelmed one request lane.

Some requests were throttled.

Every client immediately tried again.

The recovery wave became larger than the original wave.

Noah changed the retry policy:

```text
attempt 1 -> small randomized delay
attempt 2 -> larger randomized delay
attempt 3 -> larger randomized delay
```

Exponential backoff increased the delay.

Jitter prevented all clients from returning at the same moment.

AWS SDKs handle many retries automatically, but the application still needs:

- an idempotency strategy
- limits on retry duration and attempts
- awareness of which errors are retryable
- metrics that reveal sustained pressure

A retry is part of the load.

It is not an escape from the load.

---

## The Celebrity Product

Most products received ordinary traffic.

One influencer posted about P100.

Every request carried the same partition-key value:

```text
PRODUCT#P100
```

The table had many keys.

One key received most of the heat.

This is the danger of a hot key.

DynamoDB’s adaptive capacity can help uneven access patterns within service limits.

It cannot make one infinitely hot item infinitely scalable.

Possible responses depend on the access pattern:

- cache repeated reads
- redesign the grouping
- shard a write-heavy aggregate
- split an oversized tenant or time range
- reduce unnecessary requests
- precompute a view

Sharding a counter might produce:

```text
COUNTER#00
COUNTER#01
COUNTER#02
...
COUNTER#09
```

Writes spread.

Readers now need to fetch and sum several values.

The warehouse moved work.

It did not make work disappear.

---

## On-Demand Is Not a Cure for One Bad Route

The owner proposed switching capacity modes.

> “If we choose on-demand, the hot key goes away.”

Noah shook his head.

On-demand capacity is useful for workloads with unpredictable traffic because the team does not provision read and write capacity in advance.

Provisioned capacity can be useful when traffic is understood and the team wants explicit capacity settings and autoscaling.

Both modes still depend on sound key design.

Capacity mode changes how table throughput is managed and purchased.

It does not change the fact that concentrated traffic can overwhelm one route.

---

## The Cart With Missing Boxes

A warehouse worker requested one hundred known items in a batch.

Most arrived.

Some came back as unprocessed.

The worker looked at the full cart and assumed the job was complete.

Batch APIs improve transport efficiency by handling several known items in fewer requests.

They do not mean:

- every item necessarily succeeds in the first response
- the writes form one all-or-nothing transaction
- unprocessed items can be ignored

Applications must inspect and retry unprocessed items with appropriate backoff.

> **Batch reduces trips. Transaction coordinates correctness.**

---

## The Ledger With Another Page

Maya queried the customer order corridor.

The response contained several orders and a `LastEvaluatedKey`.

She displayed the orders and ignored the key.

Leo called.

> “Where is the rest of my history?”

DynamoDB `Query` and `Scan` results are paginated.

The application continues from `LastEvaluatedKey` using `ExclusiveStartKey`.

Ignoring pagination does not produce an obvious database error.

It produces a plausible, incomplete answer.

---

## The Change Bell Rings Twice

The warehouse attached a bell to the inventory ledger.

Every item change produced a stream record.

A Lambda consumer updated a replenishment system.

Then one invocation failed.

The record was retried.

If the consumer treated every delivery as unique, the same business action could happen twice.

Stream consumers should be designed for retry and duplicate processing. They also need a plan for records that repeatedly fail:

- retry controls
- batch isolation or partial-batch handling
- record-age limits where appropriate
- failure destinations or recovery workflows where supported
- monitoring of iterator age and errors

The stream says:

> “A change happened.”

The consumer must decide how to process that news safely.

---

## Architectural Mapping

```text
Request
   |
   +--> condition still true? ------ no ----> business failure
   |             |
   |            yes
   |             v
   +--> one write or transaction commits
                 |
                 +--> response may be lost
                 |
                 +--> retry uses same identity
                 |
                 +--> stream consumer processes idempotently
```

Correctness crosses the entire path.

It is not a setting applied only to the table.

---

## When to Use Each Protection

| Pressure | Protection |
|---|---|
| One item must still satisfy a rule | Conditional write |
| A stale writer must not overwrite newer state | Optimistic locking |
| Several item actions form one invariant | Transaction |
| A lost reply may trigger the same request | Idempotency |
| Temporary pressure causes throttling | Backoff and jitter |
| One key receives disproportionate traffic | Key redesign, sharding, caching, or reduced demand |
| Many known items travel together | Batch API with unprocessed-item handling |
| Results extend beyond one response | Pagination |
| Change processing may retry | Idempotent stream consumer and failure handling |

---

## Painkiller

> **Problem:** Concurrent writes, retries, throttling, and concentrated traffic violate the assumptions of a quiet test.  
> **Pain:** The application can oversell inventory, duplicate orders, return incomplete results, or amplify an outage with retries.  
> **AWS solution:** Combine conditional writes, transactions, idempotency, distributed keys, controlled retries, pagination, and monitoring according to the failure being prevented.

---

## Knife Cuts

> **Batch improves transport efficiency. Transaction protects coordinated correctness.**

> **Retry handles temporary failure. Idempotency prevents retry from duplicating success.**

> **More table capacity does not automatically repair one badly concentrated key.**

---

## The Warehouse Alarm Board

### What Actually Just Happened

| In the story | In DynamoDB | What it actually means |
|---|---|---|
| Two buyers, one pair | Conditional update | One write succeeds only while quantity remains |
| Stale clipboard | Optimistic locking | Version condition detects an outdated writer |
| One order, several promises | Transaction | Supported actions commit or fail as a unit |
| Lost checkout reply | Idempotency | A retry returns the same business outcome |
| Crowd returning together | Backoff and jitter | Retries spread out instead of forming another spike |
| Celebrity product | Hot key | One partition-key value receives disproportionate traffic |
| Cart with missing boxes | Unprocessed batch items | Partial batch results require inspection and retry |
| Ledger with another page | Pagination | The caller continues from `LastEvaluatedKey` |
| Change bell | DynamoDB Streams | Consumers process change records and tolerate retries |

The warehouse did not survive because nothing failed.

It survived because failure had a procedure.

---

## A Note From the Author

The story compresses several distinct failure modes into one launch day.

Production designs need precise handling for service errors, transaction cancellation reasons, conditional failures, client timeouts, retry budgets, stream batch behavior, capacity limits, and observability.

Atomic numeric updates are useful, but an unconditional increment or decrement is not automatically idempotent. A repeated request can apply it again. Conditions, stable request identities, or transactions may be needed depending on the invariant.

Adaptive capacity helps many uneven access patterns, but applications must still design for sustained concentration and documented partition limits.

Security, backups, point-in-time recovery, multi-Region behavior, and cost also remain part of production readiness.

Technical references:

- [Managing complex workflows with DynamoDB transactions](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/transactions.html)
- [Working with DynamoDB items and conditional writes](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/WorkingWithItems.html)
- [DynamoDB burst and adaptive capacity](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/burst-adaptive-capacity.html)

---

## The Last Bite

A table is not production-ready because `GetItem` works.

It is ready when the design can answer:

> “What happens when two requests arrive together, one reply disappears, and the hottest key catches fire?”

---

**Next chapter:** *[Amazon DynamoDB: The Developer’s Workbench](05-dynamodb-developers-workbench.md)*

The warehouse survived the promotion.

Now Akhila must translate application objects into precise requests—and decide what her code should do when DynamoDB returns only part of the work.
