---
description: "A DynamoDB schema begins as a list of application requests and becomes a map of keys."
tags:
  - "aws"
  - "databases"
  - "dynamodb"
---

# Amazon DynamoDB: Keys, Access Patterns, and the Retrieval Map

> A DynamoDB schema begins as a list of application requests and becomes a map of keys.

## The Business Goal

The retrieval lanes worked.

Give the warehouse a product number and warehouse number, and a worker could walk straight to the answer.

The owner admired the first lane.

> “Excellent. Build the rest of the warehouse.”

Noah stared at the empty floor.

> “Around what?”

---

## The Story

### Maya brings six questions

Maya arrived carrying six customer request cards.

She laid them on the dispatch desk.

```text
A1  Find one physical item by serial number.

A2  Find one product in one warehouse.

A3  Find every warehouse carrying one product.

A4  Find every product stored in one warehouse.

A5  Find every order belonging to one customer.

A6  Find one order directly by order ID.
```

The owner reached for a ruler.

> “How many product shelves should we build?”

The architect stopped him.

Products were not the design yet.

Warehouses were not the design.

Orders were not the design.

The requests were the design.

### The team marks the important details

Noah picked up the first card.

> “How often will this arrive?”

> “Constantly,” Maya said.

> “Can the answer be briefly stale?”

> “Not when a customer is buying the final item.”

They added two columns to the cards.

| ID | Application request | Expected volume | Consistency need |
|---|---|---:|---|
| A1 | Get item by serial number | Very high | Strong may matter |
| A2 | Get product inventory at warehouse | Very high | Strong may matter |
| A3 | List warehouses carrying product | High | Eventual may be enough |
| A4 | List products in warehouse | High | Eventual may be enough |
| A5 | List orders for customer | High | Eventual may be enough |
| A6 | Get order by order ID | High | Strong may matter |

This worksheet mattered more than the first drawing of the table.

It captured not only what the application would ask, but how much pressure each question would create and what promise the answer had to keep.

---

## The First Address

A repair worker arrived with a broken mouse.

> “Serial number SN-98471.”

There was no need to retrieve a group.

One serial number should identify one physical item.

The team wrote:

```text
PK = SERIAL#SN-98471
SK = METADATA
```

The request could use `GetItem`.

```text
serial number
      |
      v
full primary key
      |
      v
one physical item
```

The address existed because A1 existed.

---

## The Product Aisle

Maya lifted the next two cards.

```text
A2  Find P100 in W1.
A3  Find every warehouse carrying P100.
```

The questions were related.

The team placed all warehouse inventory for one product behind the same partition-key value:

```text
PK = PRODUCT#P100

SK = WAREHOUSE#W1
SK = WAREHOUSE#W2
SK = WAREHOUSE#W3
```

Now the precise request could use the full key:

```text
GetItem
PK = PRODUCT#P100
SK = WAREHOUSE#W1
```

And the broader request could query the product group:

```text
Query
PK = PRODUCT#P100
SK begins_with WAREHOUSE#
```

One grouping served two planned questions.

That was not an accident.

That was the model.

---

## The Customer Order Corridor

Leo opened his account page.

> “Show me my recent orders.”

The team gave each customer an order corridor:

```text
PK = CUSTOMER#C4821

SK = ORDER#2026-07-26#O900
SK = ORDER#2026-07-21#O844
SK = ORDER#2026-06-18#O711
```

The sort key carried a useful hierarchy.

```text
ORDER
  |
  +--> 2026
        |
        +--> 07
              |
              +--> 26
                    |
                    +--> O900
```

That shape allowed requests such as:

```text
all orders for customer C4821

orders beginning with 2026-07

orders between two sort-key values
```

Composite sort-key values can turn an ordered string into a navigable path.

Modern DynamoDB key schemas can also support multiple key attributes in supported configurations. The durable design lesson is the same: the key must match the retrieval pattern and preserve useful ordering or grouping.

---

## The Order That Needed Two Addresses

Leo called support.

> “My order number is O900.”

The customer corridor could find the order—but only if support already knew Leo’s customer ID.

The team needed both:

```text
customer -> orders
```

and:

```text
order ID -> order
```

They stored an order summary under the customer route and a direct lookup under the order route:

| PK | SK | Type | Important attributes |
|---|---|---|---|
| `CUSTOMER#C4821` | `ORDER#2026-07-26#O900` | CustomerOrder | total, status |
| `ORDER#O900` | `METADATA` | OrderLookup | customer ID, total, status |

The owner pointed at the two records.

> “The order appears twice.”

> “Because the application asks for it in two different ways,” the architect said.

Duplication was not automatically good.

It was justified by two explicit access patterns.

The team still needed an ownership and synchronization plan.

---

## The Retrieval Map

Noah pinned the completed routes to the wall.

| Request | Partition key | Sort-key condition | Operation |
|---|---|---|---|
| Get serial item | `SERIAL#id` | `METADATA` | GetItem |
| Get product at warehouse | `PRODUCT#id` | exact `WAREHOUSE#id` | GetItem |
| List product warehouses | `PRODUCT#id` | begins with `WAREHOUSE#` | Query |
| List customer orders | `CUSTOMER#id` | begins with `ORDER#` | Query |
| Get order directly | `ORDER#id` | `METADATA` | GetItem |

