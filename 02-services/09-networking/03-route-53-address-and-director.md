---
description: "Route 53 maps memorable names to the right destination; it gives directions, not the meal itself."
tags:
  - "aws"
  - "networking"
  - "route-53"
---

# Amazon Route 53: The Address, Signpost, and Branch Director

> Route 53 maps memorable names to the right destination; it gives directions, not the meal itself.

## The Business Goal

Customers cannot be expected to remember changing addresses for every Byte Burger component. Internal services have the same problem: hard-coded private IP addresses turn routine changes into application failures.

## The Story

`byteburger.com` is the sign customers recognize. The Branch Director can point it at the food truck, the online ordering counter, or a healthy branch. Back-of-house staff use `orders.internal.byteburger` to find the ordering service without memorizing its current private street number.

## Meet the AWS Service

Amazon Route 53 is a highly available, scalable DNS service. A hosted zone holds records for a domain. Public hosted zones answer public DNS queries; private hosted zones provide DNS within associated VPCs. Records map names to answers. Alias records can point to supported AWS resources without requiring the caller to manage their changing IP addresses.

> **Core idea:** DNS resolves a name to a destination. The subsequent request travels to that destination through its own network path.

## How It Works

### The public sign

#### Public hosted zones and records

Use records such as A/AAAA, CNAME where appropriate, and AWS alias records to connect names to public entry points. Route 53 can use routing policies—including simple, weighted, latency-based, failover, geolocation, and multivalue answer—to decide which record answer to return under configured conditions.

### The staff directory

#### Private hosted zones

Associate a private hosted zone with VPCs so internal callers use stable names for internal services. DNS success does not override security groups, routes, or IAM permissions.

### The branch check

#### Health checks and failover

Route 53 health checks can inform DNS failover designs. DNS is cached, so a routing change is not an instant, per-request load balancer decision.

## Architectural Mapping

```text
Customer → byteburger.com → DNS answer → CloudFront/API/origin
Kitchen → orders.internal.byteburger → private DNS answer → internal service
```

## When to Use It

- Users or services need stable public or private names
- DNS-level routing or health-check-aware failover is appropriate

## When Not to Use It

- You need to distribute every live connection; use the appropriate load-balancing layer

## Painkiller

> **Problem:** Callers need a stable, meaningful destination name.  
> **Pain:** Hard-coded endpoints make change brittle.  
> **AWS solution:** Route 53 provides hosted DNS zones, records, and routing policies.

## Knife Cut

> Route 53 gives an address; it does not proxy the request or guarantee immediate traffic movement after a record change.

## The Masthead

### What Actually Just Happened

| In the story | In AWS | What it actually means |
| --- | --- | --- |
| Byte Burger's sign | DNS name | Stable human-readable endpoint |
| Branch Director | routing policy | DNS answer selection |
| Staff directory | private hosted zone | Internal VPC DNS |

## A Note From the Author

The “director” metaphor is intentionally limited: DNS caching and TTL influence when resolvers see a new answer. Route 53 does not inspect or carry the application request. See [Route 53 routing](https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/welcome-dns-service.html).

## The Last Bite

Names create a stable front door; the network and application still decide whether the visitor gets in.

**Next chapter:** *[Amazon CloudFront: The Lunchtime Food Truck](04-cloudfront-lunchtime-food-truck.md)*

Once customers have the right address, Byte Burger can serve common content closer to where they work.
