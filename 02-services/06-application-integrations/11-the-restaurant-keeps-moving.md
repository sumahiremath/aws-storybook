---
description: "Reliable integration gives work, facts, workflows, streams, and client state distinct paths with explicit failure behavior."
tags:
  - "aws"
  - "application-integration"
  - "event-driven"
  - "epilogue"
---

# AWS Application Integration: Byte Burger Keeps Moving

> Reliable integration gives work, facts, workflows, streams, and client state distinct paths with explicit failure behavior.

## The Business Goal

At the start of lunch, Byte Burger looked connected.

Every system called another system.

Every department knew another department's address.

Every failure traveled backward until it reached the customer.

The architecture had many connections.

It had very little independence.

---

## The Story

At 2:03, the rush ended.

The kitchen ticket rail still held a manageable backlog.

Pickup announcements had reached their matching subscriptions.

The operations switchboard had routed facts without changing their producers.

Fulfillment runbooks showed exactly where incomplete orders waited.

The receipt stream allowed analytics consumers to catch up independently.

Customer applications displayed current order state.

The payment provider had recovered without receiving a synchronized retry attack.

Byte Burger had not eliminated failure.

It had stopped turning every local failure into one shared collapse.

---

## Meet the AWS Service

> **Core idea:** Application integration is the deliberate separation of communication responsibilities and failure boundaries.

The season's services answer different questions:

- Where should work wait?
- Who should receive a pushed copy?
- Which facts match which routes?
- What step should happen next?
- Who needs the retained activity?
- What should the connected client see?

No single answer is “event driven.” The design is a collection of precise contracts.

---

## How It Works

### Work Waits

SQS buffers jobs and lets consumers process at a sustainable rate. Visibility, deletion, duplicate delivery, DLQs, FIFO groups, and batching define the failure contract.

### News Fans Out

SNS publishes once to topic subscriptions. Filters, endpoint protocols, retries, permissions, and subscription DLQs define each delivery path.

### Facts Find Routes

EventBridge receives structured events and matches rules. Event buses, targets, schemas, archives, replay, schedules, cross-account policies, and Pipes support evolving event-driven systems.

### Processes Remember

Step Functions holds execution state across tasks, choices, parallel branches, maps, waits, and failures. Workflow type, transformations, retries, catches, and compensation define recovery.

### Activity Remains Readable

Kinesis retains partitioned records for independent consumers. Shards, keys, capacity, ordering scope, checkpoints, retention, enhanced fan-out, and batch recovery shape the stream.

### Customers Stay Connected

AppSync exposes GraphQL queries and mutations and distributes mutation-driven subscription updates to authorized connected clients. Resolvers, data sources, sync, conflicts, and reconnection complete the client contract.

### Failure Has a Budget

Timeouts stop indefinite waiting. Bounded retry handles suitable transient failure. Backoff and jitter reduce retry pressure. Idempotency limits repeated effects. Circuit breaking and reconciliation provide outcomes when retry is no longer useful.

---

## Architectural Mapping

```text
Client request
     |
AppSync / backend
     |
Step Functions execution
     |
business facts -> EventBridge
     |            |          |
    SQS          SNS       Kinesis
 buffered      fan-out     retained flow
 work
```

The diagram is not a prescription. A healthy design may use two services or several. Every arrow must justify its existence.

---

## When to Use It

Return to this model when:

- A dependency slowdown blocks unrelated work
- Producers know too much about consumers
- Bursts overwhelm downstream capacity
- One fact needs independent reactions
- A process loses track of partial completion
- Analytics consumers compete for one copy of activity
- Clients poll continuously for changing state
- Retries amplify rather than absorb failure

## When Not to Use It

Do not split a simple application into distributed components merely to use integration services. Network boundaries create partial failure, latency, authorization, observability, and operational cost.

---

## Painkiller

> **Problem:** Directly connected systems share timing, availability, and failure across the entire application.  
> **Pain:** One slow dependency blocks customers and makes recovery traffic deepen the outage.  
> **AWS solution:** Give buffered work, fan-out, event routing, orchestration, streams, and client updates separate managed integration paths.

---

## Knife Cut

> **Loose coupling is not fewer responsibilities. It is clearer ownership of each responsibility.**

---

## The Masthead

### What Actually Just Happened

|Lunch-rush lesson|Architecture lesson|
|---|---|
|Tickets wait for cooks|Queues separate producer and consumer time|
|Announcements reach listeners|Topics fan out to subscriptions|
|Facts follow routing cards|Event buses route by pattern|
|Runbooks remember progress|Workflows orchestrate stateful processes|
|Receipts remain on the conveyor|Streams retain partitioned activity|
|The app listens for order changes|Real-time APIs update connected clients|
|Retries return at different times|Backoff, jitter, and idempotency constrain recovery|

---

## A Note From the Author

Byte Burger ends with clean lines and a quiet dashboard. Production systems remain dynamic: schemas evolve, consumers lag, permissions drift, quotas throttle, poison messages appear, and external dependencies fail in new ways.

The mental model is complete only when paired with metrics, alarms, tracing, runbooks, replay tests, DLQ ownership, security review, and cost monitoring.

- [AWS Application Integration documentation](https://docs.aws.amazon.com/whitepapers/latest/aws-overview/application-integration.html)

---

## The Last Bite

Byte Burger did not become reliable because nothing failed.

It became reliable because work could wait, facts could travel, workflows could remember, streams could continue, and failure had somewhere to go.

> **The systems stopped shouting at one another. Byte Burger kept moving.**

---

**Next section:** *[AWS Deployment: The Friday Franchise Rollout](../07-deployment/00-the-friday-franchise-rollout.md)*

The application can now communicate under pressure. The next challenge is changing its code and infrastructure without turning every release into another lunch-rush outage.

