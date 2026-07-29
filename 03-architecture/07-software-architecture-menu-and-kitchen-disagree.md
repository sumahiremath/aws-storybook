---
description: "Byte Burger wants customers to see a fast menu without selling ingredients that are already gone."
tags:
  - "aws"
  - "architecture"
  - "software-architecture"
  - "data-consistency"
---

# Software Architecture: The Menu Says Available, the Kitchen Says Sold Out

## The Business Goal

Byte Burger wants customers to see a fast menu without selling ingredients that are already gone.

## The Story

The menu board keeps a copy of popular information near the counter. During a rush, the board still says fries are available after the kitchen has sold the last batch. A customer pays, reaches the counter, and learns that the meal cannot be made. Support now has a refund, an apology, and a loyalty promise to repair.

## The Decision

Use caching for data that can be reused safely, define TTL and invalidation behavior, and protect the authoritative inventory write with conditional or transactional logic. The cache is a performance layer, not the truth.

## Decision Record

| Field | Decision |
| --- | --- |
| Valid while | Brief staleness is acceptable for the cached view |
| Operational tax | Invalidation, freshness monitoring, and fallback behavior |
| Invalidation trigger | A stale answer causes unacceptable financial, safety, or customer harm |
| Reversal path | Bypass the cached view for availability decisions that can no longer tolerate staleness |

## The Last Bite

Fast copies are useful only when everyone knows which record is allowed to be stale.

**Next chapter:** *[Software Architecture: Marketing Creates a Stampede](08-software-architecture-marketing-creates-a-stampede.md)*

Byte Burger has learned which record may be stale. A national promotion will now test whether the entire system knows which work must remain available under pressure.
