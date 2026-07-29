---
description: "“Checkout is slow” is not yet an operational fact. Slow compared with what? In which Region? For every customer, or only one menu channel? Without a common view, each team measures a different restaurant."
tags:
  - "aws"
  - "observability"
  - "operations"
  - "cloudwatch"
---

# Amazon CloudWatch: The Operations Manager

## The Business Goal

“Checkout is slow” is not yet an operational fact. Slow compared with what? In which Region? For every customer, or only one menu channel? Without a common view, each team measures a different restaurant.

## The Story

The Operations Manager's board has a few deliberate gauges: orders accepted, checkout latency, error rate, queue depth, Lambda duration and throttles, and DynamoDB capacity pressure. At 11:07, latency and Lambda duration rise together while order volume is ordinary. The board rules out a lunch rush and makes the kitchen station a better hypothesis.

She puts a bell beside the indicators that truly require action. A single late order does not ring it. Sustained p95 latency does.

## Meet the AWS Service

CloudWatch metrics are time-ordered measurements. AWS services publish many metrics automatically; applications can publish custom metrics. A namespace groups related metrics, and dimensions distinguish useful slices such as function name, API stage, queue, table, or Region. A dashboard places selected metrics and alarms in one view. An alarm watches a metric or expression and can notify or trigger an approved action when it changes state.

## How It Works

Choose signals that match a customer outcome and the dependency that could explain it.

- **Availability:** errors, failed requests, unhealthy targets.
- **Latency:** duration and percentile latency, not only an average.
- **Traffic:** requests, messages, concurrency, throughput.
- **Saturation:** throttles, queue age/depth, consumed capacity, connections.

Use dimensions carefully. A global average can hide one failing function version or API route. Publish custom metrics for business events only when they answer an operational question, such as `OrdersCompleted` or `PaymentDeclines`; do not make a unique dimension value for every order ID.

Alarms have states: `OK`, `ALARM`, and `INSUFFICIENT_DATA`. Design the missing-data behavior intentionally. An alarm can notify through Amazon SNS or initiate supported automated responses, but automatic action should be reversible and understood.

## Architectural Mapping

| Byte Burger board | CloudWatch |
| --- | --- |
| Gauge | metric |
| Label such as “drive-through” | dimension |
| Wall display | dashboard |
| Escalation bell | alarm and action |
| “Too many ovens are occupied” | service quota or saturation alarm |

## Painkiller

Alert on symptoms that need a person, not every measurement that exists. A dashboard helps a human understand; an alarm asks a human or system to act.

## Knife Cut

CPU alone does not prove customer pain. A healthy-looking average also does not prove every customer is healthy. Pair infrastructure signals with request outcomes and percentiles.

## The Masthead

CloudWatch makes Byte Burger observable from the floor up, but a gauge cannot explain the words inside a failed order.

## A Note From the Author

See [CloudWatch dashboards](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/CloudWatch_Dashboards.html) and [CloudWatch alarms](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/CloudWatch_Alarms.html).

## The Last Bite

The dashboard says the Lambda station slowed down at exactly the right time. Now the incident notebook must say what it was doing.

**Next chapter:** *[Amazon CloudWatch Logs: The Incident Notebook](02-cloudwatch-logs-incident-notebook.md)*

Next: CloudWatch Logs becomes the record written during the shift.
