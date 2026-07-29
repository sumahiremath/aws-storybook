---
description: "Move large objects in recoverable parts and treat every S3 event as a notification that may be duplicated or arrive out of order."
tags:
  - "aws"
  - "storage"
  - "s3"
---

# Amazon S3: The Retouching Lab

> Move large objects in recoverable parts and treat every S3 event as a notification that may be duplicated or arrive out of order.

## The Business Goal

A client uploaded a 40-gigabyte RAW package.

At 96 percent, the network failed.

The second attempt completed, but the lab received two arrival tickets. The later ticket reached the retoucher first. Then the retoucher wrote a preview back into the same intake location and triggered the lab again.

One upload had created a retry, a duplicate, an ordering problem, and a loop.

---

## The Story

Mira changed the intake process.

Large deliveries were divided into numbered crates. Workers could resend one damaged crate without restarting the truck. The receiving clerk assembled the album only after every required crate arrived and the manifest passed inspection.

Then the clerk issued a retouching ticket.

The lab assumed tickets could be duplicated or delayed. Every ticket named the object and its event details, and the retoucher checked whether that work had already been completed.

Processed previews went to a separate outgoing shelf.

---

## The Wrong Way

Do not upload a very large object as one fragile request when multipart upload fits.

Do not leave failed multipart parts indefinitely; stored parts incur charges until the upload is completed or aborted.

Do not assume S3 Event Notifications are exactly-once or ordered.

Do not write output to the same triggering key pattern unless the workflow deliberately prevents recursion.

---

## Meet the AWS Service

> **Core idea:** S3 offers SDK operations for object transfer and event notifications for reaction, but application code owns completion, deduplication, and workflow safety.

The AWS SDK can manage multipart upload automatically at higher levels. Lower-level APIs expose initiation, part uploads, listing, completion, and abort operations.

S3 can send notifications to Lambda, SQS Standard, SNS Standard, or EventBridge, subject to destination support and configuration.

---

## How It Works

### Numbered Crates

#### Multipart Upload

Multipart upload divides one object into independently uploaded parts.

```text
CreateMultipartUpload
        |
        +--> UploadPart 1
        +--> UploadPart 2
        +--> UploadPart 3
        |
CompleteMultipartUpload
```

Parts can upload in parallel and failed parts can retry independently. S3 creates the object only after a successful completion request.

Abort abandoned uploads directly or through an S3 Lifecycle rule. Otherwise, uploaded parts continue consuming billable storage.

### The Inspection Manifest

#### Checksums

Checksums let the client and S3 validate data integrity. With multipart uploads, use the SDK and API rules for the selected checksum algorithm and consecutive part numbering.

An ETag should not be treated as a universal MD5 checksum. Multipart uploads and encryption can change its meaning.

### The Contact Sheet

#### Listing and Pagination

`ListObjectsV2` returns a bounded response. If the result is truncated, the application continues with the returned continuation token.

A prefix narrows the key namespace. It is not a database filter applied after retrieving every object.

### The Retouching Ticket

#### S3 Event Notifications

S3 can notify supported destinations about object creation, removal, restore, replication, lifecycle, and other events.

The destination needs a resource policy or permission allowing S3 to publish or invoke it. Direct SQS, SNS, and Lambda destinations must meet regional and destination-type requirements; EventBridge provides broader routing options.

### Duplicate and Late Tickets

#### Idempotency and Ordering

S3 Event Notifications are designed for at-least-once delivery and do not guarantee event order.

A safe consumer:

- Identifies the object and relevant version or event information
- Makes processing idempotent
- Records completion in durable state when duplicate work would matter
- Uses the event sequencer when comparing event order for the same object key
- Handles retries and poison events through the destination's failure controls

### Separate Intake and Output Shelves

#### Loop Prevention

Use separate buckets or prefixes for input and output, and configure filters so generated output does not match the input trigger.

---

## Architectural Mapping

```text
Client
  |
  | multipart upload
  v
S3 intake prefix
  |
  | at-least-once event
  v
Queue / EventBridge / Lambda
  |
  | idempotent processing
  v
S3 output prefix
```

The upload path and event path solve different problems. A successful object write does not guarantee downstream processing has completed.

---

## When to Use It

Use multipart upload when large-object transfer benefits from parallelism and part-level retry. Use notifications when applications should react after S3 records an event.

Use EventBridge when rules, multiple targets, or a supported route to FIFO processing are required. Use a queue between S3 and workers when buffering, backpressure, and retry isolation matter.

## When Not to Use It

Do not treat S3 notifications as a transactional job ledger. When exactly-once business effects matter, combine idempotent consumers with durable state and reconciliation.

---

## Painkiller

> **Problem:** Large transfers fail partway and object events can repeat or arrive late.  
> **Pain:** Whole-file retries waste work, while naive consumers duplicate effects or recurse forever.  
> **AWS solution:** Use multipart transfer, integrity checks, filtered notifications, and idempotent processing.

---

## Knife Cut

> **The object write stores data. The event announces that something happened. The consumer still owns the business outcome.**

---

## The Masthead

### What Actually Just Happened

|In the story|In AWS|What it actually means|
|---|---|---|
|Numbered crates|Multipart upload parts|Independent pieces of one future object|
|Final assembly|CompleteMultipartUpload|Operation that creates the completed object|
|Inspection manifest|Checksum|Integrity validation|
|Contact-sheet continuation|Pagination token|Cursor for the next listing response|
|Retouching ticket|S3 Event Notification|At-least-once event message|
|Completed-work register|Idempotency state|Durable record preventing duplicate effects|

---

## A Note From the Author

The story gives every delivery one tidy set of crates. Real SDK retries, network timeouts, concurrent writes, and abandoned uploads create ambiguous states. Use supported SDK abstractions where possible and explicitly clean up unfinished uploads.

S3 events can be duplicated and unordered. A notification is evidence to inspect—not proof that this consumer has never seen the work before.

- [Multipart upload overview](https://docs.aws.amazon.com/AmazonS3/latest/userguide/mpuoverview.html)
- [S3 event notification destinations](https://docs.aws.amazon.com/AmazonS3/latest/userguide/notification-how-to-event-types-and-destinations.html)

---

## The Last Bite

The lab became reliable when it stopped trusting perfect roads and perfect tickets.

> **Retry the parts. Verify the object. Distrust the uniqueness of the event.**

---

**Next chapter:** *[Amazon S3: The Negative Vault](06-the-negative-vault.md)*

The lab can process arrivals safely. Now Mira must survive overwrites, deletions, and records that are not allowed to change.
