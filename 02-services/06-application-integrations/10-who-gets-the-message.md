---
description: "Select a queue, topic, event bus, workflow, stream, or client API from the communication contract—not from a vague desire to be event driven."
tags:
  - "aws"
  - "application-integration"
  - "event-driven"
  - "comparison"
---

# AWS Application Integration: Who Gets the Message?

> Select a queue, topic, event bus, workflow, stream, or client API from the communication contract—not from a vague desire to be event driven.

## The Business Goal

Nia received six new requests.

> “The kitchen needs work buffered.”

> “Every interested system needs an order-ready notification.”

> “Different facts need different routes.”

> “One refund must follow an auditable sequence.”

> “Three analytics teams need the same activity history.”

> “The customer app needs live order status.”

One architect proposed EventBridge for all six because they were all “events.”

The word described the architecture's mood.

It did not describe the required behavior.

---

## The Story

Nia asked five questions:

1. Must the message wait until a worker is ready?
2. Should one consumer or many consumers receive it?
3. Is routing based on the content of a fact?
4. Must a process remember its current step?
5. Do independent readers need retained ordered activity?

Then she asked a sixth:

> “Is this communication really between backend services—or between the application and its user?”

The answers selected the service.

---

## The Wrong Way

Do not choose from service popularity or the number of targets on a diagram.

SNS and EventBridge can both fan out. SQS and Kinesis both hold data temporarily. Step Functions and EventBridge can both connect services. AppSync and SNS can both update a user experience.

The differences live in persistence, consumer model, routing, ordering, state, replay, and delivery channel.

---

## Meet the AWS Service

> **Core idea:** The correct integration service is the one whose communication and failure contract matches the workload.

Several services often work together:

```text
EventBridge -> SQS -> worker
SNS -> several SQS queues
Step Functions -> AWS service integrations
Kinesis -> Lambda consumers
backend mutation -> AppSync subscription
```

Composition is useful when every hop has a distinct job.

---

## How It Works

### Work Must Wait

#### Choose SQS

SQS persists messages until deletion or retention expiry and lets consumers pull at their own rate.

Use it for buffering, backpressure, competing workers, and retryable asynchronous jobs.

### One Publication, Many Delivery Paths

#### Choose SNS

SNS pushes a message to topic subscriptions and supports filtering and several endpoint protocols.

Use SNS for pub/sub fan-out, notifications, and SNS-to-SQS patterns.

### Route Facts by Meaning

#### Choose EventBridge

EventBridge matches structured events against patterns and routes them to AWS, cross-account, SaaS, or API targets.

Use it for event-driven architectures, many-source routing, schema tooling, archives and replay, and event-bus boundaries.

### Remember the Process

#### Choose Step Functions

Step Functions holds execution state and decides which declared step follows.

Use it for orchestration, branching, parallel work, waiting, retries, compensation, and auditable workflows.

### Preserve a Continuing Flow

#### Choose Kinesis Data Streams

Kinesis retains partitioned records so independent consumers can read and replay the same activity.

Use it for streaming, per-partition ordering, near-real-time processing, and multiple independent readers.

### Connect the Customer

#### Choose AppSync

AppSync provides typed GraphQL operations and real-time client subscriptions.

Use it when clients need queries, mutations, composed data access, and authorized live updates.

### Independent Reactions or Central Control

#### Choreography versus Orchestration

With choreography, services react to events without one central controller knowing the entire process.

With orchestration, a Step Functions state machine directs the process and remembers progress.

Choreography reduces central coupling but can make the overall journey harder to observe. Orchestration makes control flow explicit but creates a central workflow definition that must evolve.

### Immediate Answer or Deferred Work

#### Synchronous versus Asynchronous

Synchronous calls fit immediate request/response needs.

Asynchronous integration separates producer completion from consumer completion. It improves isolation and buffering but introduces eventual outcomes, duplicate handling, monitoring, and reconciliation.

### Remembering State

#### Stateful versus Stateless

