---
description: "An incident has produced a dashboard spike, a handful of logs, one slow trace, and a configuration event. Each clue is real. None, alone, proves the cause."
tags:
  - "aws"
  - "observability"
  - "operations"
  - "concept"
---

# Troubleshooting and Optimization: The General Manager

## The Business Goal

An incident has produced a dashboard spike, a handful of logs, one slow trace, and a configuration event. Each clue is real. None, alone, proves the cause.

## The Story

The General Manager writes a timeline on the whiteboard:

| Time | Evidence | Meaning |
| --- | --- | --- |
| 11:03 | CloudTrail records Lambda configuration update | A change occurred |
| 11:07 | CloudWatch checkout p95 and duration rise | Customer impact begins |
| 11:08 | Logs show repeated client setup | A local behavior changed |
| 11:09 | X-Ray shows setup dominates the checkout trace | Location and duration confirmed |
| 11:20 | Reusing the client lowers p95 without raising errors | Hypothesis tested |

This is a causal chain strong enough to act on. “It seemed like networking” is not.

## Meet the AWS Service

Root-cause analysis is a practice across services. CloudWatch supplies metrics, logs, alarms, and operational history. X-Ray exposes a request path and timing. CloudTrail establishes AWS API activity. Deployment systems supply change events and rollback evidence. The responsible engineer combines them in a time-aligned hypothesis.

## How It Works

Use a repeatable incident loop:

1. **State the symptom precisely.** Which customer outcome, scope, and time window changed?
2. **Protect the service.** Roll back, shed optional work, slow callers, or fail safely when warranted.
3. **Build a timeline.** Compare metrics, deploy/configuration events, logs, and traces in the same Region and clock window.
4. **Form competing hypotheses.** Slow dependency, capacity limit, authorization change, bad input, or deployment defect are not the same problem.
5. **Test the smallest safe change.** Prefer reversible actions and validate against the original symptom.
6. **Capture the learning.** Add the missing metric, log field, trace annotation, alarm, runbook, or guardrail.

Common patterns deserve specific checks: Lambda timeout versus throttling; API Gateway 4xx versus 5xx; SQS backlog versus poisoned batches; DynamoDB hot keys versus broad capacity; `AccessDenied` versus missing resource; CloudFormation rollback versus application runtime failure.

## Architectural Mapping

| Byte Burger | Operational practice |
| --- | --- |
| Incident whiteboard | time-aligned evidence timeline |
| “Why did the queue grow?” | hypothesis |
| Small reversible kitchen adjustment | controlled mitigation |
| Updated shift guide | runbook and improved telemetry |

## Painkiller

Separate correlation from cause. A deployment near an incident is suspicious; the chain of evidence makes it actionable.

## Knife Cut

Do not tune an alarm threshold just because it revealed a real incident. Do not raise a timeout merely because it prevented an error. Both may erase the symptom while preserving the cause.

## The Masthead

The incident is resolved, but Byte Burger should not merely return to normal. It should need less work to serve the next rush.

## A Note From the Author

This workflow combines the capabilities described in [CloudWatch](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/WhatIsCloudWatch.html), [X-Ray](https://docs.aws.amazon.com/xray/latest/devguide/xray-concepts.html), and [CloudTrail](https://docs.aws.amazon.com/awscloudtrail/latest/userguide/cloudtrail-user-guide.html).

## The Last Bite

An incident is only fully solved when the system is safer, clearer, or cheaper the next time it is under pressure.

**Next chapter:** *[Troubleshooting and Optimization: The Better Shift](11-optimization-better-shift.md)*

Next: the final shift turns evidence into durable performance improvements.
