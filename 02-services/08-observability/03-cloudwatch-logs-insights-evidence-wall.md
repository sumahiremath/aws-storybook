---
description: "Use CloudWatch Logs Insights to query growing log data for patterns that individual entries cannot reveal."
tags:
  - "aws"
  - "observability"
  - "operations"
  - "logs-insights"
  - "cloudwatch-logs"
  - "cloudwatch"
---

# Amazon CloudWatch Logs Insights: The Evidence Wall

## The Business Goal

At lunch, the notebook grows faster than any person can read. A responder who searches one phrase at a time will find anecdotes, not a pattern.

## The Story

The Operations Manager pins only relevant slips to an evidence wall. She asks: Which checkout operation took longer than two seconds after 11:07? Which error type increased? Which function version wrote those entries? The answers form a pattern: connection creation is the expensive step, not payment authorization itself.

For a known warning—“inventory service unavailable”—she also wants a counter that can ring the bell next time. For a security event stream, she wants matching entries delivered to the team that handles them. Those are three distinct jobs.

## Meet the AWS Service

CloudWatch Logs Insights runs interactive queries across log groups to investigate and aggregate log data. A **metric filter** watches matching log events and publishes values as a CloudWatch metric. A **subscription filter** routes matching log events continuously to a destination such as Lambda, Kinesis, Firehose, or a cross-account destination. They use filtering ideas, but they are not interchangeable.

## How It Works

Use the tool that matches the timing and destination of the decision:

| Need | Choose | Why |
| --- | --- | --- |
| Explore an incident now | Logs Insights | Query historical log data interactively |
| Alarm on a repeatable pattern | Metric filter | Convert matching events into a metric |
| Send matching events elsewhere | Subscription filter | Stream log events to a consumer |

Logs Insights benefits from structured fields and narrow time windows. Start with the symptom, group by error type, then correlate with function version, route, or dependency. A metric filter is intentionally simpler: it recognizes a known pattern, not a novel incident. A subscription filter creates an operational pipeline, so protect its destination from bursts and failures.

## Architectural Mapping

| Byte Burger | AWS |
| --- | --- |
| Asking the board questions | Logs Insights query |
| A counter for a recurring warning | metric filter |
| Conveyor carrying selected slips to another team | subscription filter |

## Painkiller

Investigation turns unknown questions into queries; prevention turns known answers into metrics and alarms.

## Knife Cut

Do not use a metric filter as a substitute for context-rich logs, or a subscription filter merely to avoid opening Logs Insights. Each adds operational cost and responsibility.

## The Masthead

The evidence wall identifies repeated connection setup. It still cannot show how one order crossed every service boundary.

## A Note From the Author

AWS documents [Logs Insights queries](https://docs.aws.amazon.com/AmazonCloudWatch/latest/logs/AnalyzingLogData.html) and the shared [filter pattern syntax](https://docs.aws.amazon.com/AmazonCloudWatch/latest/logs/FilterAndPatternSyntax.html).

## The Last Bite

Logs tell the story one entry at a time. The Store Manager needs the entire receipt.

**Next chapter:** *[AWS X-Ray: The Store Manager](04-xray-store-manager.md)*

Next: AWS X-Ray follows a request through Byte Burger.
