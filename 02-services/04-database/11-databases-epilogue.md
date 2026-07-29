---
description: "Good data architecture gives every important fact an owner, every important question a route, and every failure a response."
tags:
  - "aws"
  - "databases"
  - "epilogue"
---

# AWS Databases: Choose the Home for the Truth

> Good data architecture gives every important fact an owner, every important question a route, and every failure a response.

## The Business Goal

### One final inspection

The promotion was over.

The lines were gone.

The records office had stopped trying to be a warehouse.

The warehouse had stopped pretending to be an accounting office.

The memory desk knew it held copies.

The research wing no longer blocked the checkout line.

Before opening the completed headquarters, the owner asked the architect for one final inspection.

> “Show me where the truth lives.”

The architect did not point to a database.

She picked one order.

```text
ORDER#O900
```

> “Follow it.”

---

## The Order Enters

Leo pressed **Place order**.

The request arrived carrying:

- the customer
- the product
- the warehouse
- the final quantity
- a stable request identity

The warehouse checked the inventory condition.

```text
Decrease Quantity by 1
only if Quantity > 0
```

It created the order and protected the business invariant.

If the reply disappeared, the request identity prevented the company from treating the retry as a brand-new purchase.

This was not merely storage.

It was a promise:

> One valid order for one purchasing intent.

---

## The Order Finds Its Owner

Shreya needed the order connected to its payment, refund, fee, and merchant.

Those durable financial relationships belonged in the company’s relational books.

The order now had an authoritative context:

```text
Customer
   |
   v
Order -> Payment -> Refund
   |
   v
Merchant -> Fee -> Settlement
```

The relational system was not “better” than the warehouse.

It owned a different promise:

> Preserve connected business facts and let authorized users reason across their relationships.

---

## The Order Sends Copies

The authoritative records produced controlled changes.

Some downstream systems received purpose-built copies:

```text
Authoritative business records
            |
            +--> operational lookup view
            |
            +--> customer-facing cache
            |
            +--> search view
            |
            +--> analytical history
```

The copies helped the company answer different questions.

They did not all become owners.

The cache could expire.

The search index could lag.

The analytical copy could arrive in batches.

The operational view could be rebuilt from a durable source—if the architecture had designed that recovery path.

Every copy needed:

- an owner
- a synchronization path
- an acceptable freshness window
- failure detection
- repair or replay behavior
- access controls
- retention and deletion behavior

The headquarters had more rooms now.

It also had more responsibilities.

---

## Maya Asks the Operational Question

Maya opened her store dashboard.

> “Where is P100 available?”

The application already knew the route:

```text
PK = PRODUCT#P100
```

The warehouse returned the prepared inventory view.

When Maya reversed the question—

> “What products are in W1?”

—the second door answered:

```text
GSI1PK = WAREHOUSE#W1
```

The alternate directory might briefly lag the base record.

That was not a surprise anymore.

It was a documented promise the application had agreed to tolerate.

---

## Leo Asks the Repeated Question

Leo refreshed the product page.

The memory desk returned a cached description.

The company did not confuse the fast answer with the authoritative answer.

It knew:

- when the copy expired
- what happened on a miss
- how much staleness the page could tolerate
- what the application did if the cache was unavailable

Fast was no longer a vague ambition.

It was a bounded contract.

---

## Shreya Asks the Broad Question

At month end, Shreya opened the research wing.

> “Compare orders, refunds, fees, and settlements by merchant for the last three years.”

The analytical platform scanned the historical copy.

Customer checkouts continued.

The company had separated:

```text
operational work
small, frequent, latency-sensitive
```

from:

```text
analytical work
broad, historical, aggregation-heavy
```

The split was justified by conflicting pressure.

Not fashion.

Not a desire to collect services.

---

## Noah Turns Off a Door

The architect was not finished.

She asked Noah to simulate failure.

The cache became unavailable.

Could the application fall back without overwhelming the authoritative store?

The GSI lagged.

Could the page tolerate an older result?

A stream consumer failed.

Could it retry without duplicating the business action?

A database connection broke.

Could the application recover through the supported failover path?

One product became hot.

Could the key and cache strategy survive concentrated traffic?

The headquarters was not complete because every door worked.

It was complete when the team knew what happened when one did not.

---

## The Database Decision Canvas

The architect left one page on the owner’s desk.

For every important data workload, answer:

| Question | Why it matters |
|---|---|
| What truth is stored? | Defines the business fact |
| Which system owns it? | Identifies the authority |
| How is it retrieved? | Defines access patterns |
| Which routes are hottest? | Reveals scale and concentration |
| What consistency is required? | Defines acceptable freshness and correctness |
| What happens during concurrent updates? | Protects invariants |
| Can the fact be copied? | Establishes derived views |
| How are copies synchronized? | Defines propagation and repair |
| What happens when traffic spikes? | Defines capacity and retry behavior |
| What happens when a dependency fails? | Defines fallback and recovery |
| How is the data observed and restored? | Makes operations accountable |
| Who may access or delete it? | Defines security and lifecycle |

