---
description: "An accepted order is a promise to continue—not proof that the meal is ready."
tags:
  - "aws"
  - "resilience"
  - "architecture"
  - "api-gateway"
  - "sqs"
  - "lambda"
  - "capstone"
---

# Amazon API Gateway, Amazon SQS, and AWS Lambda: Accepted Is Not Complete

> An accepted order is a promise to continue—not proof that the meal is ready.

## The Business Goal

The promotion drives checkout traffic beyond the pace at which every downstream activity can finish. If the counter waits for payment, inventory, receipt, loyalty, and kitchen preparation, one slow dependency makes every customer wait.

## The Story

Byte Burger accepts a customer’s order, gives an order number, and puts durable work on the kitchen ticket rail. The customer sees “order received,” not “meal completed.” A status screen later shows the real state.

## Meet the AWS Service

API Gateway can accept a request and invoke application work. SQS durably buffers messages for consumers. Lambda event source mappings poll queues and process batches. Together they separate customer acceptance from background fulfillment.

## How It Works

Return an immediate result only for work the caller truly needs immediately. For durable asynchronous work:

- persist enough business state to show honest status;
- enqueue a durable work item with a correlation/order ID;
- make consumers idempotent because delivery can repeat;
- monitor queue depth and age, not only API success;
- communicate completion through polling, callback, notification, or a client-update path.

## Architectural Mapping

| Story | AWS |
| --- | --- |
| Order number | accepted asynchronous request |
| Ticket rail | SQS queue |
| Runner | Lambda consumer |
| Pickup status | durable business state |

## When to Use It

Use this pattern when the request can be accepted now and completed reliably later.

## When Not to Use It

Do not use it when the caller must receive the authoritative outcome before continuing.

## Painkiller

> **Problem:** Slow downstream work makes a customer-facing request fragile.  
> **Pain:** A timeout hides whether the order was lost, still working, or completed twice.  
> **AWS solution:** Accept durable work explicitly and expose an honest completion state.

## Knife Cut

> `202 Accepted`-style semantics are not successful fulfillment.

## The Masthead

### What Actually Just Happened

| Story | AWS | Meaning |
| --- | --- | --- |
| Counter receipt | API response | Request accepted |
| Kitchen ticket | queue message | Durable work request |
| Pickup screen | status API/state | Eventual customer outcome |

## A Note From the Author

Queues buffer pressure but do not create unlimited capacity. Consumers, downstream databases, visibility timeout, batch behavior, redrive policy, and status lifecycle still require deliberate design.

- [Using AWS Lambda with Amazon SQS](https://docs.aws.amazon.com/lambda/latest/dg/with-sqs.html)
- [Amazon SQS visibility timeout](https://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSDeveloperGuide/sqs-visibility-timeout.html)

## The Last Bite

Acceptance is a contract to finish safely, not permission to forget the order.

**Next chapter:** *[AWS SDK and AWS Step Functions: The Payment Supplier Slows Down](02-sdk-step-functions-payment-supplier-slows-down.md)*

The durable ticket protects the customer path. The payment dependency still needs protection from a retry storm.
