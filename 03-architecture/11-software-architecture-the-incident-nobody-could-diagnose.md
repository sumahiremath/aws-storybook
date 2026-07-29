---
description: "Byte Burger needs support and engineers to explain an order's journey when an independent service, retry, or partner delay leaves the customer with an ambiguous result."
tags:
  - "aws"
  - "architecture"
  - "software-architecture"
  - "observability"
---

# Software Architecture: The Incident Nobody Could Diagnose

## The Business Goal

Byte Burger needs support and engineers to explain an order's journey when an independent service, retry, or partner delay leaves the customer with an ambiguous result.

## The Story

Promotions says a coupon was applied. Orders says the meal was accepted. The delivery partner says it timed out. Each service has a log, but none share the same order context. Support opens five dashboards and still cannot tell whether a retry would help, duplicate the meal, or charge the customer twice.

## The Decision

Treat observability as part of the runtime boundary. Propagate a correlation ID with the business operation; emit structured logs and meaningful metrics; trace calls across services; define owners and service-level objectives. Logs explain individual events, traces show the path and timing, and metrics reveal whether the pattern is growing.

## Decision Record

| Field | Decision |
| --- | --- |
| Valid while | Independent components can affect one customer outcome or operational promise |
| Operational tax | Telemetry standards, propagation tests, storage and sampling cost, dashboards, and alert ownership |
| Invalidation trigger | A business outcome cannot be reconstructed quickly enough to make a safe customer or recovery decision |
| Reversal path | Keep the shared operation context and right-size the telemetry; do not create elaborate tracing for a local, low-risk call that basic logs and metrics explain |

## The Last Bite

A distributed system is not observable because every service writes logs. It is observable when one business outcome can be followed across its boundaries.

**Next chapter:** *[Software Architecture: Orders and Accounting Stop Agreeing](12-software-architecture-orders-and-accounting-stop-agreeing.md)*

Shared operation context makes the journey visible. Visibility alone cannot repair the next failure, where separate systems commit different parts of one business outcome.
