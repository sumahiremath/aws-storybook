---
description: "Choose storage from the required access interface and failure behavior before comparing performance or price."
tags:
  - "aws"
  - "storage"
  - "ebs"
  - "efs"
  - "s3"
  - "comparison"
---

# Amazon EBS, Amazon EFS, and Amazon S3: Choose the Right Home

> Choose storage from the required access interface and failure behavior before comparing performance or price.

## The Business Goal

Three projects reached Mira at once.

The editing server needed a boot drive.

Twenty render workers needed one directory tree.

Ten thousand clients needed private downloads.

All three requests said:

> “We need somewhere to put files.”

That sentence was true and almost useless.

---

## The Story

Mira asked each project one question:

> “How do you expect to touch the data?”

The server expected blocks and a filesystem it controlled.

The render workers expected one mounted shared tree.

The clients expected named objects through web requests.

The answer selected the room before speed, capacity, and cost refined the choice.

---

## The Wrong Way

Do not begin with durability numbers or price tables when the application interface is unresolved.

The cheapest object storage is expensive if the application must be rewritten to pretend it is a filesystem. A shared filesystem is the wrong publishing layer for millions of independent web downloads. A block volume is not a public object namespace.

---

## Meet the AWS Service

> **Core idea:** EBS serves block-device workloads, EFS serves shared NFS workloads, and S3 serves object-API workloads.

The services can appear in one architecture because different parts of an application have different access patterns.

---

## How It Works

### Project One: The Editing Server

#### Choose EBS

The server needs a boot volume and low-latency block access. It manages a filesystem and expects disk-like behavior.

EBS is the natural fit. Snapshots provide recovery points. Volume type and performance settings match the I/O profile.

### Project Two: The Render Farm

#### Choose EFS

Many Linux workers need the same paths and changing files at once.

EFS provides a shared NFS filesystem. Network, mount targets, security groups, POSIX permissions, and throughput matter.

### Project Three: The Client Gallery

#### Choose S3

The application needs durable objects addressed by key, direct uploads, temporary downloads, lifecycle rules, and object events.

S3 provides the object API. Presigned URLs grant temporary direct access.

### The Fast Delivery Desk

#### CloudFront Is Not the Storage Home

CloudFront can cache content near viewers and protect access to an S3 origin with the appropriate origin-access design.

S3 remains the origin storage. CloudFront is a content-delivery layer with cache behavior, expiration, and invalidation considerations.

### The Specialized Room

#### Know When the Three Are Not Enough

Windows SMB shares or specialized filesystem requirements can point to an Amazon FSx offering. Hybrid access may involve Storage Gateway. Large offline transfer can involve device-based migration services.

Those are specific requirements, not defaults to add to every architecture.

---

## Architectural Mapping

|Question|EBS|EFS|S3|
|---|---|---|---|
|Interface|Block device|NFS filesystem|Object API|
|Typical consumers|EC2 workload|Multiple compatible compute clients|Applications and services|
|Namespace|Filesystem chosen by client OS|Shared directories and files|Bucket and object keys|
|Sharing model|Usually attached to compatible EC2 placement|Concurrent network mounts|Independent API requests|
|Capacity model|Provisioned volume capacity|Elastic filesystem capacity|Object storage at service scale|
|Protection tools|Snapshots, encryption|Backup/lifecycle/encryption options|Versioning, Object Lock, replication, Lifecycle|
|Best first clue|“I need a disk”|“We need the same mounted files”|“We need to PUT and GET objects”|

---

## When to Use It

Use the following decision order:

1. Determine block, file, or object interface.
2. Determine who needs concurrent access.
3. Determine persistence and failure boundaries.
4. Determine latency, IOPS, throughput, and scale.
5. Determine permissions, encryption, recovery, and lifecycle.
6. Compare the complete cost model.

## When Not to Use It

Do not force a service merely because the data is colloquially called a "file." Ask what operations the application performs and what semantics it expects.

---

## Painkiller

> **Problem:** Different workloads all describe their requirement as file storage.  
> **Pain:** Choosing from the noun alone hides incompatible access semantics.  
> **AWS solution:** Select EBS, EFS, or S3 from the interface and sharing model, then refine for protection, performance, and cost.

---

## Knife Cut

> **EBS attaches. EFS mounts. S3 answers API requests.**

---

## The Masthead

### What Actually Just Happened

|In the story|In AWS|What it actually means|
|---|---|---|
|Editing server drive|EBS|Block storage for an EC2 workload|
|Render-farm directory|EFS|Shared managed NFS filesystem|
|Client gallery|S3|Object storage accessed by API|
|Fast delivery desk|CloudFront|Caching and delivery in front of an origin|
|Specialized room|FSx or hybrid service|Requirement-specific storage outside the core three|

---

## A Note From the Author

The table is a first decision aid, not a full architecture review. Availability, Regional design, quotas, client support, network paths, consistency needs, backup, compliance, and pricing can change the final answer.

Some workloads deliberately combine all three services. Selection is made per data path, not once per company.

- [AWS storage services overview](https://docs.aws.amazon.com/whitepapers/latest/aws-overview/storage-services.html)

---

## The Last Bite

The photograph did not ask for "storage."

It asked for blocks, a shared tree, or an object address.

> **Name the interface, and the right home becomes much easier to see.**

---

**Next chapter:** *[AWS Storage: What Remains](09-what-remains.md)*

The studio has found a home for every stage of the photograph. One final question remains: when stored data becomes application truth, how should that truth be organized?
