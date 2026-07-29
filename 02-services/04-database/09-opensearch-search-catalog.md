---
description: "OpenSearch creates a search-oriented view of data so applications can find documents by words, relevance, filters, and aggregations instead of exact database keys."
tags:
  - "aws"
  - "databases"
  - "opensearch"
---

# Amazon OpenSearch Service: The Search Catalog

> OpenSearch creates a search-oriented view of data so applications can find documents by words, relevance, filters, and aggregations instead of exact database keys.

## The Business Goal

The warehouse could find:

```text
PRODUCT#P100
```

The memory desk could return it instantly.

Then Leo asked:

> “Show me the quiet wireless mouse with the blue wheel.”

Nobody knew the product number.

The dispatcher could follow an address.

Leo had brought a description.

---

## The Story

### The catalog organized by words

Maya searched every product description.

Wireless Mouse P100 contained:

```text
quiet
wireless
blue wheel
ergonomic
```

Gaming Mouse P204 contained:

```text
wireless
blue lighting
high precision
```

The architect built a catalog that recorded which words appeared in which product documents.

```text
quiet ---------> P100

wireless ------> P100, P204

blue ----------> P100, P204

wheel ---------> P100
```

Leo’s words no longer needed to equal one primary key.

The catalog found candidate documents, scored their relevance, and returned the closest matches.

The source records stayed in their authoritative homes.

The catalog held a purpose-built searchable view.

> **A database key finds the address you know. A search index helps discover the address you do not.**

---

## Meet Amazon OpenSearch Service

Amazon OpenSearch Service is a managed service for deploying, operating, and scaling OpenSearch search and analytics workloads.

It supports search-oriented access patterns such as:

- full-text search
- relevance scoring
- filters
- aggregations
- log and event analytics
- document-oriented exploration
- vector search in supported configurations

OpenSearch Service manages infrastructure according to the selected domain or serverless deployment.

The application team still owns:

- document shape
- index mappings
- ingestion
- synchronization with authoritative systems
- queries
- shard and replica strategy where applicable
- access control
- data lifecycle
- monitoring and cost

OpenSearch makes search possible.

It does not decide what the search result is allowed to promise.

---

## Documents Enter the Catalog

The team transformed each product into a JSON document:

```json
{
  "productId": "P100",
  "name": "Wireless Mouse",
  "description": "Quiet ergonomic mouse with blue scroll wheel",
  "category": "accessories",
  "price": 29.95,
  "inStock": true
}
```

The document entered an OpenSearch index.

An index is a searchable collection of documents.

Mappings describe how fields are interpreted:

- text to analyze for full-text search
- keywords for exact filtering and aggregation
- numbers and dates for ranges
- Boolean fields for exact state
- other supported field types for specialized queries

The same human word may need more than one representation.

For example:

- `description` as analyzed text for search
- `category` as a keyword for exact filtering

If everything becomes full text, exact filters and aggregations become awkward.

If everything becomes a keyword, language-aware search disappears.

The mapping is the contract between the document and the questions people will ask.

---

## Analysis Changes the Words

Before indexing text, OpenSearch analysis can transform it.

Conceptually:

```text
"Quiet wireless mice"
          |
          v
       tokenizer
          |
          v
quiet | wireless | mice
          |
          v
 token filters and normalization
```

Analyzers can lowercase terms, split text, remove selected words, or apply language-specific processing.

The analyzer used at search time must remain compatible with how terms were indexed.

Otherwise, the catalog and the customer may speak different dialects.

---

## Relevance Is Not Equality

Leo searched:

```text
quiet blue wireless mouse
```

OpenSearch did not merely return `true` or `false`.

It scored matching documents.

P100 matched “quiet,” “blue,” “wireless,” and “mouse.”

P204 matched fewer useful terms.

Search results can combine:

- full-text matching
- relevance scoring
- exact filters
- sorting
- ranges
- aggregations

