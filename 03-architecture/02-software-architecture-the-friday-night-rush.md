---
description: "Byte Burger wants every customer to place an order before the lunch break ends. The existing application works, but Friday evenings expose slow pages and overloaded database queries."
tags:
  - "aws"
  - "architecture"
  - "software-architecture"
  - "scalability"
  - "performance"
---

# Software Architecture: The Friday Night Rush

## The Business Goal

Byte Burger wants every customer to place an order before the lunch break ends. The existing application works, but Friday evenings expose slow pages and overloaded database queries.

## The Story

The team first adds a faster oven and tunes the order book’s indexes. They remove session state from individual workers so another worker can answer the next request. The restaurant becomes faster without splitting into separate restaurants.

## The Decision

Profile before redesigning. Improve inefficient queries, add appropriate indexes, cache safe reads, and scale stateless application workers horizontally. Put durable state in a database, cache, or session store—not in one worker’s memory.

## Decision Record

| Field | Decision |
| --- | --- |
| Valid while | One application can scale with predictable database and cache behavior |
| Operational tax | More capacity and monitoring, but still one deployment |
| Invalidation trigger | One capability creates disproportionate load or release/failure isolation becomes necessary |
| Reversal path | Keep optimizing the single application while it meets the need; do not split it merely to work around a slow query |

## The Last Bite

Scale the measured bottleneck before multiplying the number of systems that can fail.

**Next chapter:** *[Software Architecture: The Kitchen Cannot Answer the Phone](03-software-architecture-the-kitchen-cannot-answer-the-phone.md)*

The next constraint is not speed. It is work that should not keep a customer waiting.
