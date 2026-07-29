---
description: "Recovery means determining what happened to each order, not replaying every ticket and hoping for the best."
tags:
  - "aws"
  - "resilience"
  - "architecture"
  - "step-functions"
  - "dynamodb"
  - "sqs"
  - "capstone"
---

# Amazon SQS, AWS Step Functions, and Amazon DynamoDB: The Recovery Shift

> Recovery means determining what happened to each order, not replaying every ticket and hoping for the best.

## The Business Goal

The rush is over, but the Recovery Shelf holds failed messages. Some customers were charged and not notified. Others were accepted but never charged. A few completed every step before their final response was lost.

## The Story

The night shift does not dump every ticket back onto the rail. Each ticket is checked against the order ledger: accepted, payment pending, charged, kitchen sent, fulfilled, or cancelled. Only the missing action is retried. Irreparable records are investigated; the rest are reconciled safely.

## Meet the AWS Service

SQS DLQs retain failed source-queue messages after configured redrive behavior. Step Functions can make a multi-step business process, retries, catches, and compensation visible. DynamoDB can store durable order state and idempotency markers used to make recovery decisions.

## How It Works

Recovery needs an owner and a runbook:

- alarm on DLQ depth and oldest-message age;
- inspect failure reason and event schema before replay;
- use stable order and idempotency identifiers;
- reconcile external side effects such as payment with the provider when necessary;
- retry only the missing, eligible action;
- record the final business outcome and communicate it to the customer;
- fix the policy, code, capacity, or deployment condition that created the failure.

## Architectural Mapping

| Story | AWS |
| --- | --- |
| Recovery Shelf | DLQ |
| Order ledger | DynamoDB business state |
| Recovery checklist | Step Functions/runbook |
| One missing action | idempotent reconciliation |

## When to Use It

Use it when asynchronous work has business value and failure must become an owned operational process.

## When Not to Use It

Do not treat a DLQ as an archive with no alarm, owner, or replay safety plan.

## Painkiller

> **Problem:** Failed asynchronous work can leave the business in partial states.  
> **Pain:** Blind replay can duplicate payment, notification, or fulfillment.  
> **AWS solution:** Retain failures, reconcile durable state, and retry only what remains.

## The Masthead

### What Actually Just Happened

| Story | AWS | Meaning |
| --- | --- | --- |
| Recovery Shelf | DLQ | Retained failed source records |
| Ledger check | order/idempotency state | Evidence for safe replay |
| Night-shift procedure | workflow/runbook | Explicit recovery ownership |

## A Note From the Author

AWS cannot infer the business truth of an external payment or delivery. Reconciliation needs business identifiers, provider evidence, and a policy for customer communication.

- [Amazon SQS dead-letter queues](https://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSDeveloperGuide/sqs-dead-letter-queues.html)
- [Condition expressions in DynamoDB](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/Expressions.ConditionExpressions.html)

## The Last Bite

The safest replay is the one that knows what already succeeded.

**Next chapter:** *[AWS Applications: The Restaurant Under Pressure](06-aws-applications-restaurant-under-pressure.md)*

The final map collects the whole restaurant and the habits that let it survive the next rush.
