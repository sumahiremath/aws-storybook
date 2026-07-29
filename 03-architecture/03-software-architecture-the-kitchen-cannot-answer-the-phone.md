---
description: "Byte Burger wants to accept an order immediately even when sending receipts, updating loyalty points, and notifying delivery partners take longer."
tags:
  - "aws"
  - "architecture"
  - "software-architecture"
  - "asynchronous-messaging"
  - "idempotency"
---

# Software Architecture: The Kitchen Cannot Answer the Phone

## The Business Goal

Byte Burger wants to accept an order immediately even when sending receipts, updating loyalty points, and notifying delivery partners take longer.

## The Story

The counter stops phoning every department while the customer waits. It writes the accepted order to the book, puts follow-up work on a ticket rail, and gives the customer an order number. The kitchen can now continue when the counter is free. But a runner can update loyalty points, crash before marking the ticket complete, and receive the same ticket again. The second attempt must not award the customer twice.

## The Decision

Separate synchronous acceptance from asynchronous completion. Use durable queues or events when work can happen later, preserve business status, and make consumers idempotent because messages and retries can repeat.

## Decision Record

| Field | Decision |
| --- | --- |
| Valid while | The customer can accept an honest “received” state and work is durable |
| Operational tax | Eventual completion, status tracking, retries, DLQs, and reconciliation |
| Invalidation trigger | The caller truly requires an authoritative answer before proceeding |
| Reversal path | Keep only deferrable work on the ticket rail; return authoritative, immediate decisions to the synchronous path |

## The Last Bite

Asynchronous does not mean unfinished work disappears. It means completion has its own contract.

**Next chapter:** *[Software Architecture: Store Number Two](04-software-architecture-store-number-two.md)*

Growth now creates a shared-data problem rather than a single-restaurant throughput problem.
