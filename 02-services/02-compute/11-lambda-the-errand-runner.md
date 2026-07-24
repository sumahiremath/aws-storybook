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

## When the Errand Runner Meets the Real World

One event calls a runner.

Then ten.

Then a hundred.

One runner cannot answer a hundred calls at the same time.

So Lambda creates more execution environments and runs multiple invocations in parallel.

That parallel work is called **concurrency**.

Each AWS Region has a shared pool of concurrency. When the available pool is exhausted, Lambda throttles new invocations until capacity becomes available.

### Save Some Runners for the Work That Cannot Wait

Most errands can use the shared pool.

Some cannot.

Imagine payment confirmations competing with thumbnail generation.

If thumbnails claim every available runner, payment confirmations wait behind them.

**Reserved concurrency** draws a fence around part of the pool for one function.

Other functions cannot use that reserved capacity.

The protected function cannot grow beyond the fence either.

Reserved concurrency protects critical work from noisy neighbors while also limiting how much concurrency that function can consume.

### Have the Runner Ready Before the Call

A new runner does not always begin instantly.

Lambda may need to prepare a new execution environment before the function can run.

That preparation delay is a **cold start**.

Lambda can reuse an existing environment for later invocations, but reuse is never guaranteed.

For ordinary errands, a brief preparation delay may be acceptable.

For a customer waiting on a checkout screen, it may not be.

**Provisioned concurrency** keeps a chosen number of execution environments initialized before the request arrives.

The runner is already at the door.

No scramble for shoes.

No search for the address.

The work can begin with less startup delay.

But readiness has a price. Provisioned concurrency is billed while that prepared capacity is available, even when it is waiting for work.

### Memory Buys More Than Memory

Some errands are easy.

Some involve resizing a giant photograph, parsing a large file, or calculating thousands of results.

In Lambda, increasing memory also increases the CPU and other resources available to the function.

A larger allocation costs more for every millisecond it runs.

But a faster function may finish sooner.

So the cheapest configuration is not always the smallest one.

Lambda cost is shaped by how often the function is invoked, how much memory it receives, and how long each invocation runs.

### Every Errand Has a Deadline

The errand runner is built for bounded work.

A single Lambda invocation can run for a maximum of 15 minutes.

After that, the assignment has outgrown the runner.

A large finite job may belong in AWS Batch.

A continuously running service may belong in ECS.

Lambda is strongest when work arrives as an event, can scale through independent invocations, and finishes within a clear boundary.

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
| Runners working simultaneously | Concurrency | Number of invocations executing in parallel |
| Runners inside a protected fence | Reserved concurrency | Capacity reserved for one function and a cap on that function's concurrency |
| Runners already dressed and waiting | Provisioned concurrency | Pre-initialized environments that eliminate cold start |
| Size of the assignment kit | Memory allocation | Memory configured for the function; CPU and other resources scale with it |
| Calls, capacity, and time | Requests + duration at allocated memory | Major drivers of Lambda function cost |

---

## A Note From the Author

The errand runner makes every call sound immediate, isolated, and successful. Lambda invocation, retry, batching, ordering, and failure behavior depend on the event source and invocation type. Applications must be designed for duplicate processing and partial failures where those behaviors are possible.

Lambda may reuse an execution environment, but reuse is not guaranteed. Initialization code outside the handler runs once per new environment, not once per invocation — which is both a performance opportunity (cache clients outside the handler) and a source of state-related bugs if initialization side effects are assumed to be per-invocation.

Concurrency is managed within an AWS account and Region. A burst in one function can consume shared concurrency and throttle other functions. Reserved concurrency isolates capacity for a function and caps how far that function can scale. Provisioned concurrency pre-initializes environments to reduce cold-start latency, with separate charges for keeping that capacity ready.

The 15-minute limit is absolute. Lambda is not the right compute model for workflows that may run longer — Step Functions, ECS tasks, or Batch jobs are better fits depending on the shape of the work.

- [AWS Lambda invocation methods](https://docs.aws.amazon.com/lambda/latest/dg/lambda-invocation.html)
- [Lambda concurrency](https://docs.aws.amazon.com/lambda/latest/dg/lambda-concurrency.html)
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
