---
description: "Franchisees need local menus, taxes, promotions, staff, and reporting, while Byte Burger must protect tenant data and preserve a common platform."
tags:
  - "aws"
  - "architecture"
  - "software-architecture"
  - "multi-tenancy"
---

# Software Architecture: Franchisees Want Their Own Rules

## The Business Goal

Franchisees need local menus, taxes, promotions, staff, and reporting, while Byte Burger must protect tenant data and preserve a common platform.

## The Story

One franchise starts a loyalty campaign using an old customer-field format while another sends a new menu format to the same delivery partner. A forked restaurant for each franchise would make upgrades impossible; unrestricted access to every record would destroy trust. Byte Burger chooses explicit tenant boundaries and configuration that changes behavior without rewriting the whole kitchen.

## The Decision

Model tenancy, isolation, configuration, authorization, and data access deliberately. Select the isolation strength from risk and contract, not from a preference for one database pattern.

## Decision Record

| Field | Decision |
| --- | --- |
| Valid while | Tenant variation fits a governed shared platform |
| Operational tax | Isolation testing, configuration lifecycle, support, and compliance evidence |
| Invalidation trigger | A tenant requires materially different data residency, performance, or compliance guarantees |
| Reversal path | Move a tenant to stronger isolation when its residency, performance, or compliance contract outgrows the shared platform |

## The Last Bite

Multi-tenancy is not “one database with a tenant ID.” It is an explicit promise about isolation and variation.

**Next chapter:** *[Software Architecture: Delivery Apps Arrive](06-software-architecture-delivery-apps-arrive.md)*

The platform can now vary safely by franchise. The next pressure comes from outside the company, where delivery partners need a stable entrance without receiving a key to the kitchen.
