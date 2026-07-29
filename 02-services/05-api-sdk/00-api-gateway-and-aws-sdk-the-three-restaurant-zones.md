---
description: "An API defines what may be requested. A gateway governs an application entrance. An SDK helps code call an AWS service correctly."
tags:
  - "aws"
  - "apis"
  - "sdk"
  - "api-gateway"
  - "orientation"
---

# Amazon API Gateway and AWS SDK: Byte Burger's Three Zones

> An API defines what may be requested. A gateway governs an application entrance. An SDK helps code call an AWS service correctly.

## The Business Goal

The franchise had learned where truth lived.

Then customers began walking through the kitchen door.

The mobile app knew a private Lambda name. A delivery partner sent a different order shape straight to the order database. A browser called an internal endpoint from a different website and failed before the request reached Byte Burger. Marco's backend assembled raw AWS requests by hand, copied an access key into a configuration file, and treated every timeout as proof that nothing happened.

Byte Burger had kitchens.

It had no disciplined way to accept an order.

---

## The Story

Nia divided Byte Burger into three zones.

At the **customer experience**, a mobile app showed a menu, submitted an order, and received either a receipt or a useful error.

At **front-counter operations**, Byte Burger checked who was calling, whether the order was complete, how busy the counter was, and which kitchen station should receive it.

At **kitchen operations**, systems stored an object, calculated a price, retrieved a record, or started work.

Marco also needed a trusted assistant behind the counter. When kitchen code called DynamoDB or S3, the assistant selected the Region, found temporary credentials, signed the request, sent it, retried the right transient failures, and returned a response the code could understand.

Nia wrote the boundary on the menu board:

```text
client -> application API front counter -> application kitchen
application code -> AWS SDK -> AWS service endpoint -> AWS kitchen
```

The two arrows were related.

They were not the same door.

---

## The Wrong Way

An API is not merely a URL.

It is a contract: which operation exists, which details are required, how identity is presented, which outcomes are possible, and what a caller should do when one occurs.

An SDK is not an authorization system. It makes AWS API calls easier, but the credentials it finds still need IAM permission for the requested action and resource.

API Gateway is not a replacement for every AWS service endpoint. It is a managed front door for APIs you expose to your own clients.

---

## Meet the AWS Services

> **Core idea:** API Gateway manages how clients reach an application. AWS SDKs manage how application code reaches AWS services.

- **Amazon API Gateway** can expose REST APIs and HTTP APIs, route requests to integrations, enforce selected front-door controls, and return configured responses.
- **AWS SDKs** provide language-specific service clients that handle request construction, authentication mechanisms, retries, pagination, and response parsing according to each SDK and service contract.

AWS still owns the internal implementation of a managed service. The application owns its API design, authorization model, input validation, error handling, idempotency, observability, and cost-aware use of calls.

---

## How It Works

### Customer Experience

The client sees an API contract. It sends a request and must be prepared for success, validation failure, authentication failure, throttling, or an internal error.

### Front-Counter Operations

API Gateway can match a route and method, validate parts of a request, authorize a caller, transform the request for an integration, and apply quotas or throttles. It does not make the downstream business operation correct.

### Kitchen Operations

The integration target performs application work. It might be Lambda, an HTTP endpoint, an AWS service integration, or another supported target.

When that code needs AWS, it uses an SDK client. The SDK discovers credentials, signs the call when appropriate, selects a service endpoint in a Region, and returns a result or a final error.

---

## Architectural Mapping

```text
mobile app -> API Gateway -> Lambda -> AWS SDK -> DynamoDB
                    |                         -> S3
             auth / validation
             throttle / route
```

Each arrow needs its own permissions and failure behavior. A caller authorized to invoke an API is not automatically authorized to read a DynamoDB table; the Lambda execution role makes that downstream call.

## When to Use It

Use API Gateway when an application needs a managed HTTP front door with routing and policy controls. Use an AWS SDK when code needs to call an AWS service API rather than manually implement the protocol.

## When Not to Use It

Do not put API Gateway between every internal operation without a client-facing reason. Do not handcraft signed AWS requests when an SDK supports the language and service.

## Painkiller

> **Problem:** Clients and backend code call private systems and AWS services through ad hoc interfaces.  
> **Pain:** Contracts drift, credentials leak, failures are misread, and kitchens become public entrances.  
> **AWS solution:** Define an application API behind API Gateway and use AWS SDK service clients for controlled programmatic AWS access.

## Knife Cut

> **API Gateway exposes your application to its clients. The AWS SDK lets your application call AWS.**

## The Masthead

### What Actually Just Happened

|In the story|In AWS|What it actually means|
|---|---|---|
|Menu and order form|API contract|Operations, inputs, outcomes, and error behavior|
|Front counter|API Gateway|Managed application API entry point|
|Kitchen station|Integration target|Backend that performs application work|
|Operations assistant|AWS SDK client|Language library for AWS service calls|
|AWS kitchen|AWS service|Managed implementation behind the service API|

## A Note From the Author

Byte Burger makes one request look linear. Real applications can authenticate at several layers, make asynchronous work durable through queues, and complete one customer action through several services.

The SDK's client is usually application code. A browser or mobile app might call an application API directly without an AWS SDK; it should not receive broad AWS credentials merely because it needs to upload one file.

- [Amazon API Gateway documentation](https://docs.aws.amazon.com/apigateway/)
- [AWS SDKs and Tools Reference Guide](https://docs.aws.amazon.com/sdkref/latest/guide/)

## The Last Bite

Byte Burger did not become safer by adding more doors.

It became safer when every door stated who could use it, what it accepted, and where it led.

> **A useful interface is a boundary with a promise.**

**Next chapter:** *[Amazon API Gateway: The Customer Front](01-api-gateway-the-customer-front.md)*

The front counter now exists. Nia must decide whether it needs a full-service menu or a faster, leaner express counter.

