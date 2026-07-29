---
description: "Byte Burger needs an online menu, order entry, payment recording, and kitchen tickets. Three engineers need to release changes quickly and understand the whole system."
tags:
  - "aws"
  - "architecture"
  - "software-architecture"
  - "monolith"
---

# Software Architecture: One Restaurant, One Application

## The Business Goal

Byte Burger needs an online menu, order entry, payment recording, and kitchen tickets. Three engineers need to release changes quickly and understand the whole system.

## The Story

They build one restaurant: one kitchen, one order book, one deployment. The order code, menu code, and loyalty code are separate rooms, but one team can change them together. Friday’s first rush succeeds.

## The Decision

Choose a monolith with clear internal modules and a relational source of truth. It minimizes network calls, deployment coordination, and operational surfaces while the business is small and changes together.

## Decision Record

| Field | Decision |
| --- | --- |
| Valid while | One team owns most changes and one deployment is safe |
| Operational tax | Shared release risk and shared runtime scaling |
| Invalidation trigger | Independent teams or workloads need different release, scale, or failure boundaries |
| Reversal path | Keep modules clear; remove accidental seams rather than creating services merely to look modern |

## The Last Bite

A monolith is not an absence of architecture. It is an architectural choice to keep related work close while that remains cheaper and safer.

**Next chapter:** *[Software Architecture: The Friday Night Rush](02-software-architecture-the-friday-night-rush.md)*

The first problem is not service boundaries. It is whether the one restaurant can keep up.
