---
description: "Retrying a slow supplier without judgment turns one delay into a larger outage."
tags:
  - "aws"
  - "resilience"
  - "architecture"
  - "sdk"
  - "step-functions"
  - "capstone"
---

# AWS SDK and AWS Step Functions: The Payment Supplier Slows Down

> Retrying a slow supplier without judgment turns one delay into a larger outage.

## The Business Goal

The payment provider begins timing out. The SDK retries, Lambda retries, and the workflow retries. The supplier receives a growing wave of duplicate work while recovery becomes less likely.

## The Story

The manager closes the supplier line briefly after enough failed calls. New orders remain safely on the ticket rail. A few careful probes test whether the supplier has recovered. Each payment carries the same order reference, so a repeated call cannot create a second charge.

## Meet the AWS Service

AWS SDK retry behavior, Step Functions `Retry` and `Catch`, queues, and application idempotency work together. They do not replace one another. A retry policy handles eligible transient failure; a circuit-breaker pattern stops futile calls; idempotency makes repeated business attempts safe.

## How It Works

Classify the error first:

- malformed input, rejected payment, or missing permission: fail and correct;
- timeout, throttling, or temporary service error: bounded retry may fit;
- sustained dependency failure: stop sending normal traffic, preserve work, and degrade honestly.

Use exponential backoff and jitter so callers do not retry together. Use a stable idempotency key for side-effecting operations. A workflow can make retries, catches, compensation, and the current business state explicit; it cannot guarantee an external payment provider performed an action only once.

## Architectural Mapping

| Story | AWS pattern |
| --- | --- |
| Supplier line closed | circuit breaker |
| Increasing, randomized pause | backoff and jitter |
| Same payment reference | idempotency key |
| Managed recovery runbook | Step Functions retry/catch |

## When to Use It

Use it for transient failures around a dependency that can safely be retried.

## When Not to Use It

Never retry permanent business failures or ambiguous side effects without idempotency.

## Painkiller

> **Problem:** A dependency can become slow or unavailable.  
> **Pain:** Unbounded retries turn dependency recovery into cascading failure.  
> **AWS solution:** Bound retries, spread them, preserve work, and make side effects duplicate-safe.

## The Masthead

### What Actually Just Happened

| Story | AWS | Meaning |
| --- | --- | --- |
| Careful probe | circuit-breaker recovery test | Limited re-entry after failure |
| Payment reference | idempotency | Repeated request has one business effect |
| Saved ticket | queue/workflow state | Work remains recoverable |

## A Note From the Author

Circuit breaker is an application pattern, not a single AWS switch. Choose timeout budgets, retry limits, failure thresholds, and customer communication from measured behavior.

- [AWS SDK retry behavior](https://docs.aws.amazon.com/sdkref/latest/guide/feature-retry-behavior.html)
- [AWS Step Functions error handling](https://docs.aws.amazon.com/step-functions/latest/dg/concepts-error-handling.html)

## The Last Bite

Recovery needs fewer requests, not louder requests.

**Next chapter:** *[Amazon DynamoDB, Amazon ElastiCache, and Amazon SQS: The Promotion Finds the Hot Shelf](03-dynamodb-elasticache-sqs-promotion-finds-hot-shelf.md)*

The supplier is protected. The promotion now exposes a different bottleneck inside Byte Burger.