```text
text match: quiet wireless mouse
filter: inStock = true
filter: price <= 50
aggregate: count by category
```

The text query asks:

> “Which documents resemble this request?”

The filter asks:

> “Which candidates satisfy this exact condition?”

They are different kinds of questions inside one search.

---

## The Catalog Lags the Shelf

Maya sold the final P100.

The authoritative inventory record changed immediately.

The search catalog still showed:

```text
inStock = true
```

The owner objected.

> “Search returned a product we cannot sell.”

The catalog was a derived copy.

The update pipeline had not reached it yet.

```text
authoritative data changes
          |
          v
event, stream, or ingestion pipeline
          |
          v
OpenSearch document update
          |
          v
searchable view becomes current
```

OpenSearch also has index refresh behavior before a newly indexed change becomes visible to search.

The application must decide:

- how quickly updates flow
- how failed updates are retried
- how duplicates are handled
- whether search results require confirmation against the source
- how the index is rebuilt

For the final-item checkout, the application should confirm availability through the authoritative operational system.

Search discovers the product.

The source of truth decides whether it can be sold.

---

## Bulk Loading the Catalog

The product team imported one million documents.

Sending one indexing request at a time created unnecessary overhead.

OpenSearch supports bulk requests that group index, create, update, or delete actions.

```text
bulk request
    |
    +--> P100 indexed
    +--> P101 indexed
    +--> P102 failed
    +--> P103 indexed
```

A successful HTTP response does not mean every item inside the bulk request succeeded.

The application must inspect item-level results and retry only failed operations according to the failure.

This is the same lesson the warehouse learned from DynamoDB batches:

> **Transport success is not the same as complete business success.**

---

## Shards Divide the Catalog

An OpenSearch index is divided into primary shards.

Replica shards maintain additional copies.

```text
index
  |
  +--> primary shard 0 --> replica
  +--> primary shard 1 --> replica
  +--> primary shard 2 --> replica
```

Primary shards distribute indexed documents.

Replicas improve resilience and can serve search traffic.

Too few shards can constrain growth and parallelism.

Too many small shards consume memory and coordination overhead.

Shard decisions depend on data size, ingestion rate, query behavior, node capacity, and lifecycle.

This is why OpenSearch belongs at the recognition-and-selection level for many application developers:

They need to know when search is the right access pattern and that indexes carry operational cost.

They do not need to pretend a search cluster is a key-value table.

---

## The Catalog Needs Restricted Shelves

OpenSearch Service supports several layers of access control, including:

- identity-based IAM policies
- domain resource policies
- VPC placement
- fine-grained access control
- encryption at rest
- node-to-node encryption
- TLS for client traffic

Control-plane permissions manage the domain.

Data-plane permissions govern access to indexes and OpenSearch HTTP APIs.

A policy allowing someone to describe a domain is not automatically permission to search every document.

An application also needs a request-signing or supported authentication path when the access policy requires an AWS principal.

Search documents can contain the same sensitive fields as the source.

Derived does not mean harmless.

---

## OpenSearch Is Not the Authoritative Checkout

The owner loved the catalog.

> “It can store the product document. Let it own inventory too.”

The architect refused.

OpenSearch can persist indexed documents.

That does not make it the best authoritative store for every business workflow.

A checkout may require:

- conditional inventory updates
- transactions
- strict ownership
- deterministic key access
- recovery from failed business operations

OpenSearch is optimized for search and analytics.

Use it as the authority only when the application has explicitly chosen and validated that contract.

For the product catalog, the safer pattern was:

```text
authoritative product and inventory systems
                 |
                 v
        searchable OpenSearch view
                 |
                 v
        customer discovers product
                 |
                 v
 authoritative checkout confirms purchase
```

---

## The Wrong Way

The wrong way is to use a DynamoDB Scan or a relational wildcard query as a substitute for a search index on a large, search-heavy application path.

