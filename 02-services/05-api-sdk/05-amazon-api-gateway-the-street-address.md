---
description: "An API deployment is a snapshot; a stage is the named environment through which clients reach it."
tags:
  - "aws"
  - "apis"
  - "sdk"
  - "api-gateway"
---

# Amazon API Gateway: The Street Address

> An API deployment is a snapshot; a stage is the named environment through which clients reach it.

## The Business Goal

The mobile team tested against a raw staging URL. Production clients used a different hostname. A new deployment changed routes, but nobody could say which API configuration the `prod` endpoint actually represented.

Nia would not put “ask the team” on Byte Burger's sign.

## The Story

Each counter shift received a name: `dev`, `test`, and `prod`. Each name pointed to a deployed menu snapshot with its own operational settings. Customers used one street address, `api.restaurant.example`, which mapped to the production counter rather than exposing the kitchen's internal address.

For one controlled difference, the counter shift carried a small value: the approved backend alias. It was configuration, not a place to hide credentials.

## Meet the AWS Service

For REST APIs, API Gateway deployments are associated with stages. Custom domain names can map base paths to APIs and stages.

> **Core idea:** Deploy a reviewed API configuration, associate it with a named stage, and map a stable client hostname deliberately.

## How It Works

### Deployments and Stages

A deployment captures an API configuration. A stage is a named reference to a deployment and carries stage-level settings such as logging, throttling, and variables.

Stages make development, testing, and production explicit. They do not make those environments isolated if they still share the same backend, data, credentials, or account.

### Stage Variables

Stage variables are name-value pairs that can influence selected integration configuration. They are useful for controlled non-secret environment values such as an alias name.

They are not secret storage. Use a suitable secret-management service for credentials.

### Custom Domains

A custom domain gives clients a stable HTTPS hostname. API mappings connect paths on that hostname to the intended API and stage. Certificates and DNS remain part of the operational design.

## Architectural Mapping

```text
api.restaurant.example -> API mapping -> prod stage -> reviewed deployment -> integration
```

Development and test stages need separately controlled backends and credentials.

## When to Use It

Use stages for named API environments and operational settings. Use custom domains when clients need a stable, branded HTTPS address independent of default execute-api URLs.

## When Not to Use It

Do not treat stages as a substitute for separate production security and data boundaries. Do not place secrets in stage variables.

## Painkiller

> **Problem:** Clients, tests, and production use unclear or changing API endpoints.  
> **Pain:** A release cannot be tied to a named environment or stable public address.  
> **AWS solution:** Deploy API configurations to explicit stages and map a stable custom domain to the intended stage.

## Knife Cut

> **A deployment is a captured API configuration. A stage is the named environment that exposes one.**

## The Masthead

### What Actually Just Happened

|In the story|In API Gateway|What it actually means|
|---|---|---|
|Counter shift|Stage|Named API environment|
|Printed menu snapshot|Deployment|Captured API configuration|
|Street address|Custom domain|Stable client hostname|
|Counter note|Stage variable|Non-secret stage-specific value|

## A Note From the Author

Stage and custom-domain behavior differs across API Gateway API types. Domain mappings, certificates, DNS, and authorization must be tested as part of the production API path.

- [Deploy REST APIs in API Gateway](https://docs.aws.amazon.com/apigateway/latest/developerguide/how-to-deploy-api.html)

## The Last Bite

Customers should know one address.

Byte Burger should know exactly which reviewed counter waits behind it.

**Next chapter:** *[AWS SDK: The Operations Assistant](06-aws-sdk-the-operations-assistant.md)*

The application-facing counter is ready. Behind it, Marco still needs a safe way for kitchen code to call AWS services.
