---
description: "Independent systems scale when they exchange work, facts, and state without every system knowing every other system."
tags:
  - "aws"
  - "application-integration"
  - "event-driven"
  - "orientation"
---

# AWS Application Integration: The Lunch Rush

> Independent systems scale when they exchange work, facts, and state without every system knowing every other system.

## The Business Goal

Byte Burger had found homes for customers, orders, payments, and inventory.

Its truth could survive.

Now that truth had to move.

At 11:58, Byte Burger looked ready.

The counter terminal called the payment system.

The payment system called the kitchen.

The kitchen called the pickup screen.

The delivery desk called the courier service.

The mobile application called almost everyone.

At noon, two hundred orders arrived.

One payment call slowed down. Counter terminals waited. Kitchen tickets stopped. Customers tapped **Place order** again. The delayed calls recovered together and duplicated work downstream.

Every department was open.

Byte Burger had still stopped moving.

---

## The Story

Nia drew Byte Burger on the back of a menu.

She noticed that not every conversation meant the same thing.

Some were requests:

> “Prepare order 42.”

Some were facts:

> “Order 42 was paid.”

Some were work that could wait:

> “A cook must make these fries.”

Some were announcements for several listeners:

> “Order 42 is ready.”

Some were one order's remembered journey:

> “Paid, preparing, assembled, dispatched.”

And some were a continuous river of activity:

> “Orders, payments, clicks, temperatures, and delivery scans.”

Byte Burger had treated all of them like phone calls.

That was the real failure.

---

## The Wrong Way

A direct synchronous call is useful when the caller needs an immediate answer.

It becomes dangerous when every component must be awake, fast, and reachable for the first component to succeed. A long chain makes the customer's outcome depend on the slowest link.

Retrying every failure immediately can make recovery worse. When hundreds of callers retry together, the recovering service receives another surge.

Loose coupling does not mean removing all direct calls. It means choosing deliberately where time, availability, and ownership should be separated.

---

## Meet the AWS Service

AWS provides different integration services because communication has different jobs.

> **Core idea:** A queue buffers work, a topic fans out a message, an event bus routes facts, a workflow remembers steps, a stream preserves ordered activity, and an API connects clients to data.

- **Amazon SQS** holds work until a consumer can process it.
- **Amazon SNS** pushes a published message to subscribed endpoints.
- **Amazon EventBridge** matches events and routes them to targets.
- **AWS Step Functions** coordinates a stateful sequence.
- **Amazon Kinesis Data Streams** carries a retained stream to consumers.
- **AWS AppSync** exposes GraphQL data operations and real-time client updates.

AWS manages the messaging and workflow infrastructure. The application still owns schemas, permissions, idempotency, error handling, monitoring, and business correctness.

---

## How It Works

### “Prepare This”

#### Command

A command asks a specific capability to perform work. `PrepareOrder` can be placed on SQS so the kitchen processes it asynchronously.

### “This Happened”

#### Event

`OrderPaid` is a fact. EventBridge can route it to inventory, loyalty, analytics, or fulfillment without the payment producer knowing those consumers.

### “Tell Everyone Interested”

#### Publication

SNS can fan `OrderReady` out to a pickup display, notification path, delivery integration, and an SQS queue.

### “Remember Where We Are”

#### Orchestration

Step Functions can remember that an order is paid but waiting for both food and drink before delivery begins.

### “Keep the Activity Flowing”

#### Stream

Kinesis retains ordered records within its partitioning model so dashboards, fraud detection, and forecasting consumers can read the same operational flow.

### “Keep the Customer Connected”

#### GraphQL and Real-Time Updates

AppSync lets a customer query a menu, place an order through a mutation, and subscribe to relevant status changes.

### One Kitchen or Independent Stations

#### Monoliths and Microservices

A monolith keeps application capabilities in one deployable system. In-process calls can be simpler, faster, and easier to keep consistent.

Microservices separate capabilities so teams can deploy, scale, and fail more independently. That independence introduces network calls, partial failure, distributed authorization, duplicate delivery, and observability needs.

Integration services help manage those boundaries. They are not a reason to create boundaries that the application does not need.

---

## Architectural Mapping

```text
Customer app -> AppSync -> order backend -> Step Functions
                                      |
                                      v
                                EventBridge
                               /     |      \
                            SQS     SNS    Kinesis
                          kitchen  alerts  analytics
```

This is one possible architecture, not a requirement to use every service. Each arrow must have an owner, permission model, failure policy, and observability plan.

---

## When to Use It

Introduce asynchronous integration when:

- Traffic arrives faster than a consumer can process it
- Producers and consumers should scale or fail independently
- One fact needs several reactions
- Workflows need durable state, branching, or waiting
- Multiple consumers need the same retained activity stream
- Clients need flexible data access and live updates

## When Not to Use It

Keep a direct synchronous request when the caller genuinely needs an immediate response and the dependency chain remains appropriate. Adding asynchronous infrastructure to a simple local operation can create needless complexity.

---

## Painkiller

> **Problem:** Direct calls make every Byte Burger system depend on every downstream system's speed and availability.  
> **Pain:** A single slowdown blocks orders and synchronized retries deepen the failure.  
> **AWS solution:** Use queues, topics, event routing, workflows, streams, and real-time APIs according to the communication job.

---

## Knife Cut

> **A command asks for work. An event reports a fact. A stream preserves continuing activity.**

---

## The Masthead

### What Actually Just Happened

|In the story|In AWS|What it actually means|
|---|---|---|
|Kitchen ticket|SQS message|Buffered unit of work|
|Pickup announcement|SNS publication|Message pushed to subscriptions|
|Operations switchboard|EventBridge bus and rules|Content-based event routing|
|Fulfillment runbook|Step Functions execution|Stateful orchestration|
|Activity conveyor|Kinesis data stream|Retained records read by consumers|
|Customer order screen|AppSync API|GraphQL operations and real-time updates|

---

## A Note From the Author

A restaurant provides physical intuition, not delivery guarantees. AWS services differ in duplication, ordering, persistence, retries, concurrency, and failure handling.

“Asynchronous” also does not mean “eventually correct.” Consumers must validate messages, make business effects idempotent where required, monitor backlogs, and reconcile missing outcomes.

- [AWS application integration services](https://docs.aws.amazon.com/whitepapers/latest/aws-overview/application-integration.html)

---

## The Last Bite

Byte Burger did not fail because it lacked workers.

It failed because every worker had to answer every other worker immediately.

> **Scale begins when communication stops requiring everyone to be ready at once.**

---

**Next chapter:** *[Amazon SQS: The Kitchen Ticket Rail](01-the-kitchen-ticket-rail.md)*

Nia's first repair is simple: let orders wait safely instead of making customers wait on the kitchen.
