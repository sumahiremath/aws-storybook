---
description: "A retry can recover a transient failure, but it can also repeat work whose first result was merely lost."
tags:
  - "aws"
  - "apis"
  - "sdk"
---

# AWS SDK: The Busy Supplier Line

> A retry can recover a transient failure, but it can also repeat work whose first result was merely lost.

## The Business Goal

Marco sent a request to create an order. The supplier line timed out. He sent it again.

The first request had succeeded. Only its receipt was lost.

Two orders entered the kitchen.

## The Story

The operations assistant classified calls. A malformed supplier number produced a final rejection. A busy signal or temporary line failure waited, with increasing and randomized pauses, before trying again. But the assistant marked the create-order call as ambiguous: Byte Burger needed an idempotency key before repeating it could be safe.

## Meet the AWS SDK

SDK retry behavior can retry selected transient and throttling errors with backoff and jitter. It returns non-retryable errors and final failures to application code.

> **Core idea:** Retries address transport or capacity uncertainty. Idempotency addresses repeated business effect.

## How It Works

### Exceptions

Treat errors by meaning: validation and permission failures usually need correction, not immediate retry. Throttling and transient network/service failures may be retryable. Log an error's service code, request ID, operation, and safe context.

### Backoff and Jitter

Exponential backoff increases delay across attempts. Jitter randomizes it so many clients do not retry together and recreate the overload.

SDK defaults differ by language and can change. Set a deliberate retry mode and maximum attempts when the workload requires it.

### Idempotency

For an operation that creates an external effect, use a client token, conditional write, idempotency key, or service-supported idempotency mechanism. A network timeout does not reveal whether the service acted.

## Architectural Mapping

```text
SDK call -> transient/throttle? -> bounded backoff + jitter -> retry
           final error?         -> return error to application
           create operation?    -> idempotency key protects effect
```

## When to Use It

Use SDK retries for documented transient or throttling behavior. Use explicit idempotency for any retried create, payment, message, or state-transition operation.

## When Not to Use It

Do not retry validation, authorization, or malformed requests blindly. Do not use aggressive retries against a throttled service.

## Painkiller

> **Problem:** A missing response is treated as proof that no work happened.  
> **Pain:** Automatic or manual retries duplicate orders and worsen outages.  
> **AWS solution:** Use bounded SDK retry behavior for retryable failures and design idempotent business operations for ambiguous outcomes.

## Knife Cut

> **Retry decides whether to send again. Idempotency decides whether sending again can create another effect.**

## The Masthead

### What Actually Just Happened

|In the story|In AWS SDK behavior|What it actually means|
|---|---|---|
|Busy signal|Throttling/transient error|Potentially retryable failure|
|Random wait|Backoff with jitter|Spread recovery traffic|
|Final refusal|Non-retryable exception|Return error to application|
|Order reference card|Idempotency key/token|Repeat request resolves to one effect|

## A Note From the Author

Never assume every SDK or service retries the same codes with the same defaults. Verify the specific SDK and operation. Retrying a read may be safe while retrying a write needs a business-level guard.

- [AWS SDK retry behavior](https://docs.aws.amazon.com/sdkref/latest/guide/feature-retry-behavior.html)

## The Last Bite

The line could fail without forcing Byte Burger to guess.

> **A timeout is uncertainty, not a rollback.**

**Next chapter:** *[AWS SDK: The Long Inventory Receipt](08-aws-sdk-the-long-inventory-receipt.md)*

The next supplier receipt contained one hundred inventory rows and a marker saying there were more. Marco nearly threw the marker away.
