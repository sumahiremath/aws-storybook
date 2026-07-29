---
description: "Table-level capacity can look healthy while one promoted key becomes the shelf everyone reaches for."
tags:
  - "aws"
  - "resilience"
  - "architecture"
  - "dynamodb"
  - "elasticache"
  - "sqs"
  - "capstone"
---

# Amazon DynamoDB, Amazon ElastiCache, and Amazon SQS: The Promotion Finds the Hot Shelf

> Table-level capacity can look healthy while one promoted key becomes the shelf everyone reaches for.

## The Business Goal

The payment supplier recovers, but a “free fries” promotion drives every order toward the same inventory record. DynamoDB throttles that access pattern. Queue age rises as workers wait. Adding more Lambda concurrency would only make the hot shelf more crowded.

## The Story

Every runner reaches for one bin marked `PROMO#FRIES`. The warehouse has plenty of other space, but that bin is the bottleneck. The manager serves safe repeated menu reads from the memory desk, spreads or redesigns the demand where the model permits it, and slows nonessential work rather than letting it crush checkout.

## Meet the AWS Service

DynamoDB partitions work by key. Uneven demand can create hot partitions. ElastiCache or DAX can reduce suitable repeated reads. SQS creates a buffer between intake and consumption, but consumers must still respect the capacity of the dependency they call.

## How It Works

Measure the constrained resource, then choose the appropriate lever:

- cache repeatable, safely stale reads;
- redesign a key/access pattern that concentrates demand;
- batch work only when the consumer and downstream system can handle it;
- cap concurrency or shed optional work to protect essential ordering;
- use backoff with jitter for transient throttling;
- monitor queue age, throttles, cache behavior, and customer outcome together.

## Architectural Mapping

| Story | AWS |
| --- | --- |
| One crowded bin | hot partition |
| Memory desk | cache/DAX |
| Ticket rail growing | SQS backlog |
| Pause optional orders | backpressure/graceful degradation |

## When to Use It

Use these controls when measured demand exceeds a component’s safe processing rate.

## When Not to Use It

Do not use a cache to hide a wrong write model or increase concurrency before checking downstream capacity.

## Painkiller

> **Problem:** A promotion concentrates work on one dependency.  
> **Pain:** More callers can amplify the hot spot instead of serving more customers.  
> **AWS solution:** Isolate pressure, reduce unnecessary reads, and redesign the concentrated access path.

## The Masthead

### What Actually Just Happened

| Story | AWS | Meaning |
| --- | --- | --- |
| Hot bin | hot DynamoDB partition | Uneven key demand |
| Growing ticket rail | queue age/depth | Consumer or dependency pressure |
| Limited menu | graceful degradation | Protect essential work |

## A Note From the Author

A cache has freshness and invalidation costs. A queue has retention and recovery behavior. Neither makes an unsafe business operation correct.

- [Designing DynamoDB partition keys](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/bp-partition-key-design.html)
- [Caching strategies and best practices](https://docs.aws.amazon.com/AmazonElastiCache/latest/dg/Strategies.html)

## The Last Bite

When one shelf is hot, protect it before inviting more runners into the warehouse.

**Next chapter:** *[Amazon CloudWatch, AWS X-Ray, and AWS CloudTrail: The Change Nobody Noticed](04-cloudwatch-xray-cloudtrail-change-nobody-noticed.md)*

The team has mitigated pressure. It still needs evidence for why the incident began after a release.
