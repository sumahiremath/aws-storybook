---
description: "Give multiple compatible compute clients one managed filesystem instead of copying the same project everywhere."
tags:
  - "aws"
  - "storage"
  - "efs"
---

# Amazon EFS: The Shared Darkroom

> Give multiple compatible compute clients one managed filesystem instead of copying the same project everywhere.

## The Business Goal

Mira added a colorist, a retoucher, and an album designer.

Each copied the Bellweather project onto a private drive.

By lunch, the studio had four versions of the truth:

`final.psd`

`final-mira.psd`

`final-really.psd`

`final-use-this-one.psd`

The storage was durable. The collaboration was not.

---

## The Story

Mira built one shared darkroom with labeled drawers.

Every authorized editor entered through a nearby door and worked in the same directory tree. Nobody carried a private duplicate back to a desk.

When a new editing room opened in another wing, it needed its own entrance to the darkroom.

The shared room solved copying. It did not remove doors, permissions, traffic, or contention.

---

## The Wrong Way

Attaching one ordinary EBS volume to several independent writers is not the default solution for shared files.

Block devices expose blocks. A shared filesystem coordinates filenames, directories, metadata, and concurrent access semantics.

Copying files through S3 can exchange objects, but it does not make S3 behave like an NFS filesystem.

---

## Meet the AWS Service

> **Core idea:** Amazon EFS is a managed, elastic NFS filesystem that multiple supported clients can mount.

EFS is regional and stores data across Availability Zones for regional file systems. Clients connect through mount targets in the VPC.

AWS manages the filesystem infrastructure and capacity scaling. You manage network reachability, client permissions, file permissions, performance configuration, lifecycle, and cost.

---

## How It Works

### Doors Near the Editors

#### Mount Targets

A mount target gives clients in an Availability Zone a network endpoint for the filesystem. DNS normally guides a client to an appropriate mount target.

Security groups must allow NFS traffic. An IAM permission to call EFS APIs does not by itself open the network path or grant POSIX file access.

### Names on the Drawers

#### POSIX Permissions and Access Points

Linux ownership and mode bits still matter. EFS access points can provide an application-specific entry path and POSIX identity, which helps isolate applications sharing one filesystem.

### A Room That Expands

#### Elastic Capacity

EFS grows and shrinks as files are added and removed. There is no volume size to preallocate in the ordinary model.

Performance mode and throughput configuration affect behavior. Workloads with many small metadata operations feel different from large sequential transfers.

### One Building or the Region

#### Regional and One Zone File Systems

Regional EFS file systems store data redundantly across multiple Availability Zones and are the general-purpose choice. EFS One Zone stores data within one Availability Zone at a lower cost, but it is not resilient to loss of that Availability Zone.

The choice is an availability and durability boundary, not merely a price switch.

### Moving Old Projects Downstairs

#### EFS Lifecycle Management

Lifecycle policies can move eligible files from EFS Standard into EFS Infrequent Access and EFS Archive as access fades. The filesystem path remains, while access latency and cost behavior can change.

---

## Architectural Mapping

```text
EC2 / containers / supported compute
               |
          NFS + network
               |
         EFS mount target
               |
          One filesystem
```

The client needs a route, security-group permission, mount configuration, and suitable filesystem permissions. Encryption can protect data at rest and in transit when configured.

---

## When to Use It

Use EFS when:

- Multiple Linux-based clients need the same files
- Applications require NFS and hierarchical filesystem semantics
- Capacity should grow without provisioning a fixed volume size
- Shared content, home directories, or application assets fit network-file behavior

## When Not to Use It

Use EBS for a workload that needs a block device closely associated with EC2. Use S3 for web-scale objects accessed through APIs. Consider an Amazon FSx offering when Windows SMB or a specialized filesystem is the actual requirement.

---

## Painkiller

> **Problem:** Several workers need one changing directory tree.  
> **Pain:** Private copies create stale files and conflicting versions.  
> **AWS solution:** EFS provides a managed shared NFS filesystem reachable by multiple authorized clients.

---

## Knife Cut

> **EBS gives a machine blocks. EFS gives clients a shared filesystem. S3 gives applications objects.**

---

## The Masthead

### What Actually Just Happened

|In the story|In AWS|What it actually means|
|---|---|---|
|Shared darkroom|EFS filesystem|Regional managed NFS storage|
|Nearby door|Mount target|VPC endpoint used by NFS clients|
|Door policy|Security group|Network permission for NFS traffic|
|Drawer labels|POSIX paths and permissions|Filesystem namespace and client authorization|
|Dedicated cabinet|EFS access point|Application-specific root and identity|
|One-wing darkroom|EFS One Zone|Single-AZ filesystem with a different resilience boundary|

---

## A Note From the Author

A shared room sounds frictionless. A network filesystem still has latency, throughput, metadata, concurrency, and failure behavior. Applications must tolerate network interruptions and coordinate writes correctly.

EFS does not replace application-level locking or make every workload faster than local block storage.

- [Amazon EFS User Guide](https://docs.aws.amazon.com/efs/latest/ug/whatisefs.html)

---

## The Last Bite

The studio did not need more copies.

It needed one shared place with deliberate entrances.

> **When the directory tree belongs to many workers, give it a filesystem built to be shared.**

---

**Next chapter:** *[Amazon S3: The Gallery of Objects](03-the-gallery-of-objects.md)*

The album is approved. Clients do not need the darkroom—they need a gallery address.
