---
description: "Amazon VPC gives AWS resources a logically isolated property with intentional paths in and out."
tags:
  - "aws"
  - "networking"
  - "vpc"
---

# Amazon VPC: Byte Burger's Property Line

> Amazon VPC gives AWS resources a logically isolated property with intentional paths in and out.

## The Business Goal

The team treats “inside AWS” as one safe room. It is not. A public API, a private Lambda, and a database do not need the same reachability.

## The Story

Byte Burger has customer frontage and back-of-house kitchen space. Diners may reach the entrance; they do not walk directly into the prep room. A private kitchen still needs approved routes to suppliers, but that does not make it customer-facing.

## Meet the AWS Service

An Amazon VPC is a logically isolated virtual network. Subnets divide its IP address range into placement areas. A subnet is commonly described as public when its route table provides a path to an internet gateway; private workloads do not have that direct public route. Resources also need security rules and, where relevant, public addressing to be reachable as intended.

> **Core idea:** Public and private describe network paths, not whether a workload is important or secure by itself.

## How It Works

### Frontage and back-of-house

#### Subnets and route tables

Place internet-facing components and private application/data components according to their required paths. A route table decides where traffic for a destination is sent. A private subnet can reach AWS services through an endpoint or reach the internet outbound through a NAT design; it does not need inbound public exposure.

### The kitchen's address

#### Private IP addressing

Services inside the property use private addressing and can use private DNS names. Application code should prefer stable names and service discovery over hard-coded changing addresses.

## Architectural Mapping

```text
Public caller → public entry point → private application subnet → private data service
```

The route and security controls at each handoff must permit the connection.

## When to Use It

- Lambda, ECS, EC2, or database-adjacent workloads require network isolation
- A workload must call a private dependency or use VPC-only connectivity

## When Not to Use It

- A serverless workload has no VPC-specific need; attaching it adds operational responsibility

## Painkiller

> **Problem:** Different components need different network exposure.  
> **Pain:** Treating every workload as public increases attack surface and failure modes.  
> **AWS solution:** VPC subnets and routes create deliberate paths.

## Knife Cut

> A private subnet is not automatically able to reach the internet; it needs an appropriate outbound path.

## The Masthead

### What Actually Just Happened

| In the story | In AWS | What it actually means |
| --- | --- | --- |
| Property | VPC | Isolated virtual network |
| Frontage/back room | public/private subnet path | Different reachability intent |
| Roads | route tables | Destination-based forwarding |

## A Note From the Author

This is deliberately application-facing. Real VPC design includes CIDRs, Availability Zones, routing, resilience, and organization standards. A developer must recognize the connectivity consequences, not invent the whole property plan. [Amazon VPC documentation](https://docs.aws.amazon.com/vpc/latest/userguide/what-is-amazon-vpc.html) provides the precise model.

## The Last Bite

Private means the kitchen has controlled paths—not that it has no paths.

**Next chapter:** *[Security Groups and Network ACLs: The Doors and the Fence](02-security-groups-and-nacls.md)*

Once the spaces exist, the manager must decide which doors and property gates permit traffic.
