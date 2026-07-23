# AWS Lambda: The Errand Runner

> Amazon ECS keeps your business running. AWS Batch clears a backlog. AWS Lambda is the errand runner invoked when an event needs bounded work.

---

## The Pain

Not every piece of work deserves a restaurant.

Not every task belongs in a catering company.

Sometimes a single event happens.

A customer places an online order.
A photo is uploaded.
The clock strikes midnight.
A payment arrives.

Each event needs a small piece of work.

---

## The Story

Hiring a full-time employee for every small task would be wasteful.

You want an errand runner who appears when called.

The runner receives an assignment.

Runs.

Completes the errand.

Then the invocation ends.

---

## Meet AWS Lambda

Imagine every food truck can summon an errand runner.

The runner does not cook the whole menu.
The runner does not manage the truck.

Another service, event source, or application calls with an assignment:

> "Can you take care of this?"

Lambda runs the function, performs the bounded work, and ends the invocation.

> **Core idea:** Lambda executes code in response to invocations without requiring you to manage an always-running server fleet.

---

## How It Works

### Where the Errands Come From

```text
API Gateway  --+
S3           --|
EventBridge  --|
SNS          --+--> AWS Lambda --> Function Result
SQS          --|
DynamoDB     --+
```

Invocation behavior depends on the source. Some services invoke Lambda directly, while event source mappings poll streams or queues and invoke the function with retrieved records.

### Invocation Lifecycle

```text
Event or Request Arrives
          |
          v
Lambda Starts an Invocation
          |
          v
Function Processes Work
          |
          v
Invocation Ends
```

There are no customer-managed servers or fleets to operate. A later event starts another invocation.

### ZIP Package or Container Image

The errand runner still needs instructions and supplies.

A Lambda function can be packaged as a ZIP archive.

It can also use a container image stored in Amazon ECR. A Lambda container image can be up to 10 GB uncompressed.

The package changes how the code is delivered. The operating model stays the same: something invokes the function, Lambda performs the work, and the invocation ends.

---

## AWS Batch vs. Lambda

| AWS Batch | AWS Lambda |
|---|---|
| Schedules finite jobs | Reacts to events or consumes messages |
| Drains a Batch job queue | Runs bounded function invocations |
| Catering company | Errand runner |
| Backlog-driven | Event-driven |

Batch is designed to schedule finite jobs and drain its backlog.

Lambda reacts to events or consumes messages from an event source.

When Lambda reads from Amazon SQS, SQS owns the queue. A Lambda event source mapping polls that queue and invokes the function with messages.

One invocation does not always mean one record. Depending on the event source and its configuration, an invocation may receive a batch of records for the runner to process together.

---

## Real-World Limits

The errand runner is built for short assignments.

A single Lambda invocation can run for a maximum of 15 minutes. Long-running jobs and large finite backlogs may fit AWS Batch better, while permanently running customer-facing services belong on service-oriented compute such as ECS.

The runner may also need a moment to get ready. When Lambda creates a new execution environment, the invocation can experience **cold-start latency** before the function begins processing the event.

Lambda is strongest when work is short, event-driven, and able to finish within one invocation.

---

## Architectural Mapping

| In the story | In AWS | What it means |
|---|---|---|
| Errand runner | Lambda function | Code and configuration for bounded work |
| Someone calling | Invocation source | Service, event source, or application requesting work |
| Assignment | Event payload | Data delivered to an invocation |
| One appearance | Invocation | One bounded execution of the function |
| Bag of order slips | Batched records | Multiple records delivered in one invocation |
| Completed errand | Function result or side effect | Outcome produced by the invocation |

---

## When to Use It

Use Lambda for work such as:

- processing image uploads
- handling API requests
- reacting to database changes
- running scheduled tasks
- processing queue or stream records
- coordinating short event-driven actions

---

## When Not to Use It

Consider another compute service when:

- one execution needs more than 15 minutes
- a customer-facing process must remain continuously running
- the workload requires host-level control
- a large finite backlog fits Batch scheduling better
- startup latency cannot tolerate the workload's cold-start profile

---

## Painkiller

> **Problem:** Small pieces of work need to run when events occur.
> **Pain:** Keeping servers running for occasional bounded work wastes money and operational effort.
> **AWS solution:** Use Lambda to invoke a function when work arrives and end the invocation when that work finishes.

---

## Knife Cut

> **AWS Batch drains a backlog. AWS Lambda answers an invocation.**

---

## The Masthead

### What Actually Just Happened

| In the story | In AWS | What it actually means |
|---|---|---|
| Errand runner | Lambda function | Deployable code and configuration |
| Call for help | Invocation | Request to execute the function |
| Errand instructions | Event payload | Input supplied to the handler |
| Several order slips | Record batch | Multiple source records in one invocation |
| Runner getting ready | Cold start | Initialization of a new execution environment |
| Runner leaving | Invocation completion | Execution ends after success, failure, or timeout |

---

## A Note From the Author

The errand runner makes every call sound immediate, isolated, and successful. Lambda invocation, retry, batching, ordering, and failure behavior depend on the event source and invocation type. Applications must be designed for duplicate processing and partial failures where those behaviors are possible.

Lambda may reuse an execution environment, but reuse is not guaranteed. Teams still manage function code, IAM permissions, dependencies, configuration, observability, concurrency, error handling, and downstream capacity.

- [AWS Lambda invocation methods](https://docs.aws.amazon.com/lambda/latest/dg/lambda-invocation.html)
- [Lambda quotas](https://docs.aws.amazon.com/lambda/latest/dg/gettingstarted-limits.html)
- [Using Lambda with Amazon SQS](https://docs.aws.amazon.com/lambda/latest/dg/with-sqs.html)

---

## The Last Bite

Restaurants stay open.

Catering companies drain a backlog.

The errand runner appears when an event calls.

Lambda performs the assigned event or record batch, and the invocation ends.

---

**Next chapter:** *Epilogue: The Wand Was Choosing All Along*

The errand runner completes the final compute lesson.

The epilogue returns to Ollivander's shop to reveal the question that connected every chapter.
