---
description: "Storage architecture begins with what the data must do, not with the name of a storage service."
tags:
  - "aws"
  - "storage"
  - "orientation"
---

# AWS Storage: The Memory Chooses Its Home

> Storage architecture begins with what the data must do, not with the name of a storage service.

## The Business Goal

Mira's photography studio had one enormous drive labeled `EVERYTHING`.

It held temporary previews, unfinished edits, client galleries, and wedding photographs that could never be recreated.

The drive was fast. It was also one failure away from becoming a very expensive silence.

The problem was not that the studio lacked storage.

The problem was that it treated every memory as though it had the same job.

---

## The Story

A new photograph entered the studio as a huge RAW file.

During editing, it needed a fast workbench. When three specialists began retouching it, it needed a shared room. When the client approved it, it needed a gallery address. Years later, it needed an inexpensive archive.

```text
Editing -> Sharing -> Delivering -> Preserving
```

The photograph moved because its purpose changed.

> **The data chooses its home.**

---

## The Wrong Way

Mira could keep everything on the fastest disk.

That would make cold archives unnecessarily expensive and tie published content to a workstation.

She could put everything in an object store.

That would give her scale, but an object API would not behave like the mounted working filesystem her editing tools expected.

There is no universally best storage service. There is only behavior that fits the data.

---

## Meet the AWS Service

AWS exposes three foundational storage shapes:

> **Core idea:** EBS is block storage for a machine, EFS is a shared filesystem, and S3 is an object store reached through APIs.

- **Amazon EBS** presents durable block volumes, most commonly to EC2 instances.
- **Amazon EFS** presents a managed NFS filesystem that multiple compatible clients can mount.
- **Amazon S3** stores objects in buckets and exposes them through APIs.

AWS manages the underlying storage infrastructure. You still choose the model, permissions, protection, performance, lifecycle, and cost behavior.

---

## How It Works

### The Editing Workbench

#### Instance Store and EBS

Disposable render files can use EC2 instance store when the instance type provides it. Important working files belong on persistent storage such as EBS.

An EBS snapshot records a point-in-time recovery copy of a volume. It is protection, not a second live editing disk.

### The Shared Darkroom

#### Amazon EFS

When several Linux-based workers need the same hierarchy of files and directories, EFS provides shared NFS semantics.

The filesystem is reached across the network, so mount targets, security groups, permissions, performance, and throughput still matter.

### The Client Gallery

#### Amazon S3

When a photograph becomes a published asset, it stops needing filesystem semantics. It becomes an object with a key, data, and metadata inside a bucket.

Applications use operations such as `PutObject`, `GetObject`, `HeadObject`, `ListObjectsV2`, and `DeleteObject`.

### The Underground Vault

#### S3 storage classes and Lifecycle

As access fades, S3 Lifecycle rules can transition eligible objects into lower-cost storage classes or expire them.

Some archival classes require a restore before an object can be read. Lower storage cost may bring retrieval delay, retrieval charges, or minimum-duration charges.

---

## Architectural Mapping

```text
Fast private work       Shared project       Published asset       Cold memory
Instance Store/EBS ---> EFS -------------> S3 Standard --------> S3 archive class
```

This is a journey, not a mandatory pipeline. Many applications use only one of these services. The point is to select according to access shape and lifecycle.

---

## When to Use It

Ask these questions before choosing:

- Does the application need a block device, filesystem, or object API?
- Does one machine need the data, or do many clients need it?
- Is the data changing frequently or mostly being retrieved?
- What latency, throughput, durability, and availability are required?
- Can the data be recreated?
- How long must it remain, and how quickly must it return?

## When Not to Use It

Do not choose storage from a one-word requirement such as "fast," "durable," or "cheap." Those words omit access protocol, sharing, failure boundaries, recovery, and lifecycle.

---

## Painkiller

> **Problem:** Application data has different shapes, lifetimes, and consequences of loss.  
> **Pain:** One storage system produces mismatched performance, access, cost, or recovery behavior.  
> **AWS solution:** Select block, file, or object storage according to the data's actual job.

---

## Knife Cut

> **Block belongs to a machine. File belongs to a shared directory tree. Object belongs to an API namespace.**

---

## The Masthead

### What Actually Just Happened

|In the story|In AWS|What it actually means|
|---|---|---|
|Scratch pad|Instance Store|Fast host-local storage that is not a durable home|
|Working drive|Amazon EBS|Persistent block storage for active compute|
|Shared darkroom|Amazon EFS|Managed shared NFS storage|
|Client gallery|Amazon S3|Objects stored in buckets and accessed by API|
|Archive elevator|S3 Lifecycle|Rules that transition or expire eligible objects|

---

## A Note From the Author

Files do not move themselves merely because their business value changes. Architects configure copy operations, application workflows, backups, replication, and lifecycle rules.

Durability does not replace authorization or recovery planning. A durable object can still be exposed, overwritten, or deliberately deleted when controls allow it.

- [AWS storage services overview](https://docs.aws.amazon.com/whitepapers/latest/aws-overview/storage-services.html)
- [Amazon S3 documentation](https://docs.aws.amazon.com/AmazonS3/latest/userguide/Welcome.html)

---

## The Last Bite

Compute decides where work runs.

Storage decides what remains after the work is done.

> **The data chooses its home—and its job tells you which home fits.**

---

**Next chapter:** *[Instance Store and Amazon EBS: The Photographer’s Workbench](01-the-photographers-workbench.md)*

Before a photograph can become a gallery object, it must survive the machine doing the editing.
