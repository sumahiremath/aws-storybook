---
description: "A good API rejects an impossible order at the counter and translates only when the kitchen genuinely needs a different ticket."
tags:
  - "aws"
  - "apis"
  - "sdk"
  - "api-gateway"
---

# Amazon API Gateway: The Order Contract

> A good API rejects an impossible order at the counter and translates only when the kitchen genuinely needs a different ticket.

## The Business Goal

The new counter accepted every order-shaped object.

One customer sent `quantity: "two"`. Another omitted the delivery address. A legacy partner called the new menu but still expected the old response fields. The Lambda kitchen received all of it and began accumulating validation code, response rewrites, and special cases that belonged to the entrance.

Marco had built one counter.

He had also moved the entire queue into the kitchen.

## The Story

Nia gave the cashier two tools.

The **order checker** rejected an incomplete ticket before a cook touched it. The **translator** could turn a partner's old printed form into the kitchen's current ticket—or turn a kitchen result into the customer receipt the contract promised.

For the simplest orders, the cashier passed the whole ticket directly to the station and returned its response unchanged. That was fast and transparent.

For a few old partners, translation was worth the counter work. The kitchen stayed focused on fulfilling meals.

## The Wrong Way

Validation is not authorization. A well-formed order can still come from an unauthorized caller.

Mapping is not a substitute for a stable API design. Excessive transformation can hide a confused contract and make debugging difficult. Let the backend own business validation; let the gateway reject clearly invalid HTTP requests where that improves the boundary.

## Meet the AWS Service

API Gateway REST APIs can validate request parameters, headers, and bodies against configured models. They can use mapping templates and integration responses to transform requests and responses.

> **Core idea:** Proxy integration passes a standardized request/response envelope through. Non-proxy integration lets the gateway deliberately adapt the public contract to the backend.

## How It Works

### Request Validation

Validation can require declared parameters and validate a body against a model before invoking the integration. A client receives a client error rather than consuming backend capacity for an obviously malformed request.

Validation cannot confirm that an order is affordable, that inventory exists, or that the caller may buy it. Those are business and authorization decisions.

### Proxy Integration

Lambda proxy integration sends the request context, headers, path parameters, query parameters, and body in an event. The function returns status, headers, and body in a required response structure.

This keeps the gateway thin. The function owns its translation and error representation.

### Non-Proxy Integration and Mappings

Non-proxy integration can map public request fields into a backend-specific shape and map integration outcomes to declared HTTP responses. Mapping templates are powerful when an established client contract must coexist with a different backend contract.

Treat mappings as code: version, test, and monitor them. A successful 200 response with the wrong body is still a broken order.

### Mock Integrations

A mock integration can return a configured response without a live backend. It is useful for early client development, contract demonstrations, and isolated API behavior. It is not evidence that a real kitchen integration works.

## Architectural Mapping

```text
client request
  -> validation
  -> proxy pass-through OR mapping template
  -> integration
  -> mapped or proxy response
```

## When to Use It

Use request validation for contract-level input checks. Use proxy integration when the backend can own the public shape. Use mappings when a clear compatibility or protocol boundary justifies them.

## When Not to Use It

Do not encode complex business workflows into mapping templates. Do not treat a mock integration as an integration test.

## Painkiller

> **Problem:** Bad and incompatible requests reach the kitchen unchecked.  
> **Pain:** Backends become a pile of transport-specific validation and legacy response rewrites.  
> **AWS solution:** Validate contract-level input at API Gateway and choose proxy or deliberate mapping behavior for each integration.

## Knife Cut

> **Validation checks whether a request has the required shape. Authorization checks whether that caller may make it.**

## The Masthead

### What Actually Just Happened

|In the story|In API Gateway|What it actually means|
|---|---|---|
|Order checker|Request validation|Reject missing or invalid declared input|
|Cashier passes ticket through|Proxy integration|Backend receives standardized request context|
|Ticket translator|Mapping template|Transform request or response shape|
|Practice receipt|Mock integration|Configured response without backend|

## A Note From the Author

API Gateway feature availability differs by API type. Mapping templates and request validation are REST API concepts in this story; verify the selected API type before designing around them.

- [API Gateway request validation](https://docs.aws.amazon.com/apigateway/latest/developerguide/api-gateway-method-request-validation.html)
- [Lambda proxy integration](https://docs.aws.amazon.com/apigateway/latest/developerguide/set-up-lambda-proxy-integrations.html)

## The Last Bite

The kitchen should receive a ticket it can fulfill, not every malformed scribble that reaches the building.

> **A contract is strongest where it rejects confusion early.**

**Next chapter:** *[Amazon API Gateway: The Membership Desk](03-api-gateway-the-membership-desk.md)*

The order may now be complete. Nia still must decide who is allowed to place it—and which browser is merely allowed to attempt it.

