---
description: "A timeout is a symptom. The perimeter investigation finds the missing path or rule."
tags:
  - "aws"
  - "networking"
  - "epilogue"
---

# Networking: The Locked Kitchen

> A timeout is a symptom. The perimeter investigation finds the missing path or rule.

## The Business Goal

The new private checkout service times out while uploading receipt files. The deployment is blamed, then S3, then Lambda. Each team can name a plausible culprit; none has yet tested the path.

## The Story

The General Manager follows the order from the office. The customer finds the correct address. The food truck serves the current app bundle. The private kitchen receives the order. At the supplier gate, the receipt request stops. The kitchen has no gate to S3 and no permitted route for the chosen design—not a slow supplier and not a reason to put the kitchen on the street.

## Meet the AWS Service

This is the operational use of VPC, Route 53, CloudFront, security controls, endpoints, and NAT. They create separate failure points: name resolution, DNS answer, route, security group/NACL, endpoint policy, IAM permission, NAT egress, origin/cache behavior, and dependency health.

> **Core idea:** Diagnose connectivity in request order, and do not confuse network reachability with authorization.

## How It Works

### Follow the path

#### Evidence-led triage

1. Identify the caller, destination name, port/protocol, and time window.
2. Check DNS resolution and whether the answer is the expected public or private target.
3. Check the intended route: internet gateway, endpoint, NAT, or private service path.
4. Check security groups and NACLs at the relevant boundary.
5. Check IAM, endpoint policy, resource policy, and application authorization separately.
6. Use CloudWatch logs, metrics, and traces to distinguish timeout, DNS error, connection refusal, and `AccessDenied`.

### Recover without opening the property

#### Corrective action

For the receipt failure, add the intended S3 endpoint/private path and required permissions, validate from the private workload, then alarm on the customer-facing outcome. Do not add a public IP “just to see if it works.”

## Architectural Mapping

```text
Name → DNS answer → route → network rule → identity policy → service response
```

Each arrow is a hypothesis to test.

## When to Use It

- An application in a VPC cannot reach an AWS service, internal service, or third party
- A release changes DNS, cache, security, or network behavior

## When Not to Use It

- The evidence already shows a valid request reached the application and failed in business logic

## Painkiller

> **Problem:** Connectivity failures look alike from the caller.  
> **Pain:** Broadening access can conceal the cause and weaken security.  
> **AWS solution:** Test the complete path in sequence and repair only the missing control.

## The Masthead

### What Actually Just Happened

| In the story | In AWS | What it actually means |
| --- | --- | --- |
| Wrong sign | DNS issue | Name resolves incorrectly or not at all |
| Closed road | route/endpoint/NAT issue | No viable path to destination |
| Locked door | SG/NACL issue | Network traffic blocked |
| Rejected supplier contract | IAM/resource policy | Reachable but unauthorized |

## A Note From the Author

The real network may contain more hops, proxies, and service-specific behavior. The sequence is a developer debugging discipline, not a substitute for VPC flow logs, service logs, or the organization’s network standards and network-engineering review.

## The Last Bite

Never make the kitchen public to solve a missing private path.

**Next section:** *[AWS Applications: The Saturday Rush](../10-systems-under-pressure/00-aws-applications-the-saturday-rush.md)*

The final capstone can now make every prior section collide: identity, compute, storage, data, APIs, integrations, deployment, observability, and Byte Burger's perimeter.
