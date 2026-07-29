---
description: "Byte Burger wants engineers to spend more time improving the business than coordinating thirty small deployments, dashboards, queues, and contracts."
tags:
  - "aws"
  - "architecture"
  - "software-architecture"
  - "overengineering"
---

# Software Architecture: We Built Too Much

## The Business Goal

Byte Burger wants engineers to spend more time improving the business than coordinating thirty small deployments, dashboards, queues, and contracts.

## The Story

The first promotion extraction worked, and the lesson was misread: every new initiative now deserved its own deployable service. The promotion service became three services. Each has a pipeline, alert, secret, schema, and on-call rotation. Some only relay data between their neighbors. The company finally measures the cost of its own cleverness.

## The Decision

Treat operational and cognitive load as architectural costs. Identify services without independent ownership, scale, release, or failure value. Preserve useful domain boundaries while questioning unnecessary runtime boundaries.

## Decision Record

| Field | Decision |
| --- | --- |
| Valid while | A service’s independence creates more value than operational cost |
| Operational tax | Platform work, incident coordination, latency, and fragmented expertise |
| Invalidation trigger | The service is mainly a remote wrapper, duplicate deployment, or unclear owner |
| Reversal path | Merge runtime boundaries while retaining modules and contracts that clarify the domain |

## The Last Bite

Complexity is not maturity. It is debt unless the business can name the constraint it pays to solve.

**Next chapter:** *[Software Architecture: The Great Consolidation](16-software-architecture-the-great-consolidation.md)*

The company has identified runtime boundaries that no longer create independent value. It must now remove them without erasing the domain boundaries that still clarify ownership.
