---
description: "A DynamoDB secondary index creates another key-based route to existing table data."
tags:
  - "aws"
  - "databases"
  - "dynamodb"
---

# Amazon DynamoDB: The Second Door Into the Warehouse

> A DynamoDB secondary index creates another key-based route to existing table data.

## The Business Goal

Five request cards had lanes.

The sixth sat alone on the desk:

```text
Find every product stored in warehouse W1.
```

The warehouse was organized in the other direction:

```text
PRODUCT#P100
        |
        +--> WAREHOUSE#W1
        +--> WAREHOUSE#W2
```

The facts were inside.

The question was standing at the wrong door.

---

## The Story

### A truck waits at the product entrance

A delivery truck arrived at Warehouse W1.

The driver handed Maya a list.

> “Before I unload, show me every product already stored here.”

Maya walked to the product entrance.

The directory could answer:

> “Where is product P100 stored?”

It could not answer:

> “What products are stored in W1?”

She could inspect every product route and keep only the records mentioning W1.

The truck would wait while she searched the building.

Or the team could build another entrance.

### The second directory

The architect hung a second directory beside a new door.

The original directory remained organized by product:

```text
Product P100
    |
    +--> Warehouse W1
    +--> Warehouse W2
```

The new directory was organized by warehouse:

```text
Warehouse W1
    |
    +--> Product P100
    +--> Product P200
```

The inventory did not become a new business fact.

The route changed.

> **The item is the fact. The index is another road to the fact.**

---

## Meet the Global Secondary Index

A Global Secondary Index, or GSI, can use a key schema different from the base table.

The inventory item carried both routes:

```text
Base-table route
PK     = PRODUCT#P100
SK     = WAREHOUSE#W1

GSI route
GSI1PK = WAREHOUSE#W1
GSI1SK = PRODUCT#P100
```

```text
                   One inventory item
                          |
             +------------+------------+
             |                         |
             v                         v
     Base-table route              GSI route
     PRODUCT#P100                  WAREHOUSE#W1
           |                             |
           v                             v
     WAREHOUSE#W1                  PRODUCT#P100
```

Now Maya could query:

```text
GSI1
GSI1PK = WAREHOUSE#W1
```

The second door answered the sixth request without scanning the entire table.

---

## The Directory Does Not Carry Every Detail

The driver reached the counter.

> “I need the product name, quantity, and status.”

The architect had to decide what information the second directory should contain.

That decision is called projection.

A GSI projection can include:

- index and base-table keys only
- selected non-key attributes
- all table attributes

For the warehouse page, the team projected:

```text
ProductName
Quantity
Status
```

Now the index query could return the complete warehouse listing.

Projecting too little could require additional reads.

Projecting everything could consume unnecessary storage and write resources.

The directory should carry what the request needs—not a copy of the entire warehouse by reflex.

---

## The New Door Briefly Lags

While Maya watched, a worker moved the last P200 dock out of W1.

The base inventory record changed first.

For a brief moment, the second directory still listed it.

> “Your new door is wrong,” Maya said.

> “It is catching up,” the architect replied.

GSI updates propagate asynchronously from the base table.

```text
Base-table write
       |
       v
asynchronous index update
       |
       v
GSI becomes current
```

GSI reads are eventually consistent.

They do not support strongly consistent reads.

This makes a GSI a poor route when the application must immediately confirm the latest write through that alternate access path.

The route is fast and useful.

Its promise includes possible lag.

---

## A Different Shelf Order Inside the Same Aisle

Later, Noah asked:

> “Inside product P100, can we arrange warehouses by quantity?”

This did not require a door organized around a different partition grouping.

The product aisle could remain:

```text
PK = PRODUCT#P100
```

But it needed another ordering.

That is the role of a Local Secondary Index, or LSI.

```text
Base table
PK = PRODUCT#P100
SK = WAREHOUSE#W1

LSI
PK = PRODUCT#P100
alternate sort key = QUANTITY#000120
```

| Question | GSI | LSI |
|---|---|---|
| Can the partition key differ from the table? | Yes | No |
| Does it use an alternate sort key? | It can | Yes |
| Can it be added after table creation? | Yes | No |
| Can reads be strongly consistent? | No | Yes |
| What does it usually provide? | A different grouping or route | Another ordering inside the same grouping |

An LSI must be defined when the table is created.

A GSI can be added later.

---

## The Low-Stock Bell

Maya did not want every inventory item in another directory.

She wanted only items that needed attention.

The team gave index keys only to low-stock items:

```text
Low-stock item
GSI2PK = LOW_STOCK#W1
GSI2SK = PRODUCT#P200
```

Normal-stock items omitted those attributes.

```text
Base table: all inventory
GSI2: only inventory carrying low-stock index keys
```

