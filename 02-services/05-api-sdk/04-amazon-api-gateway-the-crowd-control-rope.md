---
description: "A front counter protects the kitchen by limiting demand; it does not create capacity or identify a user by itself."
tags:
  - "aws"
  - "apis"
  - "sdk"
  - "api-gateway"
---

# Amazon API Gateway: The Crowd-Control Rope

> A front counter protects the kitchen by limiting demand; it does not create capacity or identify a user by itself.

## The Business Goal

A promotion made the menu popular. One partner's buggy app retried every failed request immediately. It consumed the counter, then the kitchen, then the database capacity meant for everyone else.

The team proposed an API key on every request. “Now we know who they are,” someone said.

Nia shook her head. “You know whose meter is moving. That is not the same thing.”

## The Story

Nia installed a crowd-control rope. It allowed a sustainable flow and returned a clear “try later” response when the line exceeded it. Partners received cards tied to usage plans, so their traffic could be metered and limited by agreement. The counter also kept a short-lived board of frequently requested menu answers.

The board helped read-heavy menu requests. It did not help a customer asking the kitchen to create an order, and it could briefly show an old price if Nia chose the wrong cache policy.

## Meet the AWS Service

API Gateway can apply throttling, quotas, API keys and usage plans for supported API types and configurations. REST APIs can use API caching.

> **Core idea:** Throttling controls rate, quota controls accumulated allowance, API keys associate usage with a consumer plan, and caching trades freshness for fewer backend calls.

## How It Works

### Throttling and Quotas

Throttling limits request rate and burst behavior. When a limit is exceeded, callers can receive throttling responses and should back off rather than retry in a synchronized storm.

A quota limits how many requests a usage-plan consumer may make during a configured period. Neither setting is a perfect hard security boundary; design backend capacity and abuse protection independently.

### API Keys and Usage Plans

An API key identifies an API consumer for metering and plan association. Usage plans associate keys with throttles and quotas.

An API key is not a password, a JWT, or authorization. Do not use it as the sole protection for sensitive APIs.

### Caching

API Gateway caching can serve eligible REST API responses from a cache rather than invoking the integration. Cache keys and TTL determine reuse. Invalidate or design around stale responses when data changes.

Caching can lower latency and backend cost. It can also return old data and create a cache-miss surge after expiry.

## Architectural Mapping

```text
partner request -> API key / usage plan -> throttle -> cache miss -> backend
                                            \-> cache hit  -> response
```

The backend remains responsible for its own concurrency, validation, and durable state.

## When to Use It

Use throttles and plans to protect shared front doors and manage partner consumption. Use caching for safely cacheable, read-heavy responses where bounded staleness is acceptable.

## When Not to Use It

Do not cache personalized or rapidly changing answers without a correct cache key. Do not use a usage plan as the only tenant-isolation control.

## Painkiller

> **Problem:** One noisy client can overwhelm a shared application entrance.  
> **Pain:** Healthy customers lose access and aggressive retries deepen the overload.  
> **AWS solution:** Apply gateway throttles and consumer plans, teach clients to back off, and cache only responses that can safely be reused.

## Knife Cut

> **An API key meters a consumer. It does not establish a trustworthy human or application identity.**

## The Masthead

### What Actually Just Happened

|In the story|In API Gateway|What it actually means|
|---|---|---|
|Crowd-control rope|Throttle|Rate and burst protection|
|Partner allowance|Quota|Requests allowed over a period|
|Partner card|API key|Consumer identifier for usage management|
|Partner agreement|Usage plan|Key-associated throttle and quota|
|Prepared menu board|API cache|Stored response reused before backend|

## A Note From the Author

Limits are not a substitute for WAF, authentication, backend concurrency, or cost monitoring. Exact throttling behavior and cache support vary by API type and account configuration.

- [API Gateway usage plans and API keys](https://docs.aws.amazon.com/apigateway/latest/developerguide/api-gateway-api-usage-plans.html)
- [API Gateway caching](https://docs.aws.amazon.com/apigateway/latest/developerguide/api-gateway-caching.html)

## The Last Bite

The rope did not make the kitchen faster.

It made one broken ordering app less able to take dinner away from everyone else.

**Next chapter:** *[Amazon API Gateway: The Street Address](05-amazon-api-gateway-the-street-address.md)*

The counter could now govern a crowd. It still needed a stable street address and separate practice shifts before a new menu reached production.
