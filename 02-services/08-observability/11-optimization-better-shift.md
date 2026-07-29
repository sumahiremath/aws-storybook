---
description: "The checkout incident is closed. The tempting response is to buy more of everything: more concurrency, larger functions, more database capacity, more alarms. That can be expensive, noisy, and still leave the next bottleneck untouched."
tags:
  - "aws"
  - "observability"
  - "operations"
  - "epilogue"
---

# Troubleshooting and Optimization: The Better Shift

## The Business Goal

The checkout incident is closed. The tempting response is to buy more of everything: more concurrency, larger functions, more database capacity, more alarms. That can be expensive, noisy, and still leave the next bottleneck untouched.

## The Story

The General Manager redesigns the shift around avoided work. The cashier does not ask the kitchen for information already displayed on the menu board. The kitchen batches compatible prep rather than walking to the pantry once per item. Regular customers do not cause every station to create a new supplier connection. Orders that do not match a specialist station never reach it. Each improvement has a measurement beside it.

When the lunch rush returns, completed orders rise, p95 checkout latency stays low, queue age is controlled, and the pantry no longer receives repeated reads for the same menu card. That is optimization: a measured improvement in a real constraint, not a collection of larger settings.

## Meet the AWS Service

Common AWS application optimizations include:

- caching with CloudFront, API Gateway cache, ElastiCache, DAX, or an application cache when data can safely be reused;
- Lambda memory/CPU sizing, concurrency choices, provisioned concurrency where startup latency matters, and connection/client reuse;
- SQS batching where the consumer and downstream dependencies can handle it;
- SNS subscription filtering to avoid delivering irrelevant messages;
- reducing unnecessary downstream calls, inefficient scans, duplicated serialization, and over-broad work;
- profiling and measuring before and after the change.

## How It Works

Pick the performance lever from the bottleneck:

| Bottleneck | Consider | Verify |
| --- | --- | --- |
| Repeated safe reads | Appropriate cache | hit rate, freshness, origin pressure |
| Lambda compute-bound work | Memory/CPU sizing | duration, cost per invocation, errors |
| Cold-start-sensitive path | Provisioned concurrency | tail latency and utilization |
| Queue processing overhead | Batch size and partial-failure design | age, throughput, downstream load |
| Irrelevant fan-out | SNS filter policy | delivered versus useful messages |
| Repeated connection/setup | Client and connection reuse | subsegment timing, errors |

Always make the trade-off explicit. A cache can serve stale data. A larger batch can increase the blast radius of a failure. More concurrency can overload a database. A lower alarm threshold can make people ignore it. Observability validates the change and detects its second-order effects.

## Architectural Mapping

| Byte Burger | Optimization |
| --- | --- |
| Menu board holding common information | cache |
| Preparing several compatible items together | batching |
| Routing only vegan orders to the vegan station | subscription filtering |
| Keeping supplier line open | connection reuse |
| Measuring the lunch shift after the change | metrics, logs, traces |

## Painkiller

The best optimization removes unnecessary work before scaling necessary work. Measure the customer outcome and the downstream cost together.

## Knife Cut

There is no universal “fastest” configuration. Freshness, correctness, cost, capacity, and recovery behavior are part of performance engineering.

## The Masthead

Byte Burger can now operate under pressure because every prior section has a role: identities protect it, compute runs it, storage and databases hold it, APIs and integrations move work through it, deployment changes it, and observability proves how it behaves.

## A Note From the Author

Relevant AWS references include [Lambda performance optimization](https://docs.aws.amazon.com/lambda/latest/dg/configuration-memory.html), [Amazon SQS batching](https://docs.aws.amazon.com/lambda/latest/dg/with-sqs.html), and [CloudWatch metrics](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/working_with_metrics.html).

## The Last Bite

The mature system is not the one that never has an incident. It is the one that can explain the incident, recover deliberately, and make the next shift better.

**Next section:** *[Amazon VPC, Amazon Route 53, and Amazon CloudFront: Byte Burger's Perimeter](../09-networking/00-the-restaurant-perimeter.md)*

The observability section is complete. Return to any earlier section with this final question: **how would we know this part of Byte Burger is failing, and what evidence would prove why?**
