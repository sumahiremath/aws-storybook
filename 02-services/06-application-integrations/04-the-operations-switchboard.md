---
description: "Publish structured facts to an event bus and route matching events to targets without teaching producers who consumes them."
tags:
  - "aws"
  - "application-integration"
  - "event-driven"
  - "eventbridge"
---

# Amazon EventBridge: The Operations Switchboard

> Publish structured facts to an event bus and route matching events to targets without teaching producers who consumes them.

## The Business Goal

Byte Burger now produced dozens of facts:

- `OrderPlaced`
- `PaymentApproved`
- `InventoryLow`
- `OrderReady`
- `DeliveryDelayed`
- `RefundCompleted`

Payment knew to call fulfillment.

Fulfillment knew to call inventory.

Inventory knew to call purchasing.

Then loyalty wanted `OrderPlaced`. Fraud wanted `PaymentApproved`. A regional operations account wanted every `DeliveryDelayed`.

Each new listener required another producer change.

---

## The Story

Nia installed an operations switchboard.

Departments reported structured facts:

> Source: `restaurant.orders`  
> Type: `OrderPlaced`  
> Detail: `{ "store": "SEA-12", "channel": "MOBILE" }`

The switchboard compared each report against routing cards.

Mobile orders went to digital fulfillment.

Delivery delays went to the courier desk and regional operations.

High-value payments went to fraud review.

The order service reported what happened and returned to its own work. It did not know how many routing cards matched.

---

## The Wrong Way

An event bus is not a queue where an event waits until one consumer claims it.

If no rule matches, the bus does not preserve the event for a future rule unless an archive or another target was configured.

An event is also not automatically a perfect immutable business fact. Producers own schema quality, identifiers, sensitivity, versioning, and truthful publication.

---

## Meet the AWS Service

> **Core idea:** Amazon EventBridge receives events on buses, matches them against rule patterns, and routes matching events to targets.

Sources include AWS services, custom applications, and supported SaaS integrations. A matching event can reach zero or more targets through one or more rules.

AWS manages event ingestion and routing. You manage event contracts, buses, patterns, targets, permissions, retries, DLQs, archives, schedules, and consumer correctness.

---

## How It Works

### The Switchboard

#### Event Bus

The default bus receives events from AWS services. Custom buses receive application events. Partner buses support eligible SaaS sources.

An event bus can use a resource policy to accept events from another account or organization.

### The Routing Cards

#### Rules and Event Patterns

A rule belongs to one bus. Its event pattern matches fields such as `source`, `detail-type`, and values inside `detail`.

Patterns match structure; they are not arbitrary application code. Test them against representative and adversarial examples so a broad pattern does not route unexpected future events.

### The Receiving Departments

#### Targets

Targets include Lambda, SQS, SNS, Step Functions, other event buses, API destinations, and many AWS services.

A rule can have multiple targets, though separate one-target rules often make ownership and change isolation clearer.

EventBridge can transform the event before target delivery. The target must receive the fields its contract expects.

### The House Style

#### Schema Registry and Discovery

Schemas describe event structure. EventBridge includes AWS-service schemas and supports custom schemas, discovery, and code bindings.

Discovery learns from observed events; it does not absolve producers and consumers from intentional schema evolution.

### The Incident Recorder

#### Archive and Replay

An archive stores matching events from one source bus for a configured retention period. Replay sends selected archived events back to the original event bus and optionally through selected rules.

Replay can recover processing after a fix or feed new functionality. Consumers must be idempotent because historical facts can cause effects again.

### The Opening Timer

#### EventBridge Scheduler

EventBridge Scheduler supports one-time and recurring invocations with target, retry, and failure-retention configuration.

Legacy scheduled rules still exist, but Scheduler is the recommended scheduling capability for new scheduled work.

### Another Restaurant Group

#### Cross-Account Routing

A source rule can target an event bus in another account when IAM roles and event-bus resource policies permit it.

Cross-account routing changes the trust boundary. Limit who may publish and which events may traverse it.

### The Dedicated Prep Chute

#### EventBridge Pipes

A pipe connects one supported source to one target with optional filtering, enrichment, and transformation.

Pipes are point-to-point. Event buses are many-source to many-target routers.

### Delivery Failure

#### Retry and Dead-Letter Queue

Targets can fail. Configure target retry behavior and an SQS DLQ where supported so exhausted events can be investigated rather than silently forgotten.

EventBridge delivery is at least once and does not guarantee event order.

---

## Architectural Mapping

```text
Order service ----\
Payment service ---+--> event bus --> rules/patterns --> targets
AWS services -----/                       |
                                          +--> archive

supported stream or queue --> Pipe: filter -> enrich -> one target
```

Publishing permission, bus policy, target role, resource policy, KMS configuration, and cross-account trust may all participate in one route.

---

## When to Use It

Use EventBridge when:

- Producers publish structured facts without knowing consumers
- Content-based patterns route events to AWS or cross-account targets
- Many event sources share a routing layer
- Archive and replay support recovery or new consumers
- Schedules or point-to-point Pipes solve adjacent integration needs

## When Not to Use It

Use SQS when work must wait for a consumer and backlog provides backpressure. Use SNS for straightforward push fan-out across supported subscription protocols. Use Step Functions when one business process needs remembered state and ordered control flow.

---

## Painkiller

> **Problem:** Every producer maintains direct knowledge of every system interested in its facts.  
> **Pain:** Adding consumers changes producers and spreads failures through Byte Burger.  
> **AWS solution:** Publish structured events to EventBridge and route matches through independently owned rules.

---

## Knife Cut

> **A producer reports what happened. EventBridge decides which configured routes match. Targets decide what the fact means to them.**

---

## The Masthead

### What Actually Just Happened

|In the story|In EventBridge|What it actually means|
|---|---|---|
|Structured report|Event|JSON fact with source, type, and detail|
|Switchboard|Event bus|Router receiving events|
|Routing card|Rule and event pattern|Match logic associated with a bus|
|Department|Target|AWS service or endpoint receiving a match|
|House style|Schema registry|Event structure and optional code bindings|
|Incident recorder|Archive and replay|Retained events sent back through the bus|
|Dedicated chute|Pipe|One source through filter/enrichment to one target|

---

## A Note From the Author

The switchboard suggests a human understands the meaning of every report. EventBridge performs configured pattern matching; it does not infer business intent or validate that an event is truthful.

Event delivery is at least once and unordered. Replays deliberately create another delivery of historical events. Targets need idempotency, observability, and safe failure handling.

- [EventBridge event buses](https://docs.aws.amazon.com/eventbridge/latest/userguide/eb-event-bus.html)
- [EventBridge archive and replay](https://docs.aws.amazon.com/eventbridge/latest/userguide/eb-archive.html)

---

## The Last Bite

The payment system should know that payment succeeded.

It should not know every future system that may care.

> **Publish the fact once; let routing evolve without rewriting the source.**

---

**Next chapter:** *[AWS Step Functions: The Fulfillment Runbook](05-the-fulfillment-runbook.md)*

Events let departments react independently. But order 42 needs payment, food, drink, assembly, and delivery to complete as one remembered journey.

