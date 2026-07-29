---
description: "A good DynamoDB model becomes a reliable application only when code serializes data correctly, chooses the right operation, handles partial results, and asks for no more access than it needs."
tags:
  - "aws"
  - "databases"
  - "dynamodb"
---

# Amazon DynamoDB: The Developer’s Workbench

> A good DynamoDB model becomes a reliable application only when code serializes data correctly, chooses the right operation, handles partial results, and asks for no more access than it needs.

## The Business Goal

The warehouse had survived launch day.

Its lanes were sound.

Its conditions protected the final item.

Its retry rules prevented duplicate orders.

Then Akhila joined the application team.

Noah handed her the retrieval map.

> “The warehouse is ready.”

Akhila opened her editor.

> “Ready for what request?”

---

## The Story

### The first package is rejected

Akhila’s application created an ordinary object:

```json
{
  "orderId": "O901",
  "customerId": "C4821",
  "total": 89.95,
  "paid": false,
  "items": ["P100", "P204"]
}
```

She carried it to the warehouse.

The low-level intake clerk expected every value to arrive with a DynamoDB type label:

```json
{
  "orderId": { "S": "O901" },
  "customerId": { "S": "C4821" },
  "total": { "N": "89.95" },
  "paid": { "BOOL": false },
  "items": {
    "L": [
      { "S": "P100" },
      { "S": "P204" }
    ]
  }
}
```

Akhila stared at the package.

Same business object.

Different language.

The warehouse did not need another retrieval lane.

The application needed a translator.

---

## Translating the Package

DynamoDB supports several attribute families:

| Family | DynamoDB types |
|---|---|
| Scalar | String, Number, Binary, Boolean, Null |
| Document | List, Map |
| Set | String Set, Number Set, Binary Set |

Primary-key attributes must use String, Number, or Binary.

DynamoDB has no native date-time type. Applications commonly store time as an ISO 8601 string or a numeric Unix epoch value, depending on the access pattern.

Low-level SDK clients expose DynamoDB’s typed `AttributeValue` representation.

Higher-level document or enhanced clients can convert native application objects to and from DynamoDB types.

```text
application object
       |
       v
marshaller or document client
       |
       v
DynamoDB AttributeValues
       |
       v
table item
```

The abstraction changes the amount of translation code.

It does not change the stored data model.

Akhila still needed to decide:

- whether a value was a number or string
- how dates were represented
- whether a collection was a list or set
- how absent, null, and empty values differed
- how numeric precision was preserved
- whether the complete item fit within DynamoDB’s 400 KB item limit

The translator could carry the package.

It could not decide what the package meant.

---

## Four Windows at the Workbench

The warehouse workbench had four primary windows.

### Put the complete package

#### PutItem

`PutItem` writes an entire item.

```text
PutItem
PK = ORDER#O901
SK = METADATA
```

If an item already exists with the same primary key, an unconditional `PutItem` replaces it.

Akhila did not want a retry or duplicate request to overwrite an existing order.

She added a condition:

```text
attribute_not_exists(PK)
```

Now the write meant:

> “Create this order only if that key does not already exist.”

`PutItem` was not merely “save.”

Its safety depended on the condition surrounding it.

### Retrieve one known package

#### GetItem

`GetItem` retrieves one item by its complete primary key.

```text
GetItem
PK = ORDER#O901
SK = METADATA
```

Reads are eventually consistent by default.

For a table or local secondary index, the application can request a strongly consistent read when the current answer is required. Global secondary indexes do not support strongly consistent reads.

Akhila could also use a projection expression to return only selected attributes.

That reduced response payload.

It did not reduce the read-capacity calculation for the item read.

### Change part of a package

#### UpdateItem

Leo paid for the order.

Akhila needed to change:

```text
Status = PAID
PaidAt = 2026-07-27T15:04:00Z
```

She did not need to replace the whole item.

An update expression can use:

- `SET` to add or replace values
- `REMOVE` to remove attributes
- `ADD` for supported number and set operations
- `DELETE` to remove members from a set

```text
SET #status = :paid, PaidAt = :paidAt
```

`UpdateItem` can update an existing item or create one when the key does not exist.

If the application requires the item to exist, it should say so:

```text
ConditionExpression:
attribute_exists(PK)
```

The warehouse follows the instruction it receives.

It does not infer whether an upsert was intended.

### Remove one known package

#### DeleteItem

`DeleteItem` removes an item by primary key.

Like the other write operations, it can use a condition expression.

```text
Delete only if Status = CANCELLED
```

That condition prevents a cleanup path from deleting an order whose state has changed.

---

## The Reserved Word on the Label

