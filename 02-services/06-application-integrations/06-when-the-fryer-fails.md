---
description: "Choose the workflow type and error path deliberately, then control retry timing, failure routing, and state data."
tags:
  - "aws"
  - "application-integration"
  - "event-driven"
  - "step-functions"
---

# AWS Step Functions: When the Fryer Fails

> Choose the workflow type and error path deliberately, then control retry timing, failure routing, and state data.

## The Business Goal

The fryer failed while order 42 was running.

The runbook immediately retried.

So did twenty other orders.

The recovering station received twenty simultaneous retries and failed again.

Meanwhile, the payment step had returned a large response containing customer, gateway, and diagnostic data. Every later state carried the whole payload even though it needed only `paymentId`.

The workflow was visible.

It was not yet disciplined.

---

## The Story

Nia changed the runbook.

Temporary fryer errors waited longer between attempts, with a small random variation so every order did not retry on the same beat.

After the retry budget ended, the runbook followed a recovery arrow to notify operations and choose a substitute item or refund.

Each station received only the part of the order it needed and returned only the result the next station needed.

Finally, Nia separated long, auditable customer orders from short, high-volume transformations.

One runbook shape did not need one execution contract.

---

## The Wrong Way

Retrying every error is not resilience.

Validation failures, authorization failures, and malformed input generally need correction rather than repetition. Retrying a non-idempotent payment without a stable idempotency key can create duplicate charges.

`Catch` also does not reverse completed side effects. A refund or release is another explicit task with its own failure behavior.

---

## Meet the AWS Service

> **Core idea:** Step Functions provides workflow-level execution types, retry policies, catch paths, and data transformation so failures are handled as part of the process.

Standard and Express workflows use the same state-machine language but differ in duration, execution semantics, history, integrations, and pricing.

---

## How It Works

### The Long Auditable Order

#### Standard Workflow

Standard workflows suit durable, auditable processes that can run for up to one year. They persist execution state between transitions and retain queryable execution history for a defined period.

They use exactly-once workflow execution semantics unless retry behavior is specified. External service behavior and deliberate retries still require careful side-effect design.

Standard pricing is based primarily on state transitions.

### The Short High-Volume Transformation

#### Express Workflow

Express workflows suit high-volume, short-duration event processing and run for up to five minutes.

Asynchronous Express uses at-least-once workflow execution. Synchronous Express uses at-most-once workflow execution. Actions should match those semantics.

Express pricing considers executions, duration, and memory consumption. Express does not support every Standard integration pattern, including job-run and callback patterns.

The workflow type cannot be changed after state-machine creation; create another state machine to change type.

### Try the Fryer Again

#### Retry

`Task`, `Parallel`, and `Map` states can define retriers by error name.

Important fields include:

- `IntervalSeconds`
- `MaxAttempts`
- `BackoffRate`
- `MaxDelaySeconds`
- `JitterStrategy`

Exponential backoff spreads attempts over increasing intervals. Full jitter randomizes within those intervals to reduce synchronized retry storms.

Retries are state transitions and affect both behavior and cost.

### Take the Recovery Route

#### Catch

After retry policies do not handle an error, a catcher can route the execution to another state.

Catchers are matched in order. `States.ALL` is a wildcard with placement rules. Preserve useful error details without letting error payloads overwrite required workflow data.

### Give Each Station the Right Slip

#### Input and Output Transformation

State input can be filtered or transformed before a task, task parameters can be constructed, results can be selected, and output can be combined or filtered for the next state.

Depending on query-language choice, workflows use JSONPath fields or JSONata expressions and fields. Keep transformations small and contract-driven.

Passing only required data reduces payload growth, accidental exposure, and coupling between states.

---

## Architectural Mapping

```text
Task fails
   |
matches Retry?
  /       \
yes       no / exhausted
 |              |
wait with        v
backoff+jitter  matches Catch?
 |              /       \
retry        recovery    execution fails
```

Metrics, execution events, logs, and traces should reveal retry count, failure rate, throttling, and time spent waiting. Logging sensitive state requires deliberate controls.

---

## When to Use It

Use Standard when the process is long-running, auditable, non-idempotent, or needs callback/job-run patterns. Use Express for short, high-volume processing whose actions fit its execution semantics.

Use Retry for bounded transient failures. Use Catch for explicit alternate control flow.

## When Not to Use It

Do not hide a persistently failing dependency behind enormous retry counts. Apply timeouts, bounded retry budgets, alarms, circuit breaking where appropriate, and a business recovery decision.

---

## Painkiller

> **Problem:** Failures cause synchronized retries, oversized state, and ambiguous execution outcomes.  
> **Pain:** The recovery mechanism overwhelms dependencies or repeats unsafe effects.  
> **AWS solution:** Choose the workflow contract, retry only suitable errors with backoff and jitter, catch exhausted failures, and shape state data deliberately.

---

## Knife Cut

> **Retry asks the same step again. Catch chooses a different next step. Compensation repairs a completed side effect.**

---

## The Masthead

### What Actually Just Happened

|In the story|In Step Functions|What it actually means|
|---|---|---|
|Long customer runbook|Standard workflow|Durable, auditable execution|
|Short rapid runbook|Express workflow|High-volume, short-duration execution|
|Increasing wait|BackoffRate|Multiplier for retry intervals|
|Random pause|JitterStrategy|Spreads concurrent retries|
|Recovery arrow|Catch|Routes matching exhausted errors|
|Small station slip|Input/output transformation|Controls data entering and leaving states|

---

## A Note From the Author

The story treats a retry as though the first attempt clearly failed before doing anything. In distributed systems, a timeout can leave the caller uncertain whether the dependency completed the side effect.

Exactly-once workflow execution is not a universal exactly-once guarantee for every external system. Stable business identifiers, idempotency controls, reconciliation, and compensation remain necessary.

- [Choosing a Step Functions workflow type](https://docs.aws.amazon.com/step-functions/latest/dg/choosing-workflow-type.html)
- [Step Functions error handling](https://docs.aws.amazon.com/step-functions/latest/dg/concepts-error-handling.html)

---

## The Last Bite

A workflow is not reliable because it retries.

It is reliable because it knows what to retry, when to stop, and where failure should go next.

> **Make recovery part of the route, not a panic after the route breaks.**

---

**Next chapter:** *[Amazon Kinesis Data Streams: The River of Receipts](07-the-river-of-receipts.md)*

Individual orders now complete through controlled workflows. Management still needs to understand the continuous activity across Byte Burger.