If these answers are unclear, the service choice is premature.

---

## The Workers We Met

### The relational accountant

```text
Preserve connected facts.
Protect relational rules.
Answer flexible SQL questions.
```

### The DynamoDB dispatcher

```text
Give me the key.
I will follow the prepared route.
Do not ask the hot path to improvise a join.
```

### The cache receptionist

```text
I remember what everyone keeps asking.
Someone else usually owns the durable truth.
```

### The search librarian

```text
Tell me the words you remember.
I will rank the documents that resemble them.
Confirm critical facts with their owner.
```

### The analytical researcher

```text
Give me history.
I will scan, group, and compare it away from checkout.
```

They are not contestants.

They are workers with different contracts.

---

## The Journey

The company began with:

> “We need a database.”

Then events made the question sharper.

| What happened | What the company learned |
|---|---|
| Every request crowded one records room | Different truths ask systems to keep different promises |
| The warehouse was organized before its questions were known | DynamoDB begins with access patterns |
| Maya brought six requests | Keys are retrieval routes, not afterthoughts |
| The sixth request arrived in reverse | An index creates another deliberate route |
| Two customers reached one item | Correctness appears under collision |
| A committed reply disappeared | Retries need idempotency |
| One product caught fire | Scale depends on traffic distribution, not table size alone |
| Akhila’s first package was rejected | Application objects must be serialized into deliberate DynamoDB operations |
| Expired cards remained overnight | Lifecycle behavior continues after the request returns |
| A damaged ledger was restored into a new room | Backups require a recovery and cutover plan |
| Shreya followed refunds through connected books | Relational systems preserve and query relationships |
| A thousand workers crowded the office door | Connection scale is different from query scale |
| Ten thousand customers repeated one question | A cache avoids reusable work |
| The popular cache card expired at once | Cache failure and stampedes need safe fallback |
| Leo remembered words but not a product key | Search is a specialized access pattern |
| The search result showed stale inventory | Derived views must confirm critical facts with their owner |
| Copies spread through headquarters | Ownership and synchronization must be explicit |

The final lesson was never a service name.

It was a way of thinking.

---

## Painkiller

> **Problem:** Teams choose a database before defining ownership, retrieval, consistency, and failure behavior.  
> **Pain:** The system looks fast in a quiet demonstration and becomes ambiguous, fragile, or expensive in production.  
> **AWS solution:** Match each workload to the data service whose contract fits, then design its keys, copies, concurrency, retries, recovery, security, and observation deliberately.

---

## Knife Cut

> **The best database is not the one with the most features. It is the one whose promises and limitations fit the work.**

---

## The Headquarters Masthead

### What Actually Just Happened

| Story lesson | Architecture lesson |
|---|---|
| Choose the room | Match the data system to the workload contract |
| Name the owner | Identify the authoritative source |
| Draw the route | Design access patterns and keys |
| Mark the copies | Define synchronization and freshness |
| Rehearse the collision | Protect concurrent business invariants |
| Repeat the request | Make retries idempotent |
| Crowd one door | Test traffic distribution and capacity |
| Turn off a room | Design fallback, recovery, and monitoring |

The owner had asked:

> “Where does the truth live?”

The answer was not:

> “Everywhere.”

The answer was:

> “In one declared home, with deliberate routes and accountable copies.”

---

## A Note From the Author

Real systems may have several authoritative sources because different bounded business domains own different facts. “One owner” means one clear authority for a particular fact within its domain, not one database for the entire company.

The headquarters also hides the difficulty of migrations, backfills, schema evolution, regional replication, compliance, audit, encryption, tenant isolation, retention, deletion, and cost allocation.

Purpose-built architecture is not permission to introduce every available service. Each boundary adds failure modes and operational work.

Use the story to remember the questions.

Use current AWS documentation, measured workload behavior, and explicit business requirements to make the production decision.

Technical references:

- [What is Amazon DynamoDB?](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/Introduction.html)
- [DynamoDB read consistency](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/HowItWorks.ReadConsistency.html)

---

## The Last Bite

A database is where a system keeps promises.

A key is how a question reaches them.

A copy is useful only when its owner remains clear.

And architecture begins when the team can say what happens after something fails.

> **The truth chooses its home.**

---

**Next section:** *[Amazon API Gateway and AWS SDK: Byte Burger's Three Zones](../05-api-sdk/00-api-gateway-and-aws-sdk-the-three-restaurant-zones.md)*

The headquarters now knows where each truth belongs.

Now customers and developers need one controlled entrance through which they can ask the application for it.
