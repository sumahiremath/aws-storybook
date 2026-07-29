---
description: "RDS and Aurora give applications managed relational databases, but developers still own schemas, transactions, connections, credentials, and failure-aware code."
tags:
  - "aws"
  - "databases"
  - "rds"
  - "aurora"
---

# Amazon RDS and Amazon Aurora: The Accounting Office

> RDS and Aurora give applications managed relational databases, but developers still own schemas, transactions, connections, credentials, and failure-aware code.

## The Business Goal

The DynamoDB warehouse was complete.

Then Shreya returned with a refund.

The refund belonged to a payment.

The payment belonged to an order.

The order belonged to a customer and a merchant.

The merchant expected a settlement.

> “Give me every fact connected to this money,” she said.

The dispatcher asked for the partition key.

Shreya placed six ledgers on the counter.

> “I do not have one question. I am following a relationship.”

---

## The Story

### The accounting office

The architect opened a room built around connected records.

Inside, each fact had a defined place:

```text
Customer
   |
   v
Order ----> OrderItem ----> Product
   |
   v
Payment ----> Refund
   |
   v
Merchant ----> Settlement
```

The clerks could join the ledgers.

They could enforce constraints.

They could update several related records inside one transaction.

Shreya asked:

> “Record the refund, adjust the payment, and update the merchant balance. If one fails, do not leave the books halfway changed.”

The chief accountant opened a transaction.

```text
BEGIN
  create refund
  update payment
  update merchant balance
COMMIT
```

All three changes committed.

Or the office rolled them back.

This room did not require every future question to be designed as a retrieval lane in advance.

Its central promise was different:

> **Preserve relationships, then let SQL follow them.**

---

## Meet Amazon RDS

Amazon Relational Database Service manages supported relational database engines.

Depending on the chosen engine and configuration, RDS can manage work such as:

- infrastructure provisioning
- operating-system and database software maintenance
- backups
- monitoring integrations
- storage management
- high-availability mechanisms
- replacement of failed infrastructure

The application team still owns:

- schema and relationship design
- SQL and indexes
- transaction boundaries
- connection behavior
- credentials and authorization
- network access
- engine configuration within the managed model
- query performance
- application retries and failover recovery
- cost

RDS manages the database platform.

It does not manage the meaning of the company’s books.

---

## Transactions Are Business Sentences

Shreya’s refund contained three statements that formed one business sentence.

If the application committed only the first clause, the books became inconsistent.

Relational transactions provide ACID behavior through the database engine:

- **Atomicity:** the transaction commits as a unit or rolls back
- **Consistency:** database rules remain satisfied
- **Isolation:** concurrent work follows the selected isolation behavior
- **Durability:** committed changes survive according to the database guarantee

The developer chooses:

- where a transaction begins and ends
- which statements belong together
- what isolation level the workflow needs
- how long locks or versions may be held
- what happens after deadlock, timeout, or serialization failure

Long transactions can create contention.

Retrying a transaction without idempotent business logic can repeat external side effects.

The database protects its transaction.

The application protects the complete workflow.

---

## The Doorway Fills With Connections

Shreya’s application opened a database connection.

Then a thousand Lambda invocations arrived.

Each opened another.

The SQL was fast.

The doorway was not.

Relational databases maintain state and resources for client connections. Applications should reuse connections when the runtime permits, close them correctly, and control pool sizes.

Connection storms can consume database memory and CPU before useful queries execute.

```text
many application workers
          |
          v
connection pool or proxy
          |
          v
smaller controlled set of database connections
```

Scaling application workers does not automatically scale database connections safely.

---

## RDS Proxy: The Door Coordinator

RDS Proxy pools and reuses connections for supported RDS and Aurora engines.

Applications connect to the proxy endpoint instead of connecting directly to the database endpoint.

```text
Lambda and application workers
             |
             v
          RDS Proxy
             |
             v
managed connection pool
             |
             v
        RDS or Aurora
```

RDS Proxy can:

- reduce connection-establishment overhead
- protect the database from connection surges
- improve application resilience during supported failovers
- use credentials stored in AWS Secrets Manager
- support IAM database authentication in supported configurations

It does not:

- repair an inefficient query
- add missing indexes
- remove transaction contention
- make every session safe to multiplex
- turn a relational workload into DynamoDB

Certain session state can pin a client connection to one database connection and reduce multiplexing efficiency.

The door coordinator helps when the problem is the doorway.

---

## The Standby Accountant

The chief accountant’s desk failed.

