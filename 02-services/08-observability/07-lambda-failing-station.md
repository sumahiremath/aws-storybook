---
description: "The new checkout configuration is real, but “the Lambda is slow” can mean several different things. The station may run out of time, be denied more concurrent workers, wait on a dependency, or simply have too little compute for the work."
tags:
  - "aws"
  - "observability"
  - "operations"
  - "lambda"
---

# AWS Lambda: The Failing Station

## The Business Goal

The new checkout configuration is real, but “the Lambda is slow” can mean several different things. The station may run out of time, be denied more concurrent workers, wait on a dependency, or simply have too little compute for the work.

## The Story

The General Manager separates four scenes that look similar from the dining room:

- A ticket expires before the cook finishes: **timeout**.
- The station has no permitted worker for the next ticket: **throttling/concurrency limit**.
- The cook is busy waiting for an outside supplier: **slow downstream dependency**.
- The cook is working hard but under-equipped: **memory/CPU sizing**.

The checkout trace and logs show repeated connection creation. The function has enough duration budget, but every warm worker starts a new client connection. The repair is to reuse the client outside the handler where appropriate, then measure duration and error rate again.

## Meet the AWS Service

Lambda publishes operational metrics such as invocations, errors, duration, throttles, and concurrent executions to CloudWatch. Timeout ends an invocation that has not completed within its configured limit. Reserved concurrency can reserve and cap a function's concurrency; provisioned concurrency keeps initialized environments ready to reduce cold-start impact for a chosen version or alias. Memory selection also affects available CPU and other resources.

## How It Works

Diagnose in order:

1. Compare errors, duration percentiles, throttles, concurrency, and iterator/queue signals in the same window.
2. Read the error type and trace the slow downstream call.
3. Check recent deployment, configuration, IAM, VPC, or secret changes with CloudTrail when relevant.
4. Match the fix to the failure: timeout only when work legitimately needs longer; concurrency only when downstream capacity and cost permit it; memory when profiling shows compute pressure; reuse clients to avoid repeated setup.

For asynchronous or poll-based sources, account for retry behavior, batch failure behavior, dead-letter handling, and idempotency. Increasing concurrency can drain a queue faster but overwhelm a database. A timeout that merely masks a slow dependency is not a cure.

## Architectural Mapping

| Byte Burger | Lambda |
| --- | --- |
| One cook working an order | execution environment |
| Maximum allowed cook time | timeout |
| Number of stations allowed open | concurrency |
| Ready-to-work station before rush | provisioned concurrency |
| Bigger station with faster equipment | more memory and CPU |

## Painkiller

Treat the observable symptom as a classification problem before changing knobs. The safest fix preserves downstream health and gives the next incident clearer evidence.

## Knife Cut

Provisioned concurrency improves readiness, not slow database queries. Reserved concurrency protects account capacity but can also throttle a function if set too low.

## The Masthead

The Lambda station is healthier. A customer may still see an error at the front counter, where failures have different owners.

## A Note From the Author

See [monitoring Lambda functions](https://docs.aws.amazon.com/lambda/latest/dg/monitoring-functions.html) and [Lambda concurrency](https://docs.aws.amazon.com/lambda/latest/dg/lambda-concurrency.html).

## The Last Bite

Tune the station only after knowing whether the station, its queue, or its supplier is the bottleneck.

**Next chapter:** *[Amazon API Gateway and Integrations: The Blocked Counter](08-api-gateway-integration-blocked-counter.md)*

Next: API Gateway and integrations separate customer-facing errors from backend failures.
