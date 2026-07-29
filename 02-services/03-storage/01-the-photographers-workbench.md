---
description: "Separate disposable scratch space from the durable block storage that active compute must keep."
tags:
  - "aws"
  - "storage"
  - "ebs"
---

# Instance Store and Amazon EBS: The Photographer’s Workbench

> Separate disposable scratch space from the durable block storage that active compute must keep.

## The Business Goal

Mira's fastest editor rendered previews onto the workstation's local scratch disk.

Then the workstation was replaced.

The previews vanished. That was acceptable; the studio could render them again.

But the editor had also saved six hours of irreplaceable retouching there.

Fast and safe had been mistaken for the same thing.

---

## The Story

Mira placed two trays beside every editing station.

The first was labeled **SCRATCH**. Anything inside could be swept away when the station left the room.

The second was a detachable working case. The workstation could be replaced, but the case could be attached to another compatible station.

Before a risky edit, the assistant photographed the case's contents as a recovery contact sheet.

Scratch tray. Working case. Recovery point.

Three different promises.

---

## The Wrong Way

A fast local disk is tempting for important data because it is physically close to compute.

But EC2 instance store is tied to its host. Stopping, terminating, or losing the instance can make that data unavailable or destroy it depending on the event and instance behavior.

If data cannot be recreated, ephemerality is not a performance optimization. It is an unpriced failure.

---

## Meet the AWS Service

> **Core idea:** Amazon EBS provides persistent block storage volumes for EC2 workloads; instance store provides temporary block storage tied to the host.

An EBS volume behaves like a raw block device. The operating system creates a filesystem or otherwise uses those blocks.

AWS manages the storage service. You manage attachment, filesystem, capacity, performance settings, encryption choices, backups, and application consistency.

---

## How It Works

### The Scratch Tray

#### EC2 Instance Store

Instance store is useful for caches, buffers, scratch data, and replicas that can be rebuilt. It is not a durable source of truth.

### The Working Case

#### EBS Volumes

EBS volumes live independently from the running process and can persist beyond an instance when deletion settings permit.

Most volumes attach within one Availability Zone. The instance and volume must therefore be placed compatibly. Multi-Attach exists for supported volume and instance combinations, but it does not turn ordinary block storage into a carefree shared filesystem; the application and filesystem must coordinate concurrent writes.

### Choosing the Case

#### Performance Fit

General Purpose SSD volumes suit many transactional workloads. Provisioned IOPS SSD volumes fit latency-sensitive workloads that require more predictable IOPS. Throughput Optimized HDD and Cold HDD fit large sequential workloads better than small random I/O.

Capacity, IOPS, throughput, latency, and workload shape are separate questions.

### The Recovery Contact Sheet

#### EBS Snapshots

Snapshots are incremental point-in-time backups stored through an AWS-managed process. A snapshot can create a new volume, including in another Availability Zone.

For application-consistent recovery, the application may need to flush writes, pause I/O, or use a coordinated backup mechanism. A crash-consistent block image is not automatically a transactionally perfect application backup.

---

## Architectural Mapping

```text
EC2 instance
  |-- instance store -> disposable scratch data
  |
  `-- EBS volume ----> durable working blocks
                           |
                           `--> EBS snapshot -> recovery point
```

Encryption with an AWS KMS key can protect an EBS volume and its snapshots. IAM controls the AWS API actions; operating-system permissions still control access inside the mounted filesystem.

---

## When to Use It

Use instance store when:

- Data is temporary or replicated elsewhere
- Very fast host-local scratch space fits the workload
- Loss with the host is acceptable

Use EBS when:

- An EC2 workload needs a boot or data volume
- The application expects block-device semantics
- Data must persist independently of a process or instance lifecycle
- Snapshot-based recovery is useful

## When Not to Use It

Choose EFS when many Linux clients need a managed shared filesystem. Choose S3 when applications need durable objects through APIs rather than mounted block storage.

---

## Painkiller

> **Problem:** Active compute needs both disposable speed and persistent working state.  
> **Pain:** Treating host-local storage as durable can erase irreplaceable work.  
> **AWS solution:** Keep reproducible scratch data on instance store and important block data on EBS with deliberate recovery protection.

---

## Knife Cut

> **A volume keeps working state. A snapshot keeps a recovery point. Neither is a shared object API.**

---

## The Masthead

### What Actually Just Happened

|In the story|In AWS|What it actually means|
|---|---|---|
|Scratch tray|Instance Store|Host-local temporary block storage|
|Working case|EBS volume|Persistent block storage attached to EC2|
|Case dimensions|Volume type, size, IOPS, throughput|Configured performance and capacity|
|Recovery contact sheet|EBS snapshot|Point-in-time recovery source|

---

## A Note From the Author

An EBS volume is not literally a portable suitcase. Availability Zone placement, attachment support, instance compatibility, operating-system behavior, and filesystem integrity constrain movement.

Snapshots are durable recovery artifacts, but recovery quality still depends on application consistency and whether the recovery procedure has been tested.

- [Amazon EBS User Guide](https://docs.aws.amazon.com/ebs/latest/userguide/what-is-ebs.html)

---

## The Last Bite

The fastest surface is allowed to forget.

The working drive is not.

> **Put disposable speed and durable state in different places.**

---

**Next chapter:** *[Amazon EFS: The Shared Darkroom](02-the-shared-darkroom.md)*

The working file now survives its workstation. The next problem begins when several editors need it at once.

