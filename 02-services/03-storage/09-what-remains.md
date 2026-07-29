---
description: "Storage architecture is the deliberate journey from temporary work to durable, protected, and affordable memory."
tags:
  - "aws"
  - "storage"
  - "epilogue"
---

# AWS Storage: What Remains

> Storage architecture is the deliberate journey from temporary work to durable, protected, and affordable memory.

## The Business Goal

On the morning after the Bellweather wedding, Mira looked at the studio.

Temporary previews filled the scratch trays.

The active album rested on a working drive.

Retouchers shared one project tree.

Approved portraits waited in the gallery.

Protected versions slept in the vault.

Nothing was stored in one universal place.

Nothing needed to be.

---

## The Story

The final portrait had changed homes four times.

At the workbench, speed mattered.

In the darkroom, sharing mattered.

In the gallery, durable object access mattered.

In the vault, history, retention, retrieval time, and cost mattered.

The photograph never changed its pixels merely because its home changed.

Its job changed.

And the architecture followed.

---

## Meet the AWS Service

> **Core idea:** AWS storage services expose different access models because data plays different roles across its lifecycle.

The studio's complete lesson fits in three verbs:

- EBS **attaches**
- EFS **mounts**
- S3 **answers object requests**

Protection and lifecycle then refine the choice:

- Snapshots preserve block recovery points
- Versioning preserves object history
- Object Lock enforces retention
- Replication creates deliberate copies
- Lifecycle changes tier or expires data

---

## How It Works

### Ask the Data

Before selecting a service, ask:

- What interface does the application expect?
- Who needs concurrent access?
- What can be recreated?
- What must survive deletion or overwrite?
- What must be immutable?
- How quickly must recovery occur?
- How will access change with age?
- Which principal or client should be allowed to perform which action?

### Follow the Failure

Architecture becomes clear when the failure is concrete:

- Host loss exposes instance-store ephemerality
- Disk-state loss exposes missing EBS recovery
- Divergent copies expose the need for a shared filesystem
- Public exposure exposes excessive S3 permissions
- Duplicate processing exposes non-idempotent event handling
- Accidental overwrite exposes missing version history
- Slow restore exposes the wrong archive class

### Keep the Boundaries

Storage does not answer every data question.

S3 can hold an invoice PDF, but it does not automatically provide transactional queries across invoice records. EBS can hold database files, but the database engine provides the data model and transactions. EFS can share files, but it does not decide which record is the authoritative truth.

Storage answers where bytes live and how they are accessed.

The next section asks how application truth is organized, queried, and changed.

---

## Architectural Mapping

```text
Disposable work -> Durable blocks -> Shared files -> API objects -> Protected archive
                                                              |
                                                              v
                                             Application truth needs a data model
```

---

## When to Use It

Return to this model whenever a design says only:

- "Put it on a disk"
- "Share the file"
- "Upload it to the cloud"
- "Keep it forever"

Each phrase conceals an interface, permission model, failure boundary, and lifecycle.

## When Not to Use It

Do not stretch the photo studio into a database analogy. A catalogue card can help explain an object's metadata, but it cannot faithfully explain keys, indexes, consistency, transactions, and access-pattern design.

The analogy has completed its job.

---

## Painkiller

> **Problem:** Data changes roles while architectures treat storage as one permanent choice.  
> **Pain:** The mismatch creates loss, exposure, duplication, poor retrieval, or unnecessary cost.  
> **AWS solution:** Move from block to file to object or archive models only when the data's access and lifecycle require it.

---

## Knife Cut

> **Storage preserves bytes. A database organizes changing truth for application questions.**

---

## The Masthead

### What Actually Just Happened

|Moment|Question|AWS answer|
|---|---|---|
|Editing|What must be fast, and what must persist?|Instance Store and EBS|
|Sharing|Who needs one filesystem?|EFS|
|Publishing|Who needs an object API?|S3|
|Protecting|What must survive change or deletion?|Versioning, Object Lock, replication|
|Preserving|How will access and cost change?|Storage classes and Lifecycle|

---

## A Note From the Author

No photograph is required to traverse every stage, and copying data between services has time, cost, permission, and consistency implications.

The analogy also hides the operational work: capacity and performance tests, IAM review, encryption configuration, observability, recovery drills, lifecycle validation, and cost monitoring remain engineering responsibilities.

- [AWS Storage documentation](https://docs.aws.amazon.com/whitepapers/latest/aws-overview/storage-services.html)

---

## The Last Bite

The studio did not preserve every memory in the same room.

It preserved each memory according to the work it still had to do.

> **The data chooses its home. The architect makes that choice survivable.**

---

**Next chapter:** *[Amazon Athena: The Photo Archivist’s Reading Room](10-amazon-athena-the-photo-archivists-reading-room.md)*

The studio has preserved years of object files. Before moving on to transactional truth, it needs a way to ask questions of that archive without turning the archive into an application database.
