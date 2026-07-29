---
description: "The trace says connection setup became slow after release. That still leaves a dangerous question: did code change, or did someone change the function's networking, role, secret, or deployment configuration?"
tags:
  - "aws"
  - "observability"
  - "operations"
  - "cloudtrail"
---

# AWS CloudTrail: The Security Guard

## The Business Goal

The trace says connection setup became slow after release. That still leaves a dangerous question: did code change, or did someone change the function's networking, role, secret, or deployment configuration?

## The Story

The Security Guard does not watch every customer walking to the pickup counter. He records the people who use the staff doors and change the building: who adjusted the oven, attached a key policy, changed a role, or updated a function. At 11:03, he finds an AWS API event showing a new Lambda configuration. The General Manager now has a lead that connects the symptom to a specific control-plane action.

## Meet the AWS Service

CloudTrail records AWS account activity, including API calls from the console, AWS SDKs, CLI tools, and AWS services. **Management events** describe control-plane operations such as creating or configuring resources. **Data events** capture high-volume resource operations for supported services, such as object-level S3 activity or DynamoDB item activity, and must be selected explicitly. Event history provides a searchable recent record of management events in a Region; a trail or CloudTrail Lake/event data store provides a configured ongoing record.

## How It Works

When an incident hypothesis involves a resource or permission change, inspect the event for:

- identity and assumed role;
- event name and source;
- time, Region, source IP, and request parameters where available;
- changed resource and response/error information.

Choose data-event logging deliberately. It can be operationally essential for a particular audit question but is not enabled by default for trails or event data stores and can add cost. Keep it scoped to the resources and operations that matter.

CloudTrail and CloudWatch have different jobs. CloudWatch tells you that Lambda duration rose or an application logged `AccessDenied`. CloudTrail can show that an IAM policy was changed or an API call was denied. Sending CloudTrail logs to CloudWatch Logs can make them easier to monitor, but it does not merge the concepts.

## Architectural Mapping

| Byte Burger | CloudTrail |
| --- | --- |
| Staff-door change book | event record |
| Regular building changes | management events |
| Recording every item taken from a specific pantry | selected data events |
| Recent desk ledger | Event history |
| Long-term archive | trail or Lake/event data store |

## Painkiller

Use CloudTrail to establish accountable AWS activity, especially around deployments, permissions, and resource configuration.

## Knife Cut

Event history is regional, recent management-event history—not a substitute for a deliberately configured, durable audit design. Nor does it show data events by default.

## The Masthead

The Guard confirms a configuration change. The next job is to recognize the common ways a Lambda station fails under real traffic.

## A Note From the Author

See [how CloudTrail works](https://docs.aws.amazon.com/awscloudtrail/latest/userguide/how-cloudtrail-works.html) and [CloudTrail event types](https://docs.aws.amazon.com/awscloudtrail/latest/userguide/cloudtrail-events.html).

## The Last Bite

Audit evidence narrows the hypothesis. It does not absolve us from reading the failing station's behavior.

**Next chapter:** *[AWS Lambda: The Failing Station](07-lambda-failing-station.md)*

Next: Lambda under pressure—timeouts, throttles, and resource choices.