Another wrong way is to use OpenSearch for an exact primary-key lookup merely because it can return the document.

Choose by the question:

| Question | Natural starting point |
|---|---|
| Get item with known key | DynamoDB or the authoritative database |
| Join connected relational records | RDS or Aurora |
| Return a reusable hot copy | ElastiCache or DAX |
| Find documents by words and relevance | OpenSearch |

The ability to produce an answer does not prove the system is the right place to ask.

---

## Architectural Mapping

```text
authoritative data
        |
        v
ingestion or change pipeline
        |
        v
OpenSearch index
        |
        +--> text analysis
        +--> shards and replicas
        |
        v
search, filters, relevance, aggregations
        |
        v
application confirms critical action with source
```

The index is a prepared search view.

Its value comes from answering questions the source was not organized to answer efficiently.

---

## When to Use OpenSearch

Use OpenSearch when:

- full-text search is a primary access pattern
- relevance ranking matters
- users search incomplete or descriptive terms
- filters and aggregations accompany search
- log or event exploration needs a search-oriented engine

Consider another store when:

- the application already knows the exact key
- relational transactions are central
- a simple cache solves the repeated-read problem
- the team cannot justify index synchronization and operations

---

## Painkiller

> **Problem:** Customers describe what they want without knowing the database key.  
> **Pain:** Exact-key stores and relational systems may require broad, awkward, or expensive search paths for relevance-oriented questions.  
> **AWS solution:** OpenSearch Service maintains a purpose-built document index for text search, relevance, filters, and aggregations while the application preserves authoritative ownership elsewhere.

---

## Knife Cut

> **The database owns the fact. The search index owns a searchable view of the fact.**

---

## The Search Catalog

### What Actually Just Happened

| In the story | In OpenSearch | What it actually means |
|---|---|---|
| Product catalog | Index | Searchable collection of documents |
| Product card | Document | JSON record indexed for search |
| Word directory | Inverted index | Terms mapped to matching documents |
| Catalog language rules | Analyzer | Tokenization and normalization |
| Best matching products | Relevance score | Ranked search results |
| Exact catalog constraints | Filters | Non-scoring conditions |
| Catalog copying shelf changes | Ingestion pipeline | Synchronization from authoritative data |
| Catalog sections | Shards | Distributed pieces of an index |
| Duplicate catalog sections | Replicas | Resilience and search capacity |

Leo did not know the address.

The catalog turned his words into a path.

---

## A Note From the Author

The inverted-index explanation is deliberately simplified. OpenSearch query behavior depends on mappings, analyzers, query types, scoring, refresh behavior, shards, replicas, and cluster health.

Amazon OpenSearch Service offers both managed domains and serverless collections. Their capacity and operational models differ.

OpenSearch can support advanced analytics and vector search, but those features should enter a story only when the application actually needs them.

Index synchronization is application architecture. If the index is derived from another source, teams need replay, backfill, deletion, schema evolution, and failure-repair plans.

Technical references:

- [Amazon OpenSearch Service documentation](https://docs.aws.amazon.com/opensearch-service/)
- [Indexing data in OpenSearch Service](https://docs.aws.amazon.com/opensearch-service/latest/developerguide/indexing.html)
- [Operational best practices for OpenSearch Service](https://docs.aws.amazon.com/opensearch-service/latest/developerguide/bp.html)
- [Access control in OpenSearch Service](https://docs.aws.amazon.com/opensearch-service/latest/developerguide/ac.html)

---

## The Last Bite

A key answers:

> “Where is P100?”

A search index answers:

> “Which product sounds like what the customer described?”

Build the catalog when discovery becomes the question.

---

**Next chapter:** *[AWS Databases: The Right Room for the Question](10-aws-databases-comparison.md)*

The headquarters now has a warehouse, accounting office, memory desk, and search catalog.

The final choice is no longer which service is best—but which promise the waiting work requires.
