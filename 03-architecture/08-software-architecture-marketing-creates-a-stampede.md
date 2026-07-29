---
description: "Marketing launches a national promotion. Byte Burger wants to protect ordering and payment even if recommendations, analytics, and nonessential notifications fall behind."
tags:
  - "aws"
  - "architecture"
  - "software-architecture"
  - "load-management"
---

# Software Architecture: Marketing Creates a Stampede

## The Business Goal

Marketing launches a national promotion. Byte Burger wants to protect ordering and payment even if recommendations, analytics, and nonessential notifications fall behind.

## The Story

The restaurant already has ticket rails for work that can finish later. The promotion raises a different question: which work must be preserved, which may wait, and which must be sacrificed? It keeps the essential counter open, temporarily removes expensive extras, and does not let every customer and every retry rush the same supplier at once.

## The Decision

Use backpressure, queues, concurrency limits, load shedding, caching, and graceful degradation. Classify which work is essential, deferrable, or optional before the incident.

## Decision Record

| Field | Decision |
| --- | --- |
| Valid while | The business can degrade optional experience to preserve core transactions |
| Operational tax | SLOs, queue monitoring, customer messaging, and recovery playbooks |
| Invalidation trigger | Backlog age or lost revenue exceeds the business’s recovery tolerance |
| Reversal path | Restore optional features gradually after capacity and downstream recovery are verified |

## The Last Bite

Resilience is choosing what to preserve before pressure chooses for you.

**Next chapter:** *[Software Architecture: Twenty Engineers, One Deployment](09-software-architecture-twenty-engineers-one-deployment.md)*

The promotion survives technically, but organizational growth creates the next constraint: twenty engineers can no longer coordinate every change through one informal release conversation.