The owner wanted another desk ready in a different Availability Zone.

For a traditional RDS Multi-AZ DB instance deployment, RDS maintains a synchronous standby and automatically fails over when required.

The standby exists for high availability.

It does not serve ordinary application reads.

```text
application
     |
     v
primary DB instance
     |
     | synchronous replication
     v
standby in another AZ
```

During failover, RDS changes where the DB endpoint resolves.

Applications must:

- connect through the endpoint rather than a fixed IP address
- tolerate broken connections
- reconnect after failover
- retry only work that is safe to retry
- respect DNS behavior and connection-pool refresh

The standby reduces infrastructure recovery work.

It does not make a live transaction survive a disconnected session.

RDS also offers Multi-AZ DB cluster deployments with a different architecture that includes readable instances. The precise behavior depends on the selected RDS deployment model.

---

## The Extra Reading Desks

Shreya’s reports competed with checkout transactions.

The office added read replicas.

RDS read replicas use engine replication to copy changes asynchronously from the source.

Applications connect to each read replica through its own endpoint.

```text
write traffic
     |
     v
primary database
     |
     | asynchronous replication
     +------------------+
     |                  |
     v                  v
read replica A     read replica B
```

Read replicas can:

- offload read-heavy queries
- support reporting
- provide geographically closer reads in some designs
- be promoted to standalone databases for supported recovery or migration scenarios

They can lag.

A read immediately after a write might not appear on a replica yet.

> **Multi-AZ protects availability. Read replicas scale or isolate reads.**

Those roles can coexist.

They are not interchangeable.

---

## Aurora Rebuilds the Office

Amazon Aurora is a relational database compatible with MySQL or PostgreSQL.

It retains SQL, transactions, tables, indexes, and connections.

Its managed architecture separates cluster storage from DB compute instances and provides purpose-built replication and recovery behavior.

An Aurora cluster normally has:

- one writer DB instance
- zero or more Aurora Replicas
- shared distributed cluster storage

Applications reach the cluster through endpoints.

### Writer endpoint

The cluster, or writer, endpoint connects to the current writer.

Use it for read/write traffic.

### Reader endpoint

The reader endpoint balances new connections across available Aurora Replicas.

Use it for read-oriented traffic that can follow replica consistency behavior.

### Instance endpoint

An instance endpoint connects to one specific DB instance.

It is useful for diagnosis or specialized routing, but normal application code should not accidentally bind itself to the instance that happens to be writer today.

### Custom endpoint

A custom endpoint routes to a chosen subset of instances.

It can separate differently sized or differently purposed reader fleets.

```text
write connection ----------> writer endpoint ----------> writer

read connections ----------> reader endpoint ----------> replicas

special workload ----------> custom endpoint ----------> selected instances
```

During failover, Aurora can promote a replica and redirect the writer endpoint.

Applications still need reconnect and retry behavior.

---

## Provisioned and Serverless Compute

Aurora can use provisioned DB instances.

Supported Aurora configurations can also use Aurora Serverless v2, where database compute scales within configured capacity boundaries.

Serverless compute does not mean:

- there are no database connections
- every query scales independently
- the schema no longer matters
- transactions stop contending
- the application pays nothing while active

The relational model remains.

The compute-management model changes.

---

## The Credentials Cannot Stay Under the Mat

The application needs database credentials or a supported IAM authentication flow.

Hard-coding a username and password in source code or a deployment template creates unnecessary exposure and rotation pain.

AWS Secrets Manager can store database credentials and support rotation workflows for supported database configurations.

Applications retrieve the secret at runtime and should cache it safely rather than calling the secrets service for every SQL statement.

IAM database authentication can replace password authentication for supported engines and scenarios by using short-lived authentication tokens.

The application still needs:

- IAM permission to connect
- a database user
- network reachability
- TLS
- token refresh behavior
- an appropriate connection strategy

IAM authentication does not replace database authorization.

It changes how the client proves its identity at connection time.

---

## The Office Has Walls

RDS and Aurora databases normally live inside a VPC.

Security groups control allowed network paths.

Database credentials or IAM authentication control login.

Database grants control what the connected user may do.

Encryption at rest can use AWS KMS.

TLS protects supported connections in transit.

These are different controls:

```text
network can reach database?
          |
          v
client can authenticate?
          |
          v
database user is authorized?
          |
          v
query may execute
```

A secret does not open a blocked security group.

An open security group does not grant a database role.

---

## The Books Need Recovery

RDS automated backups support point-in-time recovery within the configured retention period.

