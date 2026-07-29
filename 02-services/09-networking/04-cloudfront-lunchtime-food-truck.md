---
description: "CloudFront serves cacheable content from edge locations near viewers and asks the origin only when it needs to."
tags:
  - "aws"
  - "networking"
  - "cloudfront"
---

# Amazon CloudFront: The Lunchtime Food Truck

> CloudFront serves cacheable content from edge locations near viewers and asks the origin only when it needs to.

## The Business Goal

Every office worker requests the same menu images, JavaScript, and promotion details from Byte Burger's central kitchen at noon. The origin does repetitive work and distant customers wait for it.

## The Story

A Byte Burger food truck parks near the office park before lunch. It carries the current menu materials and other safely reusable items. When an employee asks for something already stocked, the truck serves it immediately. When it does not have a valid copy, it asks Byte Burger's central kitchen—the origin—for one. It does not physically drive back and forth; it is a nearby service point with a fast connection to the kitchen.

## Meet the AWS Service

Amazon CloudFront is a content delivery network that uses edge locations to deliver content. A distribution has one or more origins, cache behaviors that determine how paths are handled, and cache-control/TTL behavior that governs freshness. CloudFront can front S3, a load balancer, API Gateway, or a custom origin. Signed URLs or signed cookies can restrict access to selected private content.

> **Core idea:** An edge cache reduces repeat origin work, but freshness and authorization remain deliberate design choices.

## How It Works

### The stock on the truck

#### Cache key and TTL

CloudFront can return a cached object when the request matches the cache key and the cached response remains valid under its policy. TTL and origin cache headers influence how long it can be used. A cache hit avoids an origin trip; a miss or expired object causes an origin fetch.

### Replacing a menu card

#### Invalidation and versioning

Invalidation removes cached objects from edge caches, but versioned asset names often make deployment behavior clearer and more cache-friendly. Choose the approach that matches the release and freshness requirement.

### Private catering list

#### Signed access

Signed URLs or cookies can limit access to CloudFront content. They complement—not replace—origin permissions and application authorization.

## Architectural Mapping

```text
Viewer → CloudFront edge → cache hit → response
                       └→ cache miss → origin → response/cache
```

## When to Use It

- Static assets, downloads, or safely cacheable responses have distributed viewers
- The application needs lower origin load and lower viewer latency

## When Not to Use It

- Every response must reflect a just-completed write and cannot tolerate caching behavior

## Painkiller

> **Problem:** Repeated distant requests burden the central origin.  
> **Pain:** Latency and origin work grow with every identical request.  
> **AWS solution:** CloudFront caches appropriate content close to viewers.

## Knife Cut

> CloudFront is a cache in front of an origin, not a replacement database or an automatic guarantee of fresh content.

## The Masthead

### What Actually Just Happened

| In the story | In AWS | What it actually means |
| --- | --- | --- |
| Food truck | edge location | Nearby content-delivery point |
| Central kitchen | origin | Source when cache cannot answer |
| Current menu window | TTL/cache policy | Freshness rule |

## A Note From the Author

The truck is only a memory aid: CloudFront is a global network service, not a separate application server you operate. Cache-key, origin, security, and invalidation choices remain customer responsibilities. [CloudFront documentation](https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/Introduction.html) describes the current features.

## The Last Bite

Serve reusable things near the customer; keep the source of truth intentional.

**Next chapter:** *[VPC Endpoints and NAT Gateway: The Private Supplier Gate and Service Exit](05-vpc-endpoints-and-nat.md)*

The kitchen itself still needs safe paths to AWS suppliers and selected third parties.