Akhila tried to update an attribute named `Status`.

The expression parser objected.

Some attribute names conflict with DynamoDB reserved words. Others contain punctuation or paths that make direct use awkward.

Akhila gave the attribute an alias:

```text
UpdateExpression:
SET #status = :paid

ExpressionAttributeNames:
#status -> Status

ExpressionAttributeValues:
:paid -> PAID
```

Expression attribute names stand in for attribute names.

Expression attribute values stand in for values.

They make expressions safer and reusable.

They are not string interpolation.

Do not build expressions by concatenating untrusted input.

---

## Ask the Warehouse What Changed

After the update, Akhila needed the new status.

She could issue another read.

Or she could ask the write operation to return values.

For supported write operations, `ReturnValues` can return the relevant old or new attributes according to the operation’s allowed choices.

For `UpdateItem`, useful choices include:

```text
UPDATED_NEW
ALL_NEW
UPDATED_OLD
ALL_OLD
```

This can avoid a separate round trip.

Returned values from these write operations are strongly consistent with the completed write.

The application should request only what it needs.

---

## The Cart With Twenty-Five Slots

Akhila needed to write many imported products.

She chose `BatchWriteItem`.

One call can contain up to 25 put or delete operations and up to 16 MB of request data.

It cannot perform updates.

It cannot attach a condition expression to each write.

The individual puts and deletes are atomic.

The batch as a whole is not a transaction.

Some requests may return in `UnprocessedItems`.

```text
BatchWriteItem
       |
       +--> processed items
       |
       +--> UnprocessedItems
                    |
                    v
          retry with backoff
```

Akhila’s loop was not finished when the API returned successfully.

It was finished when `UnprocessedItems` was empty—or when her retry policy deliberately stopped and surfaced the remaining work.

For reads, `BatchGetItem` can retrieve up to 100 known items across one or more tables, subject to response-size and per-partition limits.

It can return `UnprocessedKeys`.

Those keys also require controlled retry.

> **Batch reduces network trips. It does not turn partial work into coordinated success.**

If several actions had to succeed or fail together, Akhila needed a transaction instead.

---

## The Ledger Stops at One Megabyte

Akhila queried Leo’s order history.

The response looked complete.

It also contained:

```text
LastEvaluatedKey
```

A `Query` result is limited to a page of up to 1 MB before filter processing.

The application continues by supplying the returned key as `ExclusiveStartKey`.

```text
first request
     |
     +--> items
     +--> LastEvaluatedKey
                 |
                 v
          next request
          ExclusiveStartKey
```

A filter expression does not make the pre-filter page larger.

It can produce a page with few—or even no—returned items while a `LastEvaluatedKey` still indicates more data.

The only safe stopping condition is the absence of another continuation key.

---

## Three Failures at the Counter

Akhila’s application encountered three failures.

They looked similar in a log.

They demanded different behavior.

### The condition failed

```text
ConditionalCheckFailedException
```

The warehouse was healthy.

The business precondition was false.

Blind retry would not make an already-sold item available again.

The application should translate this into the relevant business outcome or re-read and reconcile.

### The lane was throttled

The request exceeded available throughput or concentrated too much traffic.

The application could retry eligible throttling errors with exponential backoff and jitter while Noah investigated capacity, key distribution, or demand.

### The server returned an internal error after a write

For some server-side write failures, the application may not know whether the write took effect.

Repeating a non-idempotent write can duplicate the business action.

The application needs conditions, a stable request identity, transactional idempotency where applicable, or a read-and-reconcile path.

> **An error is not a retry instruction. It is evidence the application must classify.**

AWS SDKs provide retry behavior for many retryable errors.

Application code still owns business idempotency and what happens after the retry budget ends.

---

## The Key That Opened Every Aisle

Akhila’s function needed to read and update orders for one tenant.

The first IAM policy allowed:

```text
dynamodb:*
Resource: *
```

The function worked.

It could also open every aisle in every warehouse.

Noah narrowed the policy to:

- the required DynamoDB actions
- the intended table ARN
- index ARNs when the application queried indexes
- the partition-key values the caller was allowed to access, when fine-grained control fit the model

The `dynamodb:LeadingKeys` condition key can restrict access according to partition-key values.

Other DynamoDB condition keys can constrain selected attributes and returned data.

A policy that permits tenant-bound `Query` operations may intentionally omit `Scan`, because a scan is not naturally constrained to one leading partition key.

```text
application role
      |
      v
allowed operations
      |
      v
allowed table and indexes
      |
      v
allowed partition-key scope
```

The key design and the permission design met at the same partition key.

---

## The Wrong Way

