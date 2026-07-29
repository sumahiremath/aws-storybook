---
description: "Byte Burger wants every store, analyst, loyalty system, and delivery partner to react to a country-wide promotion without blocking the order counter."
tags:
  - "aws"
  - "architecture"
  - "software-architecture"
  - "event-driven"
  - "scalability"
---

# Software Architecture: The Country-Wide Promotion

## The Business Goal

Byte Burger wants every store, analyst, loyalty system, and delivery partner to react to a country-wide promotion without blocking the order counter.

## The Story

One promotion fact becomes many independent needs. The company stops broadcasting one fragile call chain and publishes durable events. Consumers receive only the work they own, at their own pace. When order matters at high volume, related events travel through the same ordered lane; consumers can catch up from retained history when they fall behind.

## The Decision

Use fan-out and event routing for independent reactions; use streams when ordered, retained, high-volume event flow is the contract. Choose the grouping key consciously—for example, an order or customer—because it determines which events must stay in sequence. Monitor consumer lag and govern replay.

## Decision Record

| Field | Decision |
| --- | --- |
| Valid while | Consumers are independently owned and facts can be processed asynchronously |
| Operational tax | Schema evolution, ordering limits, duplicate handling, lag, and replay governance |
| Invalidation trigger | A consumer needs immediate coordinated control rather than a fact it can react to later |
| Reversal path | Use a simpler point-to-point or coordinated workflow when a consumer needs immediate shared control rather than an independent fact |

## The Last Bite

Fan-out turns one fact into many independent responsibilities; it does not remove the responsibility to govern them.

**Next chapter:** *[Software Architecture: Going International](14-software-architecture-going-international.md)*

The promotion can reach consumers across the country. Crossing national borders adds a different pressure: geography now affects latency, residency, and recovery obligations.
