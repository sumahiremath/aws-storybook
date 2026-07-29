---
description: "API Gateway gives an application one managed HTTP entrance instead of teaching every client the kitchen layout."
tags:
  - "aws"
  - "apis"
  - "sdk"
  - "api-gateway"
---

# Amazon API Gateway: The Customer Front

> API Gateway gives an application one managed HTTP entrance instead of teaching every client the kitchen layout.

## The Business Goal

Every client had memorized a different kitchen door.

The mobile app called one Lambda. The web app called another through an internal hostname. A partner sent delivery orders to a third URL. When the payment service moved, every client needed a release.

Nia did not need more kitchens.

She needed one customer front that could keep the menu stable while the kitchen changed behind it.

---

## The Story

Nia opened two counters.

The **full-service counter** could handle a detailed menu, older ordering conventions, transformations, and features that some large partners required.

The **express counter** accepted simple modern orders with less ceremony and lower operational overhead.

Both counters read the same intent:

```text
POST /orders
GET  /orders/{orderId}
```

The customer did not need to know whether an order reached Lambda, a private service, or a workflow. The counter selected the right integration.

---

## The Wrong Way

A route is not a backend implementation detail. If clients call a Lambda function name or private host directly, the backend becomes the public contract. Replacing it becomes a breaking client change.

Conversely, a gateway does not erase backend design. A slow, unavailable, or incorrectly authorized integration still produces a poor API.

## Meet the AWS Service

**Amazon API Gateway** creates, publishes, maintains, monitors, and secures APIs at scale.

> **Core idea:** A route and method express what a client may ask for; an integration expresses where API Gateway sends that request.

API Gateway offers REST APIs and HTTP APIs. HTTP APIs are designed for many straightforward HTTP workloads with a simpler feature set and lower cost. REST APIs provide a broader feature set, including capabilities such as request validation, API keys and usage plans, and mapping templates.

## How It Works

### Menu Sections and Actions

A resource or route identifies a path such as `/orders` or `/orders/{orderId}`. An HTTP method states the action: `GET` retrieves, `POST` creates, `PUT` replaces or updates according to the contract, and `DELETE` removes or requests deletion.

The method alone does not guarantee business semantics. The API designer documents what the operation does, which status codes it returns, and whether a repeated request is safe.

### Integration

API Gateway forwards a matched request to an integration. Common designs use Lambda proxy integration, HTTP backends, or supported AWS service integrations.

With a proxy integration, the backend receives a structured request envelope and returns a response shape API Gateway can pass through. With non-proxy integration, API Gateway can transform the request and response deliberately; the next chapter explains why that translation is sometimes necessary.

### REST API or HTTP API

Choose **HTTP API** when a modern HTTP API needs common features such as routes, JWT authorization, CORS, and integrations without REST API-specific capabilities.

Choose **REST API** when the required contract needs REST API features such as request validation, usage plans and API keys, or detailed mapping and integration-response control.

Selection is based on required behavior, not on the word “REST” sounding more complete.

## Architectural Mapping

```text
client
  | POST /orders
  v
API Gateway route and method
  |
  v
Lambda / HTTP backend / AWS integration
```

API Gateway needs permission to invoke or reach its integration. The downstream runtime needs separate permission for the AWS resources it uses.

## When to Use It

Use API Gateway when clients need a stable HTTP contract independent of backend implementation, especially when routing, authorization, throttling, stages, or a custom domain belong at the entrance.

## When Not to Use It

Do not place it in front of a one-off private call merely to make the architecture look uniform. Do not select an API type before identifying required authorization, transformation, and consumer-management features.

## Painkiller

> **Problem:** Every client calls a private backend implementation directly.  
> **Pain:** Backend changes become client migrations and each caller gets inconsistent controls.  
> **AWS solution:** Expose stable routes and methods through API Gateway, then route them to the appropriate integration.

## Knife Cut

> **A route tells the client what may be ordered. An integration tells API Gateway where to send the order.**

## The Masthead

### What Actually Just Happened

|In the story|In API Gateway|What it actually means|
|---|---|---|
|Full-service counter|REST API|Feature-rich API type|
|Express counter|HTTP API|Simpler modern API type|
|Menu section|Route/resource|Addressable API path|
|Order action|HTTP method|Operation on a path|
|Kitchen handoff|Integration|Configured backend target|

## A Note From the Author

REST API and HTTP API are AWS product types, not a judgment of whether an API follows every REST design convention. Feature availability changes, so select against the current API Gateway documentation rather than a remembered comparison chart.

- [Choose between REST APIs and HTTP APIs](https://docs.aws.amazon.com/apigateway/latest/developerguide/http-api-vs-rest.html)

## The Last Bite

The counter did not cook the meal.

It made the menu stable enough that the kitchen could change without every customer relearning how to order.

> **The front door is part of the application contract.**

**Next chapter:** *[Amazon API Gateway: The Order Contract](02-api-gateway-the-order-contract.md)*

A stable door still accepts terrible orders. Nia now needs a checker and, sometimes, a translator before the ticket reaches the kitchen.