SQS consumers can remain stateless between messages while durable business state lives elsewhere.

Step Functions executions are stateful by design. Kinesis consumers track stream position. AppSync clients maintain connections but must recover current application state after reconnecting.

---

## Architectural Mapping

|Requirement|Primary fit|Why|
|---|---|---|
|One pool of workers processes jobs|SQS|Buffered pull-based work|
|One message pushed to subscribers|SNS|Topic fan-out and protocol delivery|
|Facts routed by content|EventBridge|Patterns, buses, and targets|
|Remembered multi-step process|Step Functions|Stateful orchestration|
|Retained ordered activity for readers|Kinesis|Partitioned stream and replay window|
|Typed client data and live updates|AppSync|GraphQL operations and subscriptions|

|Dimension|SQS|SNS|EventBridge|Kinesis|
|---|---|---|---|---|
|Primary model|Queue|Topic|Event bus|Data stream|
|Consumption|Pull / polling abstraction|Push to subscriptions|Push to matching targets|Consumers read retained shards|
|Persistence|Until delete or expiry|Delivery-oriented, not consumer-polled storage|Routing-oriented; archive is optional|Retention window|
|Ordering|Best effort or FIFO groups|Standard or FIFO topic behavior|No ordering guarantee|Within shard/partition path|
|Fan-out|Competing consumers by default|Native subscription fan-out|Matching rules and targets|Independent consumers|
|Filtering|Not content-routed among consumers|Subscription filter policies|Rich event patterns|Consumers interpret records|
|Best clue|“Work must wait”|“Tell subscribers”|“Route this fact”|“Preserve this flow”|

---

## When to Use It

Use the smallest service combination that expresses the required contract. Add another hop only when it contributes buffering, routing, fan-out, state, replay, protocol delivery, or a security boundary.

## When Not to Use It

Do not assemble every integration service into one path to demonstrate architectural sophistication. Each hop adds latency, permissions, quotas, cost, monitoring, and another failure boundary.

---

## Painkiller

> **Problem:** Similar-looking integration services are selected from labels rather than behavioral requirements.  
> **Pain:** The architecture discovers too late that it needed buffering, ordering, replay, fan-out, or workflow state.  
> **AWS solution:** Choose from persistence, consumption, routing, ordering, state, replay, and client-interaction requirements.

---

## Knife Cut

> **SQS waits. SNS announces. EventBridge routes. Step Functions remembers. Kinesis streams. AppSync connects.**

---

## The Masthead

### What Actually Just Happened

|Byte Burger question|Architecture question|Primary answer|
|---|---|---|
|Who will cook this when ready?|Who should process buffered work?|SQS|
|Who should hear this announcement?|Which subscribers receive a copy?|SNS|
|Which departments care about this fact?|Which patterns route to which targets?|EventBridge|
|What must happen next for this order?|Who remembers control flow?|Step Functions|
|Who needs this continuing history?|Which consumers read the retained stream?|Kinesis|
|What should this customer see and change?|Which API and live channel serve the client?|AppSync|

---

## A Note From the Author

Comparison tables compress service behavior. FIFO variants, delivery protocols, EventBridge targets, Kinesis capacity modes, Step Functions workflow types, and AppSync authorization can change the detailed decision.

Validate current quotas, costs, regional support, integrations, encryption, and failure contracts before treating one row as architecture approval.

- [AWS decision guide: SQS, SNS, or EventBridge](https://docs.aws.amazon.com/decision-guides/latest/sns-or-sqs-or-eventbridge/sns-or-sqs-or-eventbridge.html)

---

## The Last Bite

Every service can move information.

The important question is what must happen while it moves—and what must remain when movement fails.

> **Choose the communication contract first. The service name follows.**

---

**Next chapter:** *[AWS Application Integration: Byte Burger Keeps Moving](11-the-restaurant-keeps-moving.md)*

The lunch rush began with every department calling every other department. It ends with each communication path carrying one clear promise.

