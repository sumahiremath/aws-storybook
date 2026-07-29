---
description: "Buffer work so producers can continue while consumers process messages at their own sustainable rate."
tags:
  - "aws"
  - "application-integration"
  - "event-driven"
  - "sqs"
---

# Amazon SQS: The Kitchen Ticket Rail

> Buffer work so producers can continue while consumers process messages at their own sustainable rate.

## The Business Goal

Before the lunch rush, the counter clerk waited for a cook to answer.

If every cook was busy, the clerk held the customer at the register.

If the kitchen restarted, the spoken order disappeared.

If ten cooks became free at once, nobody knew which cook owned which meal.

The kitchen needed fewer phone calls.

It needed somewhere for work to wait.

---

## The Story

Nia installed a ticket rail between the counters and the kitchen.

Cashiers clipped new tickets to one end and returned to customers.

Available cooks pulled tickets from the other end.

When a cook finished a meal, the ticket was removed. When a cook vanished halfway through, the ticket eventually returned for another cook.

The counter no longer needed to know which cook was working.

The kitchen no longer controlled how quickly customers could place orders.

---

## The Wrong Way

A queue is not merely a slower API call.

If the cashier waits synchronously for the meal anyway, the system has kept the coupling while adding infrastructure.

A consumer must also avoid deleting a message before the business work succeeds. Early deletion turns a temporary worker failure into lost work.

---

## Meet the AWS Service

> **Core idea:** Amazon SQS is a managed message queue that decouples producers from consumers and buffers work between their rates.

A producer sends a message. A consumer receives it, processes it, and deletes it after successful handling.

AWS manages queue availability and message storage. You manage message meaning, queue type, permissions, visibility, retention, scaling, deletion, idempotency, and failure recovery.

---

## How It Works

### Clipping the Ticket

#### `SendMessage`

The producer sends a message body and optional attributes. The body should contain enough information to locate or perform the work without embedding unnecessary sensitive or oversized data.

IAM and queue policies determine who may send.

### Taking the Ticket

#### `ReceiveMessage`

Consumers poll the queue. A response can contain one or more messages and receipt handles.

The receipt handle identifies that particular receipt attempt. It is used when deleting or changing visibility.

### Cooking the Meal

#### Processing Outside SQS

SQS stores messages; it does not execute the business logic. EC2 workers, containers, Lambda, or other consumers perform the work.

Consumers should be stateless where practical and scale according to backlog, processing time, downstream capacity, and cost.

### Removing the Ticket

#### `DeleteMessage`

Receiving does not remove the message. After successful processing, the consumer deletes it using the receipt handle.

If processing fails or the consumer disappears, the message can become visible again.

### The Standard Rail

#### Standard Queue

Standard queues support high throughput with at-least-once delivery and best-effort ordering.

A message can arrive more than once or in a different order than expected. Business effects such as charging, refunding, or decrementing scarce inventory therefore need idempotency or another correctness control.

---

## Architectural Mapping

```text
Counter / mobile backend
          |
      SendMessage
          v
       SQS queue
          |
     ReceiveMessage
          v
     kitchen workers
          |
      DeleteMessage
```

Queue depth provides backpressure visibility. When arrivals exceed processing, the queue grows rather than forcing the producer to remain connected to a slow consumer.

---

## When to Use It

Use SQS when:

- Work can happen asynchronously
- Producers and consumers need independent scaling
- Traffic bursts must be buffered
- Consumers should pull work at a sustainable rate
- A durable queue is more useful than immediate broadcast

## When Not to Use It

Use SNS when one publication should be pushed to many subscribers. Use EventBridge when facts need content-based routing to targets. Use Kinesis when multiple consumers need a retained stream rather than competing for individual work messages.

---

## Painkiller

> **Problem:** The counter cannot accept orders faster than available cooks answer.  
> **Pain:** Kitchen delays become customer-facing outages and spoken work disappears during failure.  
> **AWS solution:** SQS buffers messages so producers continue while consumers pull and complete work independently.

---

## Knife Cut

> **Receiving reserves a chance to work. Deleting acknowledges that this delivery attempt is complete.**

---

## The Masthead

### What Actually Just Happened

|In the story|In SQS|What it actually means|
|---|---|---|
|Cashier|Producer|Sends a work message|
|Ticket rail|Queue|Durably buffers messages|
|Cook|Consumer|Polls and processes messages|
|Taking a ticket|ReceiveMessage|Returns a message and receipt handle|
|Removing a ticket|DeleteMessage|Acknowledges successful processing|

---

## A Note From the Author

The physical rail suggests each ticket can be held by only one cook with perfect certainty. A Standard queue uses at-least-once delivery; duplicate processing remains possible.

Queue permissions, encryption, quotas, message size, retention, consumer concurrency, and downstream limits still require deliberate design.

- [Amazon SQS Developer Guide](https://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSDeveloperGuide/welcome.html)

---

## The Last Bite

The cashier should not wait for a free cook.

The order should.

> **Put time between the producer and consumer without losing the work between them.**

---

**Next chapter:** *[Amazon SQS: The Ticket That Came Back](02-the-ticket-that-came-back.md)*

The rail survives the rush. Then one cook claims a ticket, drops it, and the same order returns.

