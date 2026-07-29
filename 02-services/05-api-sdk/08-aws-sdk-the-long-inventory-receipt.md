---
description: "A paginated response is one page of a result set, not evidence that the result set ended."
tags:
  - "aws"
  - "apis"
  - "sdk"
---

# AWS SDK: The Long Inventory Receipt

> A paginated response is one page of a result set, not evidence that the result set ended.

## The Business Goal

Marco asked for every product that needed a restock review. The SDK returned one page. The code saw products, reported success, and silently ignored the marker for the next page.

Byte Burger ran out of napkins with a perfectly green job run.

## The Story

The operations assistant handed Marco a long receipt. At the bottom was a tab: “Continue here.” Each new page required the tab from the previous one.

For a customer screen, Marco showed a limited page and a clear next button. For a nightly reconciliation, he used the SDK paginator and processed every page. He did not load the entire warehouse into memory simply because it could be requested eventually.

## Meet the AWS SDK

Many AWS APIs return paginated results through continuation tokens. SDKs commonly provide paginators that request successive pages.

> **Core idea:** Code must choose whether it needs one page, a bounded user-facing page, or deliberate iteration through every page.

## How It Works

### Continuation Tokens

A response may include a token such as `NextToken`, `LastEvaluatedKey`, or service-specific continuation value. Supplying it to the next request continues the listing according to that API's contract.

Do not invent or reuse tokens across unrelated requests. They are opaque service values.

### Paginators

An SDK paginator abstracts the request-token loop. It reduces boilerplate but does not remove cost, throttling, cancellation, or memory concerns. Process pages incrementally.

### Filtering and Completeness

Prefer server-side filters and query choices that reduce unnecessary data. A client-side filter after retrieving every page may be expensive and slow.

If the result changes while pages are read, the service's consistency and pagination contract determines what can be missed or repeated. Reconciliation code should tolerate that reality.

## Architectural Mapping

```text
list request -> page + continuation token -> next request -> next page
```

Process pages incrementally rather than assuming a response is complete.

## When to Use It

Use paginators for administrative, batch, and reconciliation work that genuinely needs full traversal. Use bounded pages for interactive screens.

## When Not to Use It

Do not assume the first page is complete. Do not run an unbounded all-page traversal on a latency-sensitive request path.

## Painkiller

> **Problem:** Code treats a successful first list response as the full inventory.  
> **Pain:** Quiet omissions create incorrect reports and missing work.  
> **AWS solution:** Recognize continuation tokens, use SDK paginators deliberately, and process pages within cost and memory limits.

## Knife Cut

> **A page is a successful response. It is not necessarily a complete answer.**

## The Masthead

### What Actually Just Happened

|In the story|In AWS|What it actually means|
|---|---|---|
|Receipt page|Paginated response|Bounded set of results|
|Continue tab|Continuation token|Opaque value for next request|
|Assistant turns pages|SDK paginator|Iterates through response pages|
|Nightly stock count|Batch traversal|Deliberate full-result processing|

## A Note From the Author

Token names, page-size controls, ordering, and consistency guarantees are service-specific. Pagination avoids oversized responses; it does not guarantee a stable snapshot of changing data.

- [AWS SDKs and Tools settings reference](https://docs.aws.amazon.com/sdkref/latest/guide/settings-reference.html)

## The Last Bite

The first receipt was truthful.

It was simply not the whole story.

**Next chapter:** *[AWS SDK and Amazon S3: The Temporary Loading Dock Pass](09-aws-sdk-and-amazon-s3-the-temporary-loading-dock-pass.md)*

A customer now needed to upload a photograph without receiving a staff badge or access to the warehouse itself.
