---
description: "Byte Burger wants to reduce incident load and release friction without losing the boundaries that protect important business capabilities."
tags:
  - "aws"
  - "architecture"
  - "software-architecture"
  - "consolidation"
---

# Software Architecture: The Great Consolidation

## The Business Goal

Byte Burger wants to reduce incident load and release friction without losing the boundaries that protect important business capabilities.

## The Story

The team does not demolish the whole city. It brings related stations back into one well-marked kitchen, retires duplicate event hops, and keeps the few independently valuable services separate. Each move has a rollback plan and a measure of success.

## The Decision

Consolidate deliberately: map callers and data ownership, preserve contracts during migration, reconcile state, remove unused infrastructure, and keep modular boundaries inside the resulting application.

## Decision Record

| Field | Decision |
| --- | --- |
| Valid while | Consolidation reduces cost and coordination without violating needed isolation |
| Operational tax | Migration risk, data reconciliation, compatibility, and decommissioning work |
| Invalidation trigger | Consolidation begins to violate an isolation, scale, or recovery boundary that the business can measure |
| Reversal path | If consolidation violates a measured isolation, scale, or recovery boundary, restore the independent boundary through the preserved compatibility layer and migration checkpoint before retiring the old path |

## The Last Bite

De-architecture is not retreat. It is choosing the smallest system that still tells the truth about the business.

**Next chapter:** *[Software Architecture: What We Would Build Today](17-software-architecture-what-we-would-build-today.md)*

After adding and removing complexity, Byte Burger can finally answer the honest retrospective question: which decisions would it make first if it began again with current evidence?