The wrong way is to treat a high-level SDK client as proof that the data model no longer matters.

The document client can marshal values.

It cannot decide:

- whether `PutItem` should replace an item
- whether `UpdateItem` should be allowed to create one
- whether a condition failure is expected
- whether a partial batch should be retried
- whether the caller may access another tenant
- whether a 400 KB item should have been split or moved to Amazon S3

Another wrong way is to retry every exception.

Some failures describe temporary service pressure.

Some describe invalid input.

Some describe a business condition that correctly refused the request.

Reliable code knows the difference.

---

## Architectural Mapping

```text
native application object
          |
          v
serialization layer
          |
          v
specific DynamoDB operation
          |
          +--> expression and condition
          |
          +--> IAM authorization
          |
          v
response, continuation, or classified error
          |
          v
application business result
```

Every layer has a separate responsibility.

Serialization translates values.

The API operation defines the requested change.

Conditions protect state.

IAM limits authority.

Error handling protects the business outcome.

---

## When to Use Each Operation

| Application intent | Starting operation |
|---|---|
| Create or replace one complete item | `PutItem`, usually with an intentional condition |
| Retrieve one item by full primary key | `GetItem` |
| Modify selected attributes | `UpdateItem` |
| Remove one item by full primary key | `DeleteItem` |
| Retrieve related items by partition key | `Query` |
| Retrieve many known keys | `BatchGetItem` |
| Put or delete many independent items | `BatchWriteItem` |
| Coordinate several item actions atomically | `TransactWriteItems` |

The table does not choose the operation.

The application intent does.

---

## Painkiller

> **Problem:** A correct DynamoDB model can still be used incorrectly by application code.  
> **Pain:** Wrong operation semantics, unsafe replacement, broken serialization, partial batches, unbounded permissions, and indiscriminate retries create data loss or duplicate work.  
> **AWS solution:** Use the appropriate SDK abstraction and DynamoDB operation, protect writes with expressions, continue partial responses deliberately, classify errors, and grant least-privilege access.

---

## Knife Cuts

> **PutItem replaces the package. UpdateItem changes named contents.**

> **Projection reduces returned data. It does not reduce the capacity required to read the item.**

> **The SDK can retry a request. Only the application can preserve the business meaning of the retry.**

---

## The Developer’s Workbench

### What Actually Just Happened

| In the story | In DynamoDB development | What it actually means |
|---|---|---|
| Package translator | SDK marshalling or document client | Convert native values to DynamoDB attribute types |
| Complete package window | `PutItem` | Create or replace an entire item |
| Change slip | `UpdateItem` | Modify selected attributes through an update expression |
| Label aliases | Expression names and values | Safely represent attribute names and values |
| Twenty-five-slot cart | `BatchWriteItem` | Group independent puts and deletes |
| Boxes left on cart | `UnprocessedItems` or `UnprocessedKeys` | Retry only unfinished work |
| Next ledger card | `LastEvaluatedKey` | Continue a paginated response |
| Key with too many doors | Overbroad IAM policy | Authority exceeds the function’s job |

The warehouse already had good lanes.

Akhila’s job was to make every request arrive with the right shape, authority, and failure plan.

---

## A Note From the Author

SDK interfaces differ by language and version.

Some expose DynamoDB’s low-level typed values directly. Others provide document, enhanced, or object-mapping abstractions. The abstraction can reduce boilerplate, but teams must still understand how native values, empty values, sets, binary data, numeric precision, and dates are represented.

The operation limits in this article are current service constraints and should be checked before implementation. Other relevant constraints include a 400 KB maximum item size, a 1 MB `Query` result page, and transaction limits.

Fine-grained IAM conditions are powerful but do not replace careful application authorization. Policies must include the exact resources and actions the request path uses, including index resources when appropriate.

Technical references:

- [Programmatic interfaces that work with DynamoDB](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/Programming.SDKs.Interfaces.html)
- [Supported DynamoDB data types](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/HowItWorks.NamingRulesDataTypes.html)
- [Working with DynamoDB items](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/WorkingWithItems.html)
- [Error handling with DynamoDB](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/Programming.Errors.html)
- [Fine-grained access control with IAM conditions](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/specifying-conditions.html)

---

## The Last Bite

A well-designed warehouse can still receive a dangerous instruction.

Choose the operation.

State the condition.

Limit the authority.

Then decide what the application will do when the answer is incomplete.

---

**Next chapter:** *[Amazon DynamoDB: After Closing Time](06-dynamodb-after-closing-time.md)*

Akhila can now operate every window at the workbench.

That night, the first reservation expires, a ledger is damaged, and a distant warehouse asks for the same truth.
