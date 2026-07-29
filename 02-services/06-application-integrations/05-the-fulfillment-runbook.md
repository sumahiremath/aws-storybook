---
description: "Coordinate a multi-step process with explicit state, choices, parallel work, waiting, and service integrations."
tags:
  - "aws"
  - "application-integration"
  - "event-driven"
  - "step-functions"
---

# AWS Step Functions: The Fulfillment Runbook

> Coordinate a multi-step process with explicit state, choices, parallel work, waiting, and service integrations.

## The Business Goal

Order 42 was paid.

The burger station finished.

The drink station never received its task.

The assembly counter waited, but nobody knew whether it was waiting for a drink, a payment confirmation, or a courier.

Each department remembered its own work.

No system remembered the order's complete journey.

---

## The Story

Nia created one fulfillment runbook for every order.

The runbook marked payment complete.

It opened parallel sections for food and drink.

It repeated the preparation step for every item.

It waited for both sections to finish.

Then it chose pickup or delivery.

If a required step failed beyond recovery, it followed a compensating path and closed the order deliberately.

The runbook did not cook.

It remembered what had happened and what should happen next.

---

## The Wrong Way

A chain of functions can hide workflow state inside code, payloads, retries, and logs.

When step four fails, developers must reconstruct which earlier steps completed and which should run again.

Step Functions does not eliminate business design. It makes the designed control flow explicit and observable.

---

## Meet the AWS Service

> **Core idea:** AWS Step Functions runs state machines whose executions move data through declared states and service integrations.

A state machine is the workflow definition. An execution is one running instance, such as fulfillment for order 42.

AWS manages workflow state transitions and supported integration behavior. You manage the state machine, inputs, permissions, timeouts, retries, compensation, idempotency, observability, and cost.

---

## How It Works

### Give One Station Work

#### Task

A `Task` performs one unit of work through Lambda, an AWS SDK integration, an optimized service integration, an activity, or another supported resource.

Examples include writing an order, publishing an event, sending an SQS message, or invoking a Lambda function.

### Choose Pickup or Delivery

#### Choice

A `Choice` evaluates data and selects a branch. It does not perform the business task itself.

Every expected condition needs a route, including a default when appropriate.

### Prepare Food and Drink Together

#### Parallel

A `Parallel` state starts different branches concurrently and waits for all branches to complete before continuing.

It fits a known set of different work streams.

### Prepare Every Item

#### Map

A `Map` repeats the same sub-workflow for items in a collection.

Inline Map fits bounded processing inside one execution. Distributed Map supports larger, highly parallel data-processing workloads with different execution and service considerations.

### Hold for a Scheduled Moment

#### Wait

A `Wait` pauses until a duration or timestamp. Step Functions maintains the workflow state while it waits.

### Reshape Without Calling a Worker

#### Pass

A `Pass` can forward, add, or transform data without performing external work. It is useful for shaping workflow input or providing fixed results.

### Close the Runbook

#### Succeed and Fail

`Succeed` ends successfully. `Fail` ends with an error.

An execution should have an intentional terminal outcome instead of drifting into an ambiguous final state.

### Call Byte Burger's Systems

#### Service Integrations

Step Functions can use request-response integrations, wait for supported jobs, or pause for a callback task token in supported workflow types.

A callback can represent an external approval or a kitchen system that reports completion later. Task tokens are credentials for completing waiting work and must be protected.

---

## Architectural Mapping

```text
Validate order
      |
Charge payment
      |
  +---+---+
  |       |
Food    Drink
  |       |
  +---+---+
      |
 Choice: pickup or delivery
    /                 \
Succeed          Dispatch courier
```

The state machine's execution role needs permission for every AWS API action it invokes. Target services may also require resource-based permission or additional trust.

---

## When to Use It

Use Step Functions when:

- One business process has multiple dependent steps
- Branching, parallelism, iteration, or waiting is explicit
- Workflow progress and execution history aid operations
- AWS service integrations can replace custom coordination code
- Errors require step-specific retry or recovery paths

## When Not to Use It

Use EventBridge choreography when independent consumers should react to facts without one central process owning the sequence. Use SQS when the main need is buffered work rather than a multi-step state machine.

---

## Painkiller

> **Problem:** Every department knows its own task but no system remembers the order's complete progress.  
> **Pain:** Partial failure leaves ambiguous work and custom coordination code.  
> **AWS solution:** Step Functions makes the process explicit as a state machine with durable execution state.

---

## Knife Cut

> **Tasks perform work. The state machine remembers which work should happen next.**

---

## The Masthead

### What Actually Just Happened

|In the story|In Step Functions|What it actually means|
|---|---|---|
|Fulfillment runbook|State machine|Workflow definition|
|Order 42's runbook|Execution|One running workflow instance|
|Station task|Task state|One service call or unit of work|
|Pickup or delivery|Choice state|Conditional branch|
|Food and drink|Parallel state|Different branches run concurrently|
|Each order item|Map state|Repeated sub-workflow over a collection|
|Hold for courier|Wait or callback|Persisted pause for time or external completion|

---

## A Note From the Author

A paper runbook suggests a human can erase a checkmark. Real executions have defined state and history behavior, and external side effects may not roll back merely because a later state fails.

Compensating actions such as refunds must be designed explicitly. Service integrations, quotas, payload limits, timeouts, task-token security, execution type, and pricing still affect the architecture.

- [Step Functions workflow states](https://docs.aws.amazon.com/step-functions/latest/dg/workflow-states.html)

---

## The Last Bite

The kitchen stations knew how to work.

The missing piece was something that remembered the journey.

> **When completion depends on ordered decisions, make the process itself visible.**

---

**Next chapter:** *[AWS Step Functions: When the Fryer Fails](06-when-the-fryer-fails.md)*

The runbook can describe the happy path. The next lunch rush tests what happens when payment times out, the fryer fails, and retries begin.

