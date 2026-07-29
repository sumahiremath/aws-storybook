---
description: "The promotions engine now creates extreme bursts, changes on its own cadence, and can fail without stopping payment or order entry. Byte Burger needs one boundary to become independently operational."
tags:
  - "aws"
  - "architecture"
  - "software-architecture"
  - "microservices"
---

# Software Architecture: The First Service Extraction

## The Business Goal

The promotions engine now creates extreme bursts, changes on its own cadence, and can fail without stopping payment or order entry. Byte Burger needs one boundary to become independently operational.

## The Story

On a flash-sale Friday, promotion recalculation consumes the shared application’s worker capacity just as checkout traffic peaks. Customers see payment retries for an order problem that promotions caused; marketing cannot deploy its correction without joining the order-release window. This time, the team does not split every kitchen station. It extracts promotions because the business pressure is real: its own owner, scaling pattern, release rhythm, and failure boundary.

## The Decision

Extract one service around a clear business capability. Give it an API or event contract, owned data, observable dependencies, and a migration path. Avoid shared-database shortcuts that preserve hidden coupling.

## Decision Record

| Field | Decision |
| --- | --- |
| Valid while | The capability’s independent needs outweigh network and operations cost |
| Operational tax | Contracts, tracing, retries, deployments, data ownership, and on-call responsibility |
| Invalidation trigger | The service cannot evolve independently or is mostly a remote wrapper around the monolith |
| Reversal path | Reintegrate it as a module while preserving the useful domain boundary |

## The Last Bite

Extract a service because a boundary earns independence—not because a diagram has empty boxes.

**Next chapter:** *[Software Architecture: The Incident Nobody Could Diagnose](11-software-architecture-the-incident-nobody-could-diagnose.md)*

Promotions has earned independence. The first cross-service incident will reveal whether the team can still follow one customer outcome across the new boundary.
