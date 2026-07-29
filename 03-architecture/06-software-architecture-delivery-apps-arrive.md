---
description: "Byte Burger wants delivery partners to place orders safely without exposing its database or making a partner timeout create duplicate meals."
tags:
  - "aws"
  - "architecture"
  - "software-architecture"
  - "api-integration"
---

# Software Architecture: Delivery Apps Arrive

## The Business Goal

Byte Burger wants delivery partners to place orders safely without exposing its database or making a partner timeout create duplicate meals.

## The Story

At first, a delivery partner is given an internal path to submit orders. Its request times out just after the order is committed, so it sends the order again. Two meals reach the kitchen and support has to decide which charge and which driver to undo. The team replaces the kitchen key with a clear ordering counter. Each order has a stable reference, and a late reply no longer makes a partner guess whether to send the same order again.

## The Decision

Expose an API contract, authenticate callers, validate requests, rate-limit abusive traffic, and make create operations idempotent. Use webhooks or asynchronous status for later outcomes.

## Decision Record

| Field | Decision |
| --- | --- |
| Valid while | External partners need a stable contract and bounded integration surface |
| Operational tax | Versioning, quotas, authentication, retries, support, and observability |
| Invalidation trigger | Partner-specific logic begins contaminating the core ordering domain |
| Reversal path | Remove a departing partner's edge adapter without changing the core ordering domain |

## The Last Bite

An API is a promise about behavior under success, failure, and repetition—not merely an endpoint.

**Next chapter:** *[Software Architecture: The Menu Says Available, the Kitchen Says Sold Out](07-software-architecture-menu-and-kitchen-disagree.md)*

The partner can now submit an order safely. But a fast external contract is still untrustworthy if the menu and the authoritative inventory disagree.
