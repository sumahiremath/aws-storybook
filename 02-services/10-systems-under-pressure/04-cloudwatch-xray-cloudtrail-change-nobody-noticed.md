---
description: "Metrics show the symptom, traces locate the delay, and audit history explains the AWS change."
tags:
  - "aws"
  - "resilience"
  - "architecture"
  - "cloudwatch"
  - "cloudtrail"
  - "x-ray"
  - "capstone"
---

# Amazon CloudWatch, AWS X-Ray, and AWS CloudTrail: The Change Nobody Noticed

> Metrics show the symptom, traces locate the delay, and audit history explains the AWS change.

## The Business Goal

The dashboard proves that p95 checkout latency rose at 12:07. A rollback is tempting, but the team needs to know whether the release, a permission, a configuration change, the payment supplier, or the hot inventory key caused the customer impact.

## The Story

The Operations Manager sees the queue and latency gauges. The Store Manager follows one order receipt through the API, Lambda, payment call, and inventory lookup. The Security Guard opens the change book and finds a configuration update minutes before the rush. The General Manager builds one timeline instead of selecting a favorite explanation.

## Meet the AWS Service

CloudWatch provides metrics, logs, alarms, and dashboards. X-Ray traces request paths and downstream timing. CloudTrail records AWS account activity such as configuration and permission API calls. They answer different questions and become powerful when aligned by time and correlation identifiers.

## How It Works

Start broad, then narrow:

1. Use CloudWatch to identify scope, onset, saturation, and customer symptom.
2. Query structured logs for error class, correlation ID, route, version, and dependency.
3. Use X-Ray to locate slow segments and downstream calls.
4. Use CloudTrail when the hypothesis involves an AWS control-plane change.
5. Test the smallest reversible mitigation and measure the original symptom again.

## Architectural Mapping

| Story | AWS |
| --- | --- |
| Floor board | CloudWatch |
| One order receipt | X-Ray trace |
| Change book | CloudTrail |
| Incident timeline | correlated evidence |

## When to Use It

Use all three when an incident could cross application behavior, dependencies, and AWS configuration.

## When Not to Use It

Do not search CloudTrail for the performance of an individual customer request; it is audit history, not request tracing.

## Painkiller

> **Problem:** A release-adjacent incident creates plausible but competing explanations.  
> **Pain:** Guessing changes production without proving a cause.  
> **AWS solution:** Align metrics, logs, traces, and audit events in one evidence timeline.

## Knife Cut

> CloudWatch describes behavior. CloudTrail describes account activity. X-Ray describes one request path.

## The Masthead

### What Actually Just Happened

| Story | AWS | Meaning |
| --- | --- | --- |
| Latency gauge | metric/alarm | Symptom and time window |
| Receipt route | trace | Location and duration |
| Configuration entry | CloudTrail event | Account activity evidence |

## A Note From the Author

Correlation is not causation. A configuration change near an incident is a hypothesis until the observed mechanism and mitigation support it.

- [Amazon CloudWatch documentation](https://docs.aws.amazon.com/cloudwatch/)
- [AWS X-Ray documentation](https://docs.aws.amazon.com/xray/)
- [AWS CloudTrail documentation](https://docs.aws.amazon.com/cloudtrail/)

## The Last Bite

The fastest incident response is a short, repeatable chain of evidence.

**Next chapter:** *[Amazon SQS, AWS Step Functions, and Amazon DynamoDB: The Recovery Shift](05-sqs-step-functions-dynamodb-recovery-shift.md)*

The immediate pressure is contained. Byte Burger must now recover incomplete orders without replaying successful work.
