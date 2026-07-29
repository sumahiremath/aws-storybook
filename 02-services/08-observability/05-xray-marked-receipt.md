---
description: "After the first incident, every team wants to put every detail on every trace. Soon the Store Manager has a mountain of receipts and no dependable way to find “checkout requests from the mobile app that hit the new function version.”"
tags:
  - "aws"
  - "observability"
  - "operations"
  - "x-ray"
---

# AWS X-Ray: The Marked Receipt

## The Business Goal

After the first incident, every team wants to put every detail on every trace. Soon the Store Manager has a mountain of receipts and no dependable way to find “checkout requests from the mobile app that hit the new function version.”

## The Story

She adds a few stamped labels to each selected receipt: `route=checkout`, `channel=mobile`, and `release=2026-07-27`. Those stamps can be sorted and searched. She keeps a richer note—safe dependency details and diagnostic context—inside the receipt, where it is visible when opened but not part of the fast index. She samples normal traffic while ensuring unusual errors remain available for investigation.

## Meet the AWS Service

X-Ray **annotations** are indexed key-value pairs used in filter expressions and grouping. **Metadata** can hold richer values, including objects and lists, but is not indexed for search. Sampling selects a representative subset of requests for tracing, balancing visibility, overhead, and cost. Errors, faults, and throttles should be treated as signals to investigate rather than hidden by a sample strategy.

## How It Works

Use annotations for a small, stable set of questions you expect to ask: route, tenant tier where appropriate, deployment version, or feature flag. Use metadata for additional diagnostic detail that helps when the trace is already open. Never put secrets or sensitive customer data in either just because the trace feels internal.

Sampling is a design choice. A high-volume, healthy path may need a representative rate; a new release, rare workflow, or error condition may justify more visibility. When an incident happens, verify the telemetry policy before concluding that “nothing happened.” An absent trace can mean it was not sampled or instrumented.

## Architectural Mapping

| Byte Burger | X-Ray |
| --- | --- |
| Searchable stamp | annotation |
| Detailed note inside a receipt | metadata |
| Inspecting selected receipts | sampling |

## Painkiller

Make telemetry answer repeatable questions. A few intentional labels beat unlimited, unsearchable detail.

## Knife Cut

Annotations are searchable, not a license to index customer identity or high-cardinality values. Metadata is not searchable, so it cannot power the first cut of an incident.

## The Masthead

The trace points to a rollout-related station. Now the team must determine whether an AWS configuration change created the condition.

## A Note From the Author

AWS distinguishes [annotations and metadata](https://docs.aws.amazon.com/xray/latest/devguide/xray-concepts.html) and explains X-Ray's [sampling behavior](https://docs.aws.amazon.com/xray/latest/devguide/xray-concepts.html).

## The Last Bite

The Store Manager can say where the request waited. The Security Guard can say who changed the building.

**Next chapter:** *[AWS CloudTrail: The Security Guard](06-cloudtrail-security-guard.md)*

Next: CloudTrail records the AWS activity behind the service.
