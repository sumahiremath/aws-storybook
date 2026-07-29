---
description: "The checkout Lambda is now fast, yet some orders still slow down during a promotion. Table capacity looks generous. The team proposes buying more pantry shelves for every ingredient."
tags:
  - "aws"
  - "observability"
  - "operations"
  - "dynamodb"
---

# Amazon DynamoDB: The Hot Pantry

## The Business Goal

The checkout Lambda is now fast, yet some orders still slow down during a promotion. Table capacity looks generous. The team proposes buying more pantry shelves for every ingredient.

## The Story

The General Manager walks into the pantry and sees the flaw: every promotion order reaches for the same bin labeled `TODAY-SPECIAL`. Other bins are almost untouched. The total pantry may be large, but the popular bin has become the bottleneck.

Adding shelves to the quiet side of the room will not make that one bin easier to reach. The team first confirms the access pattern, then spreads the demand with a better key design or changes how the promotion is read. For repeat reads that can tolerate cached data, they consider a cache layer—but keep the source of truth and freshness rules clear.

## Meet the AWS Service

DynamoDB throttling can reflect insufficient configured capacity, on-demand scaling behavior, account/service limits, or an uneven access pattern that creates a **hot partition**. CloudWatch metrics and application-level evidence help distinguish those cases. A partition key with poor distribution can concentrate traffic even when table-level capacity appears sufficient.

DynamoDB Accelerator (DAX) and other caching approaches reduce repeated read pressure for appropriate access patterns. They are performance tools, not a universal answer to write-heavy or strongly fresh-read requirements.

## How It Works

When DynamoDB appears in an incident:

1. Confirm the table/index and operation experiencing throttling or elevated latency.
2. Correlate it with request keys, traffic shape, and recent application changes.
3. Ask whether the partition-key design concentrates popular work.
4. Choose a remedy: distribute the key space, model the access pattern differently, adjust capacity mode/settings where appropriate, smooth the caller's retry behavior, or cache safe repeated reads.

Use exponential backoff with jitter for transient throttling. Do not let many callers retry in lockstep. A `Scan` used as a recurring hot-path lookup can create unnecessary work; a `Query` aligned with the key design is usually the more intentional read operation.

## Architectural Mapping

| Byte Burger | DynamoDB |
| --- | --- |
| Ingredient bins | partitions |
| Label used to choose a bin | partition key |
| Everyone reaching for one bin | hot partition |
| Pre-staged commonly requested menu card | cache/DAX |

## Painkiller

Fix uneven demand at the access-pattern level. Capacity is important, but it cannot undo a key that funnels a crowd into one place.

## Knife Cut

Caching can reduce reads but introduces freshness, invalidation, and failure-mode decisions. Do not use it to conceal an incorrect data model.

## The Masthead

The team now has several true clues. The General Manager's job is to turn them into a cause—not a confident guess.

## A Note From the Author

See [DynamoDB throttling diagnosis](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/throttling-diagnosing-workflow.html) and [DynamoDB Accelerator](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/DAX.html).

## The Last Bite

When one pantry bin is hot, the right remedy is rarely “make every bin bigger.”

**Next chapter:** *[Troubleshooting and Optimization: The General Manager](10-troubleshooting-general-manager.md)*

Next: the General Manager runs a disciplined root-cause investigation.
