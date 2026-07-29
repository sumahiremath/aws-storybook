---
description: "Security groups control allowed traffic at a resource boundary; network ACLs control traffic at a subnet boundary."
tags:
  - "aws"
  - "networking"
  - "security-groups"
  - "network-acls"
---

# Security Groups and Network ACLs: The Doors and the Fence

> Security groups control allowed traffic at a resource boundary; network ACLs control traffic at a subnet boundary.

## The Business Goal

The checkout service resolves the right internal name but cannot connect. One engineer changes every network rule to “allow all.” The request succeeds, but the property is now less safe and the team learned nothing.

## The Story

Each kitchen station has a door list: the payment station may accept a handoff from checkout, but not directly from the public street. The property also has a fence gate governing whole areas. The two controls overlap in purpose but stand in different places.

## Meet the AWS Service

Security groups are stateful virtual firewalls associated with resources such as ENIs; they define allowed inbound and outbound traffic. Network ACLs are stateless rules associated with subnets and evaluate inbound and outbound traffic separately. Both are part of connectivity, alongside routes and application-level authorization.

> **Core idea:** Open only the path the application needs, at the boundary where that rule belongs.

## How It Works

### Station doors

#### Security groups

Use security-group references where supported to express “this service may call that service” rather than relying on a broad address range. Because they are stateful, return traffic for an allowed connection is handled as part of that connection.

### Property fence

#### Network ACLs

NACLs apply at the subnet. Their rules are ordered and stateless, so return traffic requires appropriate rules in the reverse direction. They are useful as broader subnet controls, but an application usually benefits from precise resource-level security groups.

## Architectural Mapping

| In the story | In AWS | What it actually means |
| --- | --- | --- |
| Door list at a station | security group | Allowed resource traffic |
| Property fence gate | network ACL | Subnet-level traffic rules |
| Approved return handoff | stateful SG behavior | Return traffic for allowed flow |

## When to Use It

- A service must accept traffic only from a defined caller or network
- A subnet needs a broader inbound/outbound boundary rule

## When Not to Use It

- You need user authorization; use IAM, Cognito, API authorization, or application controls

## Painkiller

> **Problem:** A path needs to be permitted without exposing unrelated paths.  
> **Pain:** Broad rules hide the cause and increase risk.  
> **AWS solution:** Security groups and NACLs constrain traffic at different layers.

## Knife Cut

> Security groups are stateful and resource-associated; NACLs are stateless and subnet-associated.

## The Masthead

### What Actually Just Happened

| In the story | In AWS | What it actually means |
| --- | --- | --- |
| Kitchen door | SG rule | Resource-level allowed flow |
| Fence | NACL | Subnet-level rule set |
| Staff badge | IAM/app auth | A separate identity decision |

## A Note From the Author

The fence analogy simplifies protocol and ephemeral-port details. Do not solve an authorization problem with a network rule. [AWS compares security groups and NACLs](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-security-groups.html) in its VPC documentation.

## The Last Bite

The right rule is narrow enough to explain why the connection is allowed.

**Next chapter:** *[Amazon Route 53: The Address, Signpost, and Branch Director](03-route-53-address-and-director.md)*

Secure paths are not useful if callers do not know the stable name for the right entrance.
