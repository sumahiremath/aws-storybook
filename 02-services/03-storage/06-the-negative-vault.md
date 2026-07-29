---
description: "Preserve recoverable history with Versioning, enforce retention with Object Lock, and copy objects deliberately with replication."
tags:
  - "aws"
  - "storage"
  - "s3"
---

# Amazon S3: The Negative Vault

> Preserve recoverable history with Versioning, enforce retention with Object Lock, and copy objects deliberately with replication.

## The Business Goal

An editor uploaded a corrected portrait to the same key.

The correction was wrong.

Then an assistant deleted the object while cleaning the gallery.

Mira had built durable storage, but durability had faithfully preserved the most recent mistake.

The system had survived hardware failure.

It had not survived an authorized human.

---

## The Story

Mira opened a negative vault.

Each time a photograph changed, the vault kept the older negative behind the newest one. When someone asked to remove the photograph, the attendant placed a black card at the front rather than burning every negative.

For legally protected collections, Mira sealed a particular negative in a sleeve that could not be opened until its retention period ended.

For regional resilience and ownership requirements, an authorized courier copied new negatives to another vault.

History, retention, and copying were three different controls.

---

## The Wrong Way

S3 durability is not the same as protection from overwrite or deletion.

Versioning is not a free, automatic backup policy. Old versions consume storage and can be permanently deleted by a principal with the required permission.

Replication is not retroactive by default for every historical object and is not a substitute for understanding delete behavior, KMS permissions, replication status, or destination configuration.

---

## Meet the AWS Service

> **Core idea:** S3 Versioning preserves multiple versions, Object Lock enforces WORM-style retention on versions, and Replication copies eligible objects under configured rules.

These features can work together, but they solve separate risks.

AWS performs the configured storage operations. You choose retention, lifecycle, permissions, destination, encryption behavior, and recovery procedures.

---

## How It Works

### The Stack of Negatives

#### S3 Versioning

With Versioning enabled, a new `PUT` to an existing key creates a new version and retains the previous version as noncurrent.

Applications can request a specific version ID. Copying a previous version to the same key can make its data current while preserving history.

Versioning can be enabled and later suspended, but a bucket does not return to a never-versioned state.

### The Black Card

#### Delete Markers

A simple `DeleteObject` request in a versioning-enabled bucket usually creates a delete marker. A normal `GET` then behaves as though the object is absent, while older versions remain.

Deleting that delete marker can reveal the previous version again. Permanently deleting a specific version requires its version ID and sufficient permission.

### The Sealed Sleeve

#### S3 Object Lock

Object Lock applies WORM-style protection to object versions in a versioned bucket.

- **Governance mode** can be bypassed by specially authorized principals.
- **Compliance mode** prevents protected deletion or shortening retention even by the account root user during the retention period.
- **Legal hold** remains until explicitly removed by an authorized action.

Object Lock protects a version. It does not prevent a new version or a delete marker from becoming current.

### The Second Vault

#### S3 Replication

Same-Region Replication and Cross-Region Replication copy eligible objects according to rules. Versioning is required on source and destination general purpose buckets.

S3 assumes an IAM role with permission to read from the source and write to the destination. SSE-KMS objects require deliberate KMS configuration and permission.

Live replication applies to new eligible objects after the rule is active. S3 Batch Replication can address existing or failed objects.

Replication status should be monitored; a configured courier can still encounter permission or encryption failures.

---

## Architectural Mapping

```text
PUT same key
    |
    v
Current version ----> noncurrent versions
    |
    +--> optional Object Lock retention
    |
    `--> replication rule --> destination bucket versions
```

Versioning creates history. Lifecycle controls its long-term cost. Object Lock constrains deletion. Replication creates another copy. None of them independently defines a complete recovery plan.

---

## When to Use It

Use Versioning when accidental overwrite and delete recovery matter. Add Object Lock when records require enforced immutability. Use replication for compliance, ownership separation, latency, resilience, or another intentional copy.

## When Not to Use It

Do not enable indefinite history without lifecycle and cost planning. Do not use replication alone as protection from a bad write if the same bad write is eligible to replicate.

---

## Painkiller

> **Problem:** Authorized changes can overwrite or hide valuable objects.  
> **Pain:** Infrastructure durability cannot distinguish a correction from a mistake.  
> **AWS solution:** Preserve versions, enforce retention where required, and replicate eligible objects under explicit rules.

---

## Knife Cut

> **Versioning keeps history. Object Lock constrains deletion. Replication creates another copy.**

---

## The Masthead

### What Actually Just Happened

|In the story|In AWS|What it actually means|
|---|---|---|
|Stack of negatives|Object versions|Current and noncurrent data under one key|
|Black card|Delete marker|Current marker that hides older versions from ordinary reads|
|Sealed sleeve|Object Lock|Retention or legal hold on a version|
|Courier|S3 replication role|Authorized service role copying eligible objects|
|Second vault|Destination bucket|Same-Region or cross-Region replica location|

---

## A Note From the Author

The vault metaphor suggests that every earlier negative is safe forever. Permissions and lifecycle rules can permanently delete noncurrent versions when Object Lock does not prevent it.

Replication behavior varies for delete markers, existing objects, ownership, encryption, and rule configuration. Test the intended recovery event rather than treating a green configuration screen as proof.

- [S3 Versioning](https://docs.aws.amazon.com/AmazonS3/latest/userguide/Versioning.html)
- [S3 Object Lock](https://docs.aws.amazon.com/AmazonS3/latest/userguide/object-lock.html)
- [S3 Replication](https://docs.aws.amazon.com/AmazonS3/latest/userguide/replication.html)

---

## The Last Bite

Durability protects against failing storage.

History protects against changing minds and changing hands.

> **Keep the version you may need before the mistake teaches you its value.**

---

**Next chapter:** *[Amazon S3: The Archive Elevator](07-the-archive-elevator.md)*

The vault now keeps history. Without lifecycle rules, it will also keep every storage bill.
