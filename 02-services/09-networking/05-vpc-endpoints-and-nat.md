---
description: "VPC endpoints keep supported AWS service traffic on private paths; NAT gives private workloads controlled outbound internet access."
tags:
  - "aws"
  - "networking"
  - "vpc"
  - "nat-gateway"
---

# VPC Endpoints and NAT Gateway: The Private Supplier Gate and Service Exit

> VPC endpoints keep supported AWS service traffic on private paths; NAT gives private workloads controlled outbound internet access.

## The Business Goal

The checkout Lambda is placed in a private subnet. It now cannot upload a receipt to S3 or call an external payment API. Someone proposes assigning it a public address. That solves the wrong problem.

## The Story

Byte Burger builds two different doors. The **private supplier gate** connects the kitchen directly to trusted AWS suppliers across the internal delivery network. The **one-way service exit** lets staff call an outside supplier, but outside visitors cannot use that exit to walk into the kitchen.

## Meet the AWS Service

A VPC endpoint provides private connectivity from a VPC to supported AWS services or endpoint services without requiring traffic to traverse the public internet. Gateway endpoints are available for specific services such as S3 and DynamoDB; interface endpoints use private network interfaces and PrivateLink for many services. A NAT Gateway enables instances in a private subnet to initiate outbound connections to the internet or other AWS services while not accepting unsolicited inbound connections from the internet.

> **Core idea:** Private AWS access and outbound internet access are distinct requirements with different paths.

## How It Works

### Private supplier gate

#### VPC endpoints

Use an endpoint when a private workload needs a supported AWS service and the goal is private connectivity. Endpoint policy, IAM policy, DNS configuration, security groups for interface endpoints, and route-table association for gateway endpoints still matter.

### One-way exit

#### NAT Gateway

A private subnet routes outbound traffic to a NAT Gateway placed in a public subnet with an internet gateway path. The workload can initiate the request; the internet cannot initiate a new request back to it through NAT.

## Architectural Mapping

| In the story | In AWS | What it actually means |
| --- | --- | --- |
| Private supplier gate | VPC endpoint | Private service connectivity |
| One-way service exit | NAT Gateway | Outbound internet path |
| Supplier contract | IAM/endpoint policy | Permission still required |

## When to Use It

- A private workload needs S3, DynamoDB, or another supported AWS service
- A private workload must call a third-party public API without inbound public exposure

## When Not to Use It

- You assume a VPC endpoint lets a workload call arbitrary internet destinations
- You assume NAT grants IAM permission to an AWS service

## Painkiller

> **Problem:** Private workloads need dependencies without becoming public.  
> **Pain:** A missing path looks like an application timeout and tempts unsafe exposure.  
> **AWS solution:** Endpoints provide private AWS paths; NAT provides controlled outbound internet paths.

## Knife Cut

> An endpoint is a private route to supported AWS services; NAT is an outbound path to public destinations. Neither substitutes for permissions.

## The Masthead

### What Actually Just Happened

| In the story | In AWS | What it actually means |
| --- | --- | --- |
| Private dock | VPC endpoint | Private AWS-service path |
| One-way exit | NAT Gateway | Egress without public ingress |
| Supplier authorization | IAM | Identity permission decision |

## A Note From the Author

Endpoint types, DNS behavior, policies, availability design, and cost differ. The analogy deliberately compresses those choices into two path types. See [VPC endpoints](https://docs.aws.amazon.com/vpc/latest/privatelink/vpc-endpoints.html) and [NAT gateways](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-nat-gateway.html).

## The Last Bite

Keep the kitchen private, then give it only the exits and supplier gates it truly needs.

**Next chapter:** *[Networking: The Locked Kitchen](06-networking-locked-kitchen.md)*

The last chapter uses the perimeter as an incident map instead of a list of services.