Manual snapshots remain until deleted.

A restore creates a new DB instance or cluster rather than rewinding the active database in place.

The team must plan:

- recovery point and recovery time objectives
- restoration testing
- endpoint changes
- credential and security configuration
- application cutover
- reconciliation after the selected recovery point

The backup stores the books.

The runbook reopens the office.

---

## The Wrong Way

The wrong way is to say:

> “Multi-AZ makes reads scale.”

That is not true for the standby in a traditional Multi-AZ DB instance deployment.

Another wrong way is:

> “A read replica is always current.”

Replication is asynchronous.

And another:

> “RDS Proxy fixes database performance.”

It fixes a class of connection-management problems.

The correct tool follows the pressure:

- transactions and relationships -> relational database
- infrastructure availability -> Multi-AZ design
- read scaling -> read replicas or Aurora readers
- connection storms -> pooling or RDS Proxy
- secret rotation -> Secrets Manager or supported IAM authentication

---

## Architectural Mapping

```text
application workers
        |
        v
RDS Proxy or application pool
        |
        v
writer endpoint
        |
        +--> transactional primary / Aurora writer
        |
        +--> standby or failover target
        |
        +--> asynchronous read replicas
```

The application needs different endpoints and failure behavior for writes, reads, and failover.

One connection string should not be asked to express every role accidentally.

---

## When to Use RDS or Aurora

Start with a managed relational database when:

- SQL and flexible joins are central
- constraints protect the data model
- transactions span connected records
- the team depends on a supported relational engine ecosystem
- access patterns are not limited to predefined key lookups

Consider DynamoDB or another store when:

- the hottest workload is known-key operational access at managed horizontal scale
- joins are not required
- application-defined keys and denormalization fit better

---

## Painkiller

> **Problem:** Connected business records require SQL transactions while application scale creates availability, read, connection, and credential pressure.  
> **Pain:** Treating every replica, standby, endpoint, and proxy as interchangeable leads to stale reads, exhausted connections, or fragile failover.  
> **AWS solution:** Use RDS or Aurora for relational work, then match Multi-AZ, read replicas, Aurora endpoints, RDS Proxy, and secret management to the specific pressure.

---

## Knife Cuts

> **Multi-AZ protects availability. Read replicas scale or isolate reads.**

> **RDS Proxy manages connections. It does not optimize the SQL traveling through them.**

> **Aurora changes the managed architecture. It remains a relational database.**

---

## The Accounting Office

### What Actually Just Happened

| In the story | In AWS | What it actually means |
|---|---|---|
| Connected ledgers | Relational schema | Tables, keys, constraints, and joins |
| One business sentence | Transaction | Related statements commit or roll back |
| Doorway coordinator | RDS Proxy | Managed connection pooling and reuse |
| Standby accountant | Multi-AZ standby | High-availability failover target |
| Extra reading desks | Read replicas | Asynchronous read scaling |
| Writer desk | Aurora writer endpoint | Current read/write DB instance |
| Reading room | Aurora reader endpoint | Connections distributed across replicas |
| Key under the mat | Hard-coded credential | Secret that cannot be rotated safely |

The relational office did not replace the warehouse.

It accepted the work whose meaning lived in relationships.

---

## A Note From the Author

RDS behavior varies by database engine and deployment model. A traditional Multi-AZ DB instance standby is not a read target, while an RDS Multi-AZ DB cluster has readable instances. Always identify the exact configuration in an architecture question.

Aurora endpoints balance connections, not individual SQL statements. Existing connections remain attached to the instance they reached until the client reconnects.

Read replicas can lag. Applications must decide which reads tolerate that lag.

RDS Proxy supports specific engines, versions, authentication modes, and session behavior. Transactions and session state can affect connection multiplexing.

Technical references:

- [Working with RDS read replicas](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/USER_ReadRepl.html)
- [RDS Multi-AZ failover](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/Concepts.MultiAZ.Failover.html)
- [Aurora endpoints](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/Aurora.Overview.Endpoints.html)
- [Amazon RDS Proxy for Aurora](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/rds-proxy.html)

---

## The Last Bite

Relationships need an office.

Availability needs a standby.

Read traffic needs another desk.

Connection storms need a coordinator.

Do not hire them all for the same job.

---

**Next chapter:** *[Amazon ElastiCache: The Memory Desk](08-elasticache-memory-desk.md)*

The accounting office can answer Leo’s product request.

Then ten thousand customers ask the identical question before the underlying fact changes once.
