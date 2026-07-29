---
description: "Byte Burger opens a second store. Customers expect one loyalty account, headquarters needs a unified menu, and each store still needs local hours, inventory, and promotions."
tags:
  - "aws"
  - "architecture"
  - "software-architecture"
  - "multi-location"
---

# Software Architecture: Store Number Two

## The Business Goal

Byte Burger opens a second store. Customers expect one loyalty account, headquarters needs a unified menu, and each store still needs local hours, inventory, and promotions.

## The Story

Copying the first restaurant’s order book feels fast. Soon two books disagree about a customer’s points and which menu is current. Headquarters gives some facts a shared owner and treats store-specific settings as configuration rather than a forked application.

## The Decision

Define data ownership before replicating systems. Separate tenant/store configuration from shared business truth. Use read scaling or caching for repeated reads, but state which system owns writes and how stale copies are reconciled.

## Decision Record

| Field | Decision |
| --- | --- |
| Valid while | Stores share a common business model with controlled local variation |
| Operational tax | Configuration lifecycle, tenancy boundaries, cache freshness, and data-access controls |
| Invalidation trigger | Franchise or regional rules require stronger isolation or independent ownership |
| Reversal path | Keep shared truth and controlled configuration until a tenant's risk or contract earns stronger isolation |

## The Last Bite

The second store does not require a second truth. It requires an explicit owner for every kind of truth.

**Next chapter:** *[Software Architecture: Franchisees Want Their Own Rules](05-software-architecture-franchisees-want-their-own-rules.md)*

The next question is how one platform can honor local rules without letting one franchise see, change, or fork another franchise’s business.
