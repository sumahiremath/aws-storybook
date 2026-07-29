---
description: "Authentication identifies a caller. Authorization decides what that identified caller may do. Browser CORS decides neither."
tags:
  - "aws"
  - "apis"
  - "sdk"
  - "api-gateway"
---

# Amazon API Gateway: The Membership Desk

> Authentication identifies a caller. Authorization decides what that identified caller may do. Browser CORS decides neither.

## The Business Goal

The counter began rejecting malformed tickets. Then a customer discovered they could replace another customer's order ID in the URL and retrieve its receipt.

At the same time, the web team celebrated because their browser request passed CORS. They assumed the API was secure.

Nia closed the counter.

“A browser permission is not a membership check,” she said.

## The Story

Byte Burger installed a membership desk.

Some customers presented a signed membership token from Cognito. Some corporate callers used an AWS-signed request. A few special partners needed a custom inspector who understood a legacy identity header and returned a policy decision.

Only after identity was checked did the kitchen decide whether the caller could read **this** order.

Outside, the website host put up a sign saying which web origins could use the counter from a browser. It was useful.

It did not issue membership cards.

## Meet the AWS Service

API Gateway can use JWT/Cognito authorizers, Lambda authorizers, and IAM authorization according to the API type and integration design.

> **Core idea:** An authorizer establishes access at the API entrance; downstream code still enforces business ownership and resource-level permission where needed.

## How It Works

### JWT and Cognito

A JWT authorizer validates tokens issued by a configured identity provider. Cognito user pools commonly provide tokens for application users. The API can use claims and scopes as part of its access design.

Token validation does not prove a user may access every order. The backend must still apply tenant and ownership rules.

### Lambda Authorizer

A Lambda authorizer runs custom authorization logic. A token authorizer receives a token; a request authorizer can use configured request identity sources. Authorizer results may be cached, so the cache key and TTL must match the revocation and policy needs.

### IAM Authorization

IAM authorization uses signed AWS requests. It is useful for trusted AWS principals and service-to-service access. The caller needs IAM permission to invoke the API; API and resource policies can add restrictions.

### CORS

CORS controls whether a browser permits JavaScript from one origin to call another origin. It is not a server-side authentication or authorization mechanism. Non-browser clients can ignore CORS entirely.

## Architectural Mapping

```text
browser -> CORS browser check -> API Gateway authorizer -> backend ownership check
AWS principal -> signed request -> IAM authorization -> API Gateway -> backend
```

Identity at the front door and ownership in the kitchen are complementary controls.

## When to Use It

Use JWT/Cognito authorization for application users. Use IAM authorization for AWS principals. Use a Lambda authorizer only when standard token validation cannot express the required decision.

## When Not to Use It

Do not use API keys as user authentication. Do not rely on CORS to protect data. Do not put all tenant authorization solely in a coarse gateway rule when the business resource decision belongs in the backend.

## Painkiller

> **Problem:** Any caller who can form a URL can attempt another customer's operation.  
> **Pain:** Identity, browser behavior, and business ownership become confused.  
> **AWS solution:** Use the appropriate API Gateway authorizer at the entrance and enforce resource-level business authorization in the application.

## Knife Cut

> **CORS tells a browser which origin may call. Authorization tells the server which principal may act.**

## The Masthead

### What Actually Just Happened

|In the story|In AWS|What it actually means|
|---|---|---|
|Membership token|JWT/Cognito authorizer|Validated application-user identity|
|Custom inspector|Lambda authorizer|Custom authorization decision|
|Staff badge|IAM authorization|Signed request from AWS principal|
|Website sign|CORS|Browser cross-origin rule|

## A Note From the Author

Authorization types and exact features vary between HTTP APIs and REST APIs. Cached authorizer decisions are an operational choice: faster checks can delay the effect of changed identity state.

- [API Gateway authorizers](https://docs.aws.amazon.com/apigateway/latest/developerguide/apigateway-use-lambda-authorizer.html)
- [CORS for REST APIs](https://docs.aws.amazon.com/apigateway/latest/developerguide/how-to-cors.html)

## The Last Bite

Byte Burger let browsers approach the counter.

It still required every caller to prove what they could do once there.

**Next chapter:** *[Amazon API Gateway: The Crowd-Control Rope](04-amazon-api-gateway-the-crowd-control-rope.md)*

Authorized customers arrived faster than the kitchen could work. Nia needed a fair rope line, a partner meter, and a decision about which answers could be prepared ahead of time.