This produced a sparse index.

Absence became part of the design.

When inventory crossed the threshold, the application had to add or remove the index-key attributes correctly.

The index could only reflect the state written to the item.

---

## Every Door Creates Work

The owner loved the new entrance.

> “Build ten more.”

The architect handed him the construction bill.

Every relevant base-table write might now update several index structures:

```text
Write base item
    |
    +--> update GSI1
    +--> update GSI2
    +--> update GSI3
```

Indexes add:

- storage
- write work
- projection decisions
- propagation behavior
- capacity and throttling considerations
- operational complexity

A GSI can also create back-pressure. In provisioned mode, insufficient GSI write capacity can throttle writes to the base table.

The new door solved a real request.

That did not make it free.

---

## Opening a Door After the Warehouse Is Full

The team added the warehouse directory after inventory already filled the table.

DynamoDB had to build the new index from existing items.

During this backfill:

- the index was created asynchronously
- normal table writes could continue
- the team needed to watch capacity and throttling
- the index became available after reaching the active state

This was renovation while the warehouse remained open.

It was possible.

It still deserved a plan.

---

## The Wrong Way

The wrong way is to build an index because an attribute looks interesting.

```text
We have Status.
We have Region.
We have CreatedAt.
Let us index all of them.
```

Indexes exist to serve deliberate access patterns.

Another wrong way is to treat a filter as a substitute for a key condition.

Filtering can reduce returned results after DynamoDB reads the candidate items.

It does not build a new retrieval route.

And the most dangerous wrong way is to promise:

> “The GSI will always show the write immediately.”

That is not the consistency contract a GSI offers.

---

## Architectural Mapping

```text
Base-table write
        |
        +--> authoritative table item
        |
        +--> asynchronous GSI maintenance
                    |
                    v
             alternate query route
```

The application writes to the table.

DynamoDB maintains the index.

The application queries the index using its alternate key, but must design for its projection, cost, and consistency behavior.

---

## When to Use an Index

Create a secondary index when:

- an important access pattern cannot use the base primary key
- the alternate key is known at request time
- the projected data can satisfy the request economically
- eventual consistency is acceptable for a GSI
- the additional write and storage cost is justified

## When Not to Use an Index

Reconsider when:

- the request is rare and a controlled batch process is simpler
- the index would concentrate traffic on a poor key
- immediate strong consistency is required through a GSI
- the projection or write amplification outweighs the benefit
- the index exists only because the table was designed before its access patterns

---

## Painkiller

> **Problem:** The base primary key answers one direction of a relationship but not another.  
> **Pain:** The missing route forces broad reads or extra application work.  
> **AWS solution:** A GSI or LSI creates another key structure for a deliberate access pattern, with explicit consistency, projection, and cost trade-offs.

---

## Knife Cut

> **A GSI can create a new grouping. An LSI creates another order within the original partition-key grouping.**

---

## The Second Door

### What Actually Just Happened

| In the story | In DynamoDB | What it actually means |
|---|---|---|
| Product directory | Base table | Primary access path |
| Warehouse directory | GSI | Alternate partition and sort-key route |
| Details printed in directory | Projection | Attributes copied into the index |
| Directory briefly behind | Eventual consistency | GSI may lag the base-table write |
| Alternate shelf order | LSI | Same partition key, different sort key |
| Low-stock list | Sparse index | Only items with index keys appear |
| Construction bill | Write amplification | Base writes maintain index entries |
| Renovation while open | Backfill | Existing items are added to a new GSI asynchronously |

The warehouse did not move the fact.

It gave an important question another way to reach it.

---

## A Note From the Author

The directory analogy can hide important behavior.

An index is a maintained data structure, not a pointer painted on a wall. Its entries consume storage and write resources. A GSI receives asynchronous updates and supports eventually consistent reads. An LSI shares the table’s partition-key grouping and must be created with the table.

Projection determines which non-key attributes are available directly from an index. Base-table primary-key attributes are also projected so an index entry can identify its source item.

Real designs must also account for quotas, key distribution, permissions, monitoring, deployment safety, and the effect of index throttling on table writes.

Technical references:

- [Using Global Secondary Indexes in DynamoDB](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/GSI.html)
- [DynamoDB read consistency](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/HowItWorks.ReadConsistency.html)

---

## The Last Bite

A secondary index is a promise made in advance:

> “This question matters enough to deserve its own door.”

Build the door.

Pay for the door.

And remember what kind of answer can walk through it.

---

**Next chapter:** *[Amazon DynamoDB: The Warehouse Under Pressure](04-dynamodb-under-pressure.md)*

The retrieval lanes are ready.

Then the promotion begins, two customers reach the last pair of shoes, and every door fills at once.
