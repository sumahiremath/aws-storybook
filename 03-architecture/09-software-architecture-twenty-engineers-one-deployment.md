---
description: "Twenty engineers now change ordering, loyalty, promotions, and inventory. Byte Burger wants speed without turning every release into a negotiation among strangers."
tags:
  - "aws"
  - "architecture"
  - "software-architecture"
  - "team-topology"
  - "deployment"
---

# Software Architecture: Twenty Engineers, One Deployment

## The Business Goal

Twenty engineers now change ordering, loyalty, promotions, and inventory. Byte Burger wants speed without turning every release into a negotiation among strangers.

## The Story

They first split code into many services because independent teams sound faster. Instead, every small change crosses network calls, deployment pipelines, and unclear ownership. They regroup into one modular restaurant: separate stations, explicit internal contracts, one deployable system.

## The Decision

Choose a modular monolith when domain boundaries need clarity but independent runtime deployment has not earned its cost. Define bounded contexts, ownership, internal APIs, and prohibited cross-module shortcuts. Dependency rules, module-level tests, code ownership, and build checks keep a convenient reach-in from turning the system back into one big ball of mud.

## Decision Record

| Field | Decision |
| --- | --- |
| Valid while | Modules mostly deploy together and one runtime remains operationally simpler |
| Operational tax | Boundary discipline, code review, and ownership agreements |
| Invalidation trigger | A module needs independent scale, release cadence, technology, or fault isolation with measurable business value |
| Reversal path | Merge accidental services back into modules when distributed cost exceeds benefit |

## The Last Bite

Modularity is a boundary in the codebase. Microservices are a boundary in production. They do not have to arrive together.

**Next chapter:** *[Software Architecture: The First Service Extraction](10-software-architecture-the-first-service-extraction.md)*

The modular monolith restores clear ownership without distributed-runtime cost. One capability must now prove that independent deployment, scaling, and failure isolation are worth paying for.
