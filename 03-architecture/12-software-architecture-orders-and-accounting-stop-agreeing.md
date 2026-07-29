---
description: "Byte Burger needs an order and a payment record to converge even when separate systems, retries, and outages interrupt the journey."
tags:
  - "aws"
  - "architecture"
  - "software-architecture"
  - "distributed-data"
---

# Software Architecture: Orders and Accounting Stop Agreeing

## The Business Goal

Byte Burger needs an order and a payment record to converge even when separate systems, retries, and outages interrupt the journey.

## The Story

The order kitchen says “accepted”; accounting says “not charged”; a customer sees both. Support cannot simply retry the entire journey: it could create a second charge or a second meal. A single database transaction no longer spans the boundary. Byte Burger needs to know which fact is durable, which action is still pending, and who can repair the exception.

## The Decision

First, use an outbox when a local state change and the fact announcing it must not drift: the order is committed and its `OrderAccepted` fact is recorded for reliable publication together. An outbox does not coordinate payment; it prevents “order saved but event lost.”

For a customer journey with ordered dependent steps—reserve stock, charge payment, create delivery—use a saga or workflow coordinator. It records progress, retries eligible technical failures, and invokes a compensating action such as releasing stock when a later step cannot complete. A coordinator is useful when the business needs a visible owner of sequence and recovery.

Use choreography when independent consumers can react to a durable fact without a central conductor: analytics records the sale; loyalty awards points; notifications prepare a receipt. Each consumer owns safe retries and idempotency. Neither pattern removes the need for reconciliation: ambiguous payment responses, partner outages, and business disputes still require an auditable repair path.

## Decision Record

| Field | Decision |
| --- | --- |
| Valid while | Each bounded capability owns its state and explicit contracts |
| Operational tax | Event schemas, retries, idempotency, compensation, audit, and reconciliation |
| Invalidation trigger | Hidden coordination or shared data recreates a distributed monolith |
| Reversal path | Keep operations that truly require one atomic boundary inside one bounded module or system; do not simulate a global transaction with hidden shared data |

## The Last Bite

Across boundaries, correctness is a process of durable facts, safe retries, and reconciliation—not a wish for one global transaction.

**Next chapter:** *[Software Architecture: The Country-Wide Promotion](13-software-architecture-country-wide-promotion.md)*

Byte Burger can now coordinate a multi-step order and reconcile exceptions. The next promotion turns one durable fact into many independently owned reactions.
