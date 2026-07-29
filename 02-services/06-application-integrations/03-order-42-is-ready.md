---
description: "Publish one message and push copies to every relevant subscription without maintaining a private recipient list in the publisher."
tags:
  - "aws"
  - "application-integration"
  - "event-driven"
  - "sns"
---

# Amazon SNS: Order 42 Is Ready

> Publish one message and push copies to every relevant subscription without maintaining a private recipient list in the publisher.

## The Business Goal

When order 42 was ready, the kitchen called the customer.

Then it called the pickup display.

Then the courier desk.

Then loyalty.

Then the store manager.

One subscriber was offline, so the kitchen waited and retried while finished food cooled.

The kitchen had become a notification service.

---

## The Story

Nia gave the kitchen one announcement button.

The kitchen announced:

> “Order 42 is ready.”

The pickup display subscribed to pickup orders.

The courier desk subscribed to delivery orders.

The customer-notification system subscribed to both.

The analytics queue subscribed so it could process announcements later at its own rate.

The kitchen published once and returned to cooking.

---

## The Wrong Way

SNS is not a durable work queue for a consumer to poll whenever convenient.

SNS pushes messages to subscriptions according to the endpoint protocol and delivery policy. If a subscriber needs buffering and independent consumption, subscribe an SQS queue.

Adding many hard-coded HTTP calls to the publisher recreates tight coupling and makes the publisher own every subscriber's availability.

---

## Meet the AWS Service

> **Core idea:** Amazon SNS is a managed publish/subscribe service that fans one published message out to subscribed endpoints.

Publishers send to topics. Subscriptions connect topics to endpoints such as SQS, Lambda, HTTP/S, email, SMS, mobile push, and supported delivery services.

AWS manages topic fan-out and protocol delivery behavior. You manage topic type, subscriptions, filter policies, endpoint permissions, retries, dead-letter handling, message design, and cost.

---

## How It Works

### The Announcement Channel

#### Topic

A topic is the publication address. Publishers need `sns:Publish` permission.

Subscribers do not ask the publisher for messages. SNS delivers a copy to each matching subscription.

### The Listening Departments

#### Subscriptions

Each subscription combines a topic with an endpoint and protocol.

A topic with four matching subscriptions produces four delivery paths. Failure on one path does not require the publisher to call the other three again.

### Pickup or Delivery

#### Subscription Filter Policies

Without a filter policy, a subscription receives every topic message.

Filters can evaluate message attributes or a JSON message body, according to filter scope. A courier subscription can accept only messages whose fulfillment type is `DELIVERY`.

Filtering reduces needless downstream work. It does not replace endpoint authorization.

### Announce Now, Process Later

#### SNS-to-SQS Fan-Out

Subscribe an SQS queue when a department needs durable buffering, pull-based processing, or independent scaling.

The queue policy must allow the SNS topic to call `sqs:SendMessage`, normally constrained by the topic ARN.

### Immediate Code Reaction

#### SNS-to-Lambda

SNS can invoke subscribed Lambda functions. The Lambda resource-based policy must allow the SNS service principal and appropriate source.

The invocation is asynchronous. Function code must handle duplicate delivery and its own business idempotency.

### Failed Announcements

#### Delivery Retries and Subscription DLQs

SNS retry policy depends on the delivery protocol. HTTP/S subscriptions can define supported delivery-policy settings; AWS-managed endpoints use AWS-defined behavior.

When retries are exhausted, a subscription can send undeliverable messages to an SQS dead-letter queue. The DLQ belongs to the subscription because delivery failure occurs on one endpoint path.

### Ordered Announcements

#### FIFO Topics

FIFO topics provide ordering, message groups, and deduplication for supported FIFO fan-out patterns. Standard topics support the broadest protocol selection and do not provide strict ordering.

---

## Architectural Mapping

```text
Kitchen
   |
 Publish once
   v
SNS OrderReady topic
   |        |        |
 filter   filter    all
   v        v        v
pickup    courier   SQS analytics queue
display   system
```

SNS requires permission to reach protected endpoints. Monitoring should distinguish publishing success from delivery success on each subscription.

---

## When to Use It

Use SNS when:

- One publication should be pushed to multiple subscribers
- Subscribers use different supported protocols
- Subscription filtering can reduce unwanted delivery
- SQS queues need fan-out copies for independent consumers
- Mobile, SMS, email, HTTP/S, Lambda, or service endpoints fit the notification path

## When Not to Use It

Use SQS for one pool of competing consumers processing buffered work. Use EventBridge for richer event patterns, many event sources, cross-account routing, archives, and event-bus behavior.

---

## Painkiller

> **Problem:** The kitchen maintains and calls every notification destination.  
> **Pain:** Subscriber failures and recipient changes become kitchen failures and kitchen code changes.  
> **AWS solution:** Publish once to SNS and let filtered subscriptions own their delivery paths.

---

## Knife Cut

> **SNS pushes copies to subscriptions. SQS holds work for consumers to pull.**

---

## The Masthead

### What Actually Just Happened

|In the story|In SNS|What it actually means|
|---|---|---|
|Announcement button|Publish API|Sends one message to a topic|
|Announcement channel|Topic|Logical publication address|
|Listening department|Subscription|Delivery configuration for one endpoint|
|Pickup or delivery rule|Filter policy|Selects messages for a subscription|
|Department inbox|Subscribed SQS queue|Durable buffered fan-out copy|
|Undeliverable desk|Subscription DLQ|Captures exhausted delivery failures|

---

## A Note From the Author

A public announcement sounds ephemeral and best-effort. SNS stores published messages redundantly and applies protocol-specific retry behavior, but it is still not a consumer-polled work queue.

Subscribers must be authorized, monitored, and able to tolerate duplicate delivery where applicable. SMS, mobile push, email, HTTP/S, Lambda, and SQS do not share one identical delivery contract.

- [Amazon SNS features](https://docs.aws.amazon.com/sns/latest/dg/welcome-features.html)
- [SNS message filtering](https://docs.aws.amazon.com/sns/latest/dg/sns-message-filtering.html)

---

## The Last Bite

The kitchen should announce readiness once.

It should not become the address book for all of Byte Burger.

> **Publish the news; let subscriptions decide who receives a copy.**

---

**Next chapter:** *[Amazon EventBridge: The Operations Switchboard](04-the-operations-switchboard.md)*

Ready announcements solve fan-out. Nia's next problem is larger: hundreds of different facts must reach different systems according to what each fact means.

