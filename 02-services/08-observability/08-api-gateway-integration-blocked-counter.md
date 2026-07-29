---
description: "The customer sees a failed order at the counter. The kitchen insists it never received one. The team calls everything a “500,” which hides the difference between an invalid order, a rejected identity, a throttled counter, and a broken backend integration."
tags:
  - "aws"
  - "observability"
  - "operations"
  - "api-gateway"
---

# Amazon API Gateway and Integrations: The Blocked Counter

## The Business Goal

The customer sees a failed order at the counter. The kitchen insists it never received one. The team calls everything a “500,” which hides the difference between an invalid order, a rejected identity, a throttled counter, and a broken backend integration.

## The Story

At Byte Burger, the counter clerk has distinct responses. A missing meal choice is sent back for correction. An expired staff badge is refused. A counter at capacity asks the customer to try later. A valid order handed to a kitchen that fails becomes a different incident entirely.

The Operations Manager puts these outcomes on separate lines of the dashboard. The Store Manager follows a failing request to see whether API Gateway spent time at the counter or in the Lambda integration. The Security Guard is called only if a recent authorizer, route, permission, or deployment change is suspected.

## Meet the AWS Service

API Gateway exposes an API contract and routes requests to integrations such as Lambda or other AWS services. Client-side 4xx responses and service-side 5xx responses have different likely causes. Gateway and integration latency metrics help separate time at the managed front door from time spent waiting for the backend. Authentication, authorization, request validation, throttling, mapping, and backend permissions can all shape the outcome.

## How It Works

Use a simple triage sequence:

| Symptom | First questions |
| --- | --- |
| 4xx increase | Is the request malformed, unauthorized, forbidden, missing a route, or throttled? |
| 5xx increase | Did the integration fail, time out, or lack permission? |
| High integration latency | Which backend subsegment or log line is slow? |
| Queue depth/age rises | Is consumption limited, downstream work slow, or a poison message repeatedly failing? |

For asynchronous integrations, monitor both the acceptance point and the work's eventual result. An API returning success after queuing work does not prove the customer's order is complete. Use correlation IDs and explicit status transitions.

## Architectural Mapping

| Byte Burger | AWS |
| --- | --- |
| Customer order counter | API Gateway |
| Menu and required order details | API contract and validation |
| Badge check | authentication/authorization |
| Counter-to-kitchen handoff | integration |
| Tickets waiting at the pass | queue backlog |

## Painkiller

Separate caller mistakes, front-door policy, and backend failure. Each has a different owner and remediation path.

## Knife Cut

A 4xx is not automatically an attack, and a 5xx is not automatically a Lambda bug. Start with the status pattern and the request path.

## The Masthead

The counter can be healthy while the pantry is overloaded. The next investigation moves to DynamoDB's uneven demand.

## A Note From the Author

See [Amazon API Gateway monitoring](https://docs.aws.amazon.com/apigateway/latest/developerguide/monitoring-cloudwatch.html) and [SQS monitoring](https://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSDeveloperGuide/monitoring-using-cloudwatch.html).

## The Last Bite

An accepted ticket is not a completed meal; observe the whole path.

**Next chapter:** *[Amazon DynamoDB: The Hot Pantry](09-dynamodb-hot-pantry.md)*

Next: DynamoDB pressure reveals why evenly sized pantries can still fail unevenly.
