---
description: "Design visibility, retries, ordering, dead-letter handling, and batching around work that can fail or be delivered again."
tags:
  - "aws"
  - "application-integration"
  - "event-driven"
  - "sqs"
---

# Amazon SQS: The Ticket That Came Back

> Design visibility, retries, ordering, dead-letter handling, and batching around work that can fail or be delivered again.

## The Business Goal

Marco pulled order 42 from the rail.

Halfway through cooking, the fryer failed.

The ticket returned. A second cook started it. Then Marco's fryer recovered, and he finished the same order.

Two meals reached the pickup counter.

Across the kitchen, order 91 had returned six times because its product code no longer existed. Every retry failed in the same way and blocked attention from healthy work.

The queue had preserved both tickets.

The application still needed to survive what preservation meant.

---

## The Story

Nia added rules to the rail.

When a cook claimed a ticket, a cover hid it for a limited time. A long order could extend the cover. A vanished cook let the cover expire so another cook could try.

Tickets that repeatedly failed moved to a problem-order desk.

Orders whose internal steps required sequence entered labeled lanes. Work for order 42 stayed ordered within group `42`, while order 43 could proceed in parallel.

And before serving anything, the kitchen checked whether that fulfillment had already completed.

---

## The Wrong Way

Visibility timeout is not a transaction lock, and it does not guarantee duplicate delivery cannot occur.

A dead-letter queue is not automatic problem resolution. It isolates messages after a configured receive count; operators still need alarms, diagnosis, redrive criteria, and a fix.

FIFO deduplication prevents certain duplicate sends into the queue. It does not make every downstream side effect exactly once.

---

## Meet the AWS Service

> **Core idea:** SQS reliability comes from making failed work visible again, isolating repeated failures, and constraining order only where the business requires it.

Standard and FIFO queues share core queue behavior but make different ordering and deduplication trade-offs.

---

## How It Works

### The Temporary Ticket Cover

#### Visibility Timeout

After receipt, a message remains in the queue but is temporarily hidden from other consumers. The consumer should delete it after success.

If processing will exceed the timeout, the consumer can call `ChangeMessageVisibility`. If the timeout expires first, the message can be delivered again.

Set visibility according to realistic processing time. Excessively short visibility creates premature duplicates; excessively long visibility delays recovery from a dead consumer.

### The Problem-Order Desk

#### Dead-Letter Queue

A redrive policy sends a message to a DLQ after `maxReceiveCount` is exceeded.

Monitor the source queue's oldest-message age and the DLQ's visible-message count. Redrive only after the underlying failure is understood, or the same poison message will repeat the journey.

### The Ordered Lanes

#### FIFO Queue

FIFO queues use `MessageGroupId` to preserve strict ordering within a group. Different groups can be processed concurrently.

`MessageDeduplicationId`, or content-based deduplication when configured, suppresses duplicate sends within the deduplication interval. A message in flight blocks later messages in the same group until it is deleted or becomes visible again.

Choose a group key that preserves necessary order without forcing unrelated orders into one serial lane.

### Tickets That Wait Before Appearing

#### Delay Queues and Message Timers

A delay prevents newly sent messages from becoming visible for a configured period. It is useful for deferred availability, not for long-running business scheduling.

For rich or long-duration waits, a workflow or EventBridge Scheduler is usually a clearer fit.

### How Long Tickets Remain

#### Message Retention

Messages expire after the queue's retention period if they are not deleted. Retention is a recovery window, not permanent event history.

### Waiting Efficiently

#### Long Polling

Long polling waits for messages to arrive instead of returning empty responses immediately. It reduces empty receives and can lower cost.

### Carrying Several Tickets

#### Batching

Send, receive, and delete batch operations reduce request overhead. A batch API can partially succeed, so code must inspect each entry's result.

### Lambda as the Cook

#### Event Source Mapping

For SQS, Lambda event-source mappings poll the queue and invoke the function with batches. Function code should be idempotent.

Without partial batch reporting, a failed batch can cause successfully handled messages in that batch to return. Reporting individual failures allows Lambda to retry only the failed messages when configured correctly.

---

## Architectural Mapping

```text
Message received
      |
      v
temporarily invisible
   /        \
success     failure / timeout
  |               |
delete       visible again
                  |
          receive count exceeds limit
                  |
                  v
                 DLQ
```

IAM authorizes SQS API calls. Queue policies are especially important when another service or account sends messages.

---

## When to Use It

Use:

- Standard queues for high-throughput work tolerant of duplicate delivery and best-effort order
- FIFO queues when strict order within message groups and send deduplication matter
- DLQs when repeated failures need isolation and investigation
- Long polling and batching to improve consumer efficiency

## When Not to Use It

Do not select FIFO merely because chronological order feels comforting. Global serialization can destroy concurrency while providing an ordering guarantee the business never required.

---

## Painkiller

> **Problem:** Consumers fail, messages return, and poison work repeats indefinitely.  
> **Pain:** Duplicate effects and blocked processing turn queue durability into business corruption.  
> **AWS solution:** Combine visibility, idempotent consumers, DLQs, appropriate queue type, and deliberate redrive.

---

## Knife Cut

> **Visibility controls when work may return. Idempotency controls whether returning work causes damage.**

---

## The Masthead

### What Actually Just Happened

|In the story|In SQS|What it actually means|
|---|---|---|
|Temporary cover|Visibility timeout|Period when a received message is hidden|
|Extending the cover|ChangeMessageVisibility|Adjusts invisibility for that receipt|
|Problem-order desk|DLQ|Queue receiving repeatedly failed messages|
|Labeled ticket lane|Message group|Ordering and concurrency boundary in FIFO|
|Duplicate ticket stamp|Deduplication ID|Suppresses matching FIFO sends within the window|
|Cook receives a stack|Lambda batch|Messages delivered together for processing|

---

## A Note From the Author

The rail makes retry timing look deterministic. Distributed delivery, worker crashes, Lambda batching, throttling, and visibility changes create more complicated outcomes.

FIFO documentation describes exactly-once processing in the context of queue deduplication and delivery design. Applications should still make consequential side effects idempotent because a consumer can fail after performing work but before acknowledging it.

- [SQS visibility timeout](https://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSDeveloperGuide/sqs-visibility-timeout.html)
- [SQS FIFO queues](https://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSDeveloperGuide/sqs-fifo-queues.html)

---

## The Last Bite

A returned ticket is not a broken queue.

It is the queue refusing to forget unfinished work.

> **Expect the message to return, and make the business effect safe when it does.**

---

**Next chapter:** *[Amazon SNS: Order 42 Is Ready](03-order-42-is-ready.md)*

The kitchen can now finish work safely. But one ready order must reach the customer, pickup screen, courier desk, and loyalty system.

