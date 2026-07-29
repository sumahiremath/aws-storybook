---
description: "Give applications a typed GraphQL API for precise data operations and real-time updates to connected clients."
tags:
  - "aws"
  - "application-integration"
  - "event-driven"
  - "appsync"
---

# AWS AppSync: The Order in Your Pocket

> Give applications a typed GraphQL API for precise data operations and real-time updates to connected clients.

## The Business Goal

The mobile application called one endpoint for the menu.

Another endpoint returned prices.

Another created orders.

Another returned delivery status.

Every two seconds, every open phone asked:

> “Is my order ready now?”

Most answers said no.

The application downloaded fields it did not display and generated traffic merely to discover that nothing had changed.

---

## The Story

Nia replaced the scattered counters with one typed ordering desk.

The customer asked for exactly the menu fields the screen needed.

The customer placed an order through a declared operation.

Then the phone opened a private status channel for order 42.

When the backend changed the order from `PREPARING` to `READY`, the connected phone received the selected status fields.

The customer stopped asking every two seconds.

Byte Burger spoke when the relevant data changed.

---

## The Wrong Way

GraphQL does not automatically make every backend efficient.

A poorly designed resolver can make many downstream calls, expose excessive data, or allow expensive queries. Real-time subscriptions do not replace authorization, reconnection, or current-state recovery.

And an AppSync subscription is not the same as an SMS or operating-system mobile push notification delivered while the application is closed.

---

## Meet the AWS Service

> **Core idea:** AWS AppSync provides managed GraphQL and real-time APIs that connect application operations to configured resolvers and data sources.

A GraphQL schema defines types and operations. Resolvers implement fields against data sources such as DynamoDB, Lambda, OpenSearch, HTTP endpoints, and supported relational integrations.

AWS manages API request handling and real-time connection infrastructure. You manage schema design, resolvers, authorization, data sources, limits, caching, conflict behavior, client recovery, and business logic.

---

## How It Works

### The Typed Menu

#### GraphQL Schema

The schema defines object types, fields, arguments, and the available root operations.

Clients can request selected fields, but the server still controls which fields exist and which identities may access them.

### “Show My Order”

#### Query

A query reads data:

```graphql
query GetOrder {
  getOrder(id: "42") {
    id
    status
    estimatedReadyAt
  }
}
```

GraphQL field selection reduces unnecessary response data. It does not guarantee fewer backend operations unless resolvers are designed accordingly.

### “Place This Order”

#### Mutation

A mutation requests a change such as `createOrder` or `updateOrderStatus`.

Resolvers validate and authorize the operation before reading or writing the configured data source or invoking business logic.

### The Staff Behind the Counter

#### Resolvers and Data Sources

A resolver connects a schema field to a data source and transforms request and response data.

Pipeline resolvers can perform a sequence of functions. Direct Lambda resolvers can batch supported requests to reduce invocation overhead. Resolver errors should return useful, sanitized GraphQL errors without exposing sensitive internals.

### “Tell Me When Mine Changes”

#### Subscription

GraphQL subscriptions establish secure WebSocket connections for real-time updates.

AppSync GraphQL subscriptions are triggered by associated mutations. The mutation's selected result fields constrain what can be sent to subscribers.

Subscription arguments or enhanced filters can narrow delivery, such as order ID and authorized customer identity.

### Private Order Channels

#### Authorization

AppSync supports authorization modes including IAM, Cognito user pools, OpenID Connect, Lambda authorization, and API keys for appropriate cases.

Field- and resolver-level rules must ensure customer 42 cannot subscribe to customer 99's order merely by changing an argument.

### Reconnecting After the Tunnel

#### Sync, Versioning, and Conflict Resolution

A disconnected client can miss live transitions. On reconnect, it should query or synchronize current state instead of assuming it received every update.

For supported versioned DynamoDB data sources, AppSync sync operations and conflict detection can help clients retrieve changes and handle concurrent updates. Conflict strategies include optimistic concurrency, automerge, and Lambda-based resolution.

Client libraries may add offline storage and synchronization behavior. The application must still decide what offline actions are safe and how conflicts affect the business.

### App Screen or Phone Notification

#### AppSync versus SNS

AppSync subscriptions update a connected application through its real-time API.

SNS can participate in SMS, mobile push, email, or other subscribed notification delivery. An `OrderReady` fact can drive both experiences without pretending they are the same channel.

---

## Architectural Mapping

```text
Customer app
   | query / mutation
   v
AppSync GraphQL API
   |
 resolver
   v
data source / business service
   |
status mutation
   v
AppSync subscription --> connected customer app
```

Monitor resolver latency, errors, throttling, real-time connections, messages, authorization failures, and downstream data-source behavior.

---

## When to Use It

Use AppSync when:

- Clients benefit from a typed GraphQL schema
- Screens need precise field selection
- One API composes supported data sources
- Connected clients need filtered real-time updates
- Client synchronization and conflict behavior fit the data model

## When Not to Use It

Use a simpler REST or HTTP API when GraphQL flexibility and subscriptions add no value. Use SNS or another notification service for external notification channels rather than treating a live WebSocket subscription as guaranteed background mobile delivery.

---

## Painkiller

> **Problem:** Client applications call scattered endpoints and poll repeatedly for order changes.  
> **Pain:** The client overfetches data, generates empty traffic, and embeds backend structure.  
> **AWS solution:** AppSync exposes typed GraphQL operations and pushes relevant mutation results to authorized connected subscribers.

---

## Knife Cut

> **A query asks for current state. A mutation requests a change. A subscription receives selected future changes while connected.**

---

## The Masthead

### What Actually Just Happened

|In the story|In AppSync|What it actually means|
|---|---|---|
|Typed ordering desk|GraphQL schema|Defined types and operations|
|Menu request|Query|Read selected fields|
|Place order|Mutation|Request a data change|
|Staff behind the counter|Resolver|Connects a field to logic or a data source|
|Kitchen systems|Data sources|DynamoDB, Lambda, HTTP, OpenSearch, and supported services|
|Private status channel|Subscription|Authorized real-time WebSocket update|
|Reconnect and catch up|Sync/query|Recover current or changed state|

---

## A Note From the Author

The story implies that every status change reaches the phone. A disconnected client can miss live messages; reconnection should recover current state.

Subscription filters are not a substitute for authorization. Schema complexity, resolver fan-out, depth, payloads, connection quotas, caching, downstream consistency, and cost still require engineering.

- [AppSync real-time subscriptions](https://docs.aws.amazon.com/appsync/latest/devguide/aws-appsync-real-time-data.html)
- [AppSync conflict detection and sync](https://docs.aws.amazon.com/appsync/latest/devguide/conflict-detection-and-sync.html)

---

## The Last Bite

The customer should not repeatedly ask whether lunch changed.

The application should ask once, place the order, and listen for the relevant change.

> **Query what is true, mutate what should change, and subscribe to what changes next.**

---

**Next chapter:** *[AWS Service Integrations: The Retry Storm](09-the-retry-storm.md)*

The customer experience now spans ordering, payment, fulfillment, and notification. Then the external payment provider begins timing out.