Five requests had direct routes.

One card remained on the desk.

```text
A4  Find every product stored in warehouse W1.
```

The base table could answer:

```text
product -> warehouses
```

It could not directly answer:

```text
warehouse -> products
```

The facts existed.

The route did not.

---

## The Temptation to Search Every Shelf

The owner shrugged.

> “The products are somewhere in the warehouse. Just look through everything.”

A worker began a Scan.

He inspected products in San Jose.

Then Reno.

Then Portland.

He discarded every record that did not mention W1.

The filter made his final clipboard shorter.

It did not make his walk shorter.

This was the warning:

> **Data existing in the table does not mean the application has an efficient way to retrieve it.**

The missing request did not need a clever filter.

It needed another door.

---

## Key Design Under Traffic

Before construction began, Noah checked the proposed route labels.

A weak route grouped almost everything together:

```text
PK = STATUS#AVAILABLE
```

One status value could receive enormous traffic.

A better route offered many routing values:

```text
PK = PRODUCT#P100
PK = PRODUCT#P101
PK = PRODUCT#P102
```

But even a high-cardinality design could contain one celebrity product that became disproportionately hot.

Key design therefore asks two questions:

1. Does the key answer the request?
2. Does the key distribute the expected traffic?

Sometimes spreading writes across sharded key values can relieve concentration:

```text
LOW_STOCK#00
LOW_STOCK#01
LOW_STOCK#02
```

The trade-off is that readers may need to query several shards and combine the results.

More distributed writes can create more complicated reads.

---

## The Wrong Way

The wrong way is to begin with nouns alone:

```text
Products table
Warehouses table
Inventory table
Orders table
```

That can reproduce a relational schema while removing the joins that made it useful.

Another wrong way is to invent keys and indexes first, then search for questions they might answer.

Every route should be able to finish this sentence:

> “This exists because the application must retrieve...”

If the team cannot finish the sentence, it is designing decoration.

---

## Architectural Mapping

```text
Application requests
        |
        v
Access-pattern worksheet
        |
        v
Primary-key routes
        |
        +--> exact item
        +--> related item collection
        +--> ordered or ranged subset
        |
        v
Missing request discovered before launch
```

The worksheet is not merely documentation.

It is the specification the key design must satisfy.

---

## When to Use This Approach

Write the retrieval map before building the table when:

- the workload has known operational requests
- several item types must be retrieved together
- the application needs predictable key-based access
- the model may deliberately duplicate facts
- traffic distribution affects the key choice

## When It Is Not Enough

The worksheet does not solve:

- unknown future analytical questions
- flexible joins across arbitrary relationships
- synchronization of duplicated facts
- concurrency and retry behavior
- every alternate lookup with the base primary key

Those limitations are not reasons to skip the worksheet.

They are the questions the worksheet exposes early.

---

## Painkiller

> **Problem:** A DynamoDB table feels impossible to design when the team begins with entities instead of application requests.  
> **Pain:** Missing retrieval paths lead to scans, extra calls, hot keys, and redesign under load.  
> **AWS solution:** Write an access-pattern matrix first, then assign primary keys and deliberate alternate routes to each important request.

---

## Knife Cut

> **A relational schema begins with nouns and relationships. A DynamoDB schema begins with requests and routes.**

---

## The Retrieval Map

### What Actually Just Happened

| In the story | In DynamoDB | What it actually means |
|---|---|---|
| Maya’s request cards | Access patterns | Required application reads and writes |
| A direct address | Primary key | Main retrieval route |
| Product aisle | Shared partition-key value | Related item collection |
| Shelf path | Sort key | Ordering, range, and hierarchy inside a group |
| Order in two locations | Deliberate duplication | Two access patterns served by prepared records |
| Crowded route | Hot key risk | Too much traffic concentrated on one key value |
| Missing reverse request | Unsupported access pattern | A question the base key cannot answer directly |

The team did not draw a schema and hope it answered Maya’s questions.

It wrote the questions first.

Then it made every key defend its place on the floor.

---

## A Note From the Author

The retrieval map is intentionally simplified.

Real DynamoDB design must also account for item size, pagination, consistency, authorization, capacity, quotas, global replication, backups, recovery, cost, and the failure behavior of duplicated writes.

Synthetic key strings are a common modeling technique, but they are not the only key-schema option. Regardless of representation, the important properties remain stable: the application must know the retrieval route, sort semantics must be intentional, and traffic must distribute safely.

Sharding is not a free fix. It spreads writes by making reads fan out.

Technical references:

- [Core components of Amazon DynamoDB](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/HowItWorks.CoreComponents.html)
- [Multi-attribute key schemas](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/GSI.DesignPattern.MultiAttributeKeys.html)

---

## The Last Bite

DynamoDB keys are not labels applied after the design.

They are the design.

Write the question.

Then build the address that answers it.

---

**Next chapter:** *[Amazon DynamoDB: The Second Door Into the Warehouse](03-dynamodb-indexes-second-door.md)*

Five request cards now have direct lanes.

The sixth asks the same inventory question in reverse—and the warehouse has no entrance for it.
