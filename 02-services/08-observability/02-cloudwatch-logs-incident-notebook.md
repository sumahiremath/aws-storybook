---
description: "The Operations Manager knows a station slowed down, but the dashboard cannot tell her whether it waited for payment, retried a database request, or received malformed orders."
tags:
  - "aws"
  - "observability"
  - "operations"
  - "cloudwatch-logs"
  - "cloudwatch"
---

# Amazon CloudWatch Logs: The Incident Notebook

## The Business Goal

The Operations Manager knows a station slowed down, but the dashboard cannot tell her whether it waited for payment, retried a database request, or received malformed orders.

## The Story

Every station at Byte Burger keeps a notebook. One page per shift is too coarse, and one notebook for the whole chain is chaos. The manager uses a notebook for the service and separates the entries by the worker that wrote them. When a checkout request fails, the entry includes the request ID, route, outcome, duration, and safe diagnostic context. It does not copy the customer's card details onto the wall.

The first slow requests reveal repeated connection setup before the payment call. That is evidence. A raw pile of prose would have made it much harder to see.

## Meet the AWS Service

CloudWatch Logs stores log events in **log groups** and **log streams**. A group is a logical collection, often per application or service; streams separate sources of events, such as Lambda execution environments. Retention controls how long the group keeps data. Applications and AWS services can publish logs, and developers should make entries structured enough to search reliably.

Embedded Metric Format (EMF) lets an application embed metric data in structured log events so CloudWatch can extract metrics while retaining the rich event for investigation. It is useful when a business or application signal needs both a graphable number and supporting context.

## How It Works

Write logs for investigation, not for narration:

- Use consistent fields: timestamp, level, request or correlation ID, operation, outcome, duration, dependency, error type.
- Emit structured JSON when the runtime and tools support it.
- Keep secrets, credentials, tokens, payment data, and unnecessary personal information out.
- Set retention deliberately: incident evidence has value, but indefinite retention adds cost and risk.
- Preserve the request or trace correlation ID so a log line can join a wider story.

EMF is not a reason to encode every event as a metric. Turn stable, aggregatable measurements into metrics; leave a unique stack trace or payload summary as diagnostic detail.

## Architectural Mapping

| Byte Burger | CloudWatch Logs |
| --- | --- |
| A notebook for checkout | log group |
| One worker's entries | log stream |
| How long the archive stays | retention policy |
| A tally written inside an incident note | embedded metric format |

## Painkiller

Good logs let a new responder reconstruct what happened without asking the original developer to translate every line.

## Knife Cut

Logging an exception is not sufficient if the line lacks operation, correlation, and outcome. Conversely, logging the entire request is usually a data-handling mistake.

## The Masthead

The notebook contains the clue. The next job is to ask it a precise question at speed.

## A Note From the Author

See [CloudWatch Logs concepts](https://docs.aws.amazon.com/AmazonCloudWatch/latest/logs/WhatIsCloudWatchLogs.html) and [Embedded Metric Format](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/CloudWatch_Embedded_Metric_Format.html).

## The Last Bite

The incident notebook has thousands of entries. The Operations Manager needs an evidence wall, not a longer scroll bar.

**Next chapter:** *[Amazon CloudWatch Logs Insights: The Evidence Wall](03-cloudwatch-logs-insights-evidence-wall.md)*

Next: Logs Insights, metric filters, and subscription filters turn log events into action.
