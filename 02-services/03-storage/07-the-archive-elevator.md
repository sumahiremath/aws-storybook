---
description: "Match storage class to access behavior, then automate transitions and expiration without confusing cheap storage with free retrieval."
tags:
  - "aws"
  - "storage"
  - "s3"
---

# Amazon S3: The Archive Elevator

> Match storage class to access behavior, then automate transitions and expiration without confusing cheap storage with free retrieval.

## The Business Goal

The Bellweather gallery was busiest during its first month.

A year later, almost nobody opened it. Ten years later, the studio still paid to keep every RAW file on its most active shelf.

Mira moved everything to the deepest archive.

The next morning, a client asked for one photograph immediately.

The storage was cheap.

The promise to the client was not.

---

## The Story

Mira installed an archive elevator.

New photographs stayed in the bright gallery. Collections with unpredictable attention moved between staffed rooms automatically. Rarely requested albums moved downstairs. The deepest vault cost less, but an attendant had to restore an album before a visitor could view it.

Every shelf had rules: retrieval charges, minimum stays, resilience boundaries, and waiting time.

The elevator followed labels and age. It did not understand the sentimental value of a photograph.

---

## The Wrong Way

The lowest per-gigabyte price is not automatically the lowest total cost.

Short-lived objects can incur minimum-duration charges in some classes. Small objects can interact badly with minimum billable sizes or per-object monitoring charges. Retrieval and request costs can outweigh storage savings.

Lifecycle expiration can permanently remove data. A rule is automation, not judgment.

---

## Meet the AWS Service

> **Core idea:** S3 storage classes trade access pattern, availability design, retrieval behavior, and cost; S3 Lifecycle automates transitions and expiration for matching objects.

All current S3 storage classes are designed for high durability, but they do not have identical Availability Zone resilience, access latency, retrieval workflow, or pricing dimensions.

---

## How It Works

### The Bright Gallery

#### S3 Standard

S3 Standard fits frequently accessed data with millisecond access and multi-Availability Zone resilience.

### The Self-Adjusting Room

#### S3 Intelligent-Tiering

Intelligent-Tiering monitors access and can move eligible objects among access tiers. It fits unknown or changing patterns.

It has per-object monitoring and automation considerations. Archive access tiers require restore behavior when enabled and used.

### The High-Speed Editing Bay

#### S3 Express One Zone

S3 Express One Zone is a high-performance, single-Availability Zone storage class used with directory buckets. It is designed for latency-sensitive object workloads and has different endpoint, naming, API, and resilience considerations from S3 Standard in a general purpose bucket.

Choose it because the workload needs its access model and performance—not merely because "Express" sounds universally better.

### The Quiet Shelves

#### Standard-IA and One Zone-IA

Infrequent Access classes keep millisecond access but add retrieval charges and minimum-duration considerations.

Standard-IA is resilient across multiple Availability Zones. One Zone-IA stores data in one Availability Zone and fits recreatable or secondary data when that resilience trade-off is acceptable.

### The Vault

#### Glacier Instant, Flexible, and Deep Archive

S3 Glacier Instant Retrieval provides millisecond access for rarely accessed data.

S3 Glacier Flexible Retrieval and S3 Glacier Deep Archive store archived objects that must be restored before ordinary real-time access. Retrieval choices trade speed and cost. Deep Archive fits the longest waiting tolerance.

These are S3 storage classes. The objects remain managed through S3.

### The Elevator Rules

#### S3 Lifecycle

Lifecycle rules match objects by scope such as prefix, tags, or other supported filters and can:

- Transition current or noncurrent versions
- Expire current versions
- Permanently delete noncurrent versions
- Remove expired delete markers
- Abort incomplete multipart uploads

In a versioning-enabled bucket, expiring a current version normally creates a delete marker. Noncurrent-version expiration is the action that permanently removes old versions when not blocked by retention.

### Calling an Album Back

#### Restore

An archived object in Flexible Retrieval or Deep Archive must be restored before it can be read normally. Restoration creates temporary accessible status for a requested number of days; it does not permanently change the object's archival storage class.

---

## Architectural Mapping

```text
Object age / tag / prefix
          |
          v
     Lifecycle rule
       /       \
 transition   expiration
    |             |
new class     delete marker or permanent removal,
              depending on version state and rule
```

Lifecycle actions occur asynchronously after eligibility. Applications should not build second-by-second workflows around the exact transition time.

---

## When to Use It

Use storage classes and lifecycle when data has:

- Predictable decline in access
- Unknown or changing access suited to Intelligent-Tiering
- Retention requirements with known retrieval expectations
- Old versions or incomplete uploads that need controlled cleanup

## When Not to Use It

Do not archive data that the application must read immediately unless the chosen class provides that access. Do not expire data without understanding versioning, Object Lock, replication, and recovery requirements.

---

## Painkiller

> **Problem:** Access fades while active-tier storage cost continues.  
> **Pain:** Manual movement is inconsistent, while indiscriminate archiving breaks retrieval promises.  
> **AWS solution:** Select storage classes by access behavior and automate deliberate transitions and expiration with Lifecycle.

---

## Knife Cut

> **A transition changes the cost-and-access tier. An expiration removes the object's current or historical presence according to version state.**

---

## The Masthead

### What Actually Just Happened

|In the story|In AWS|What it actually means|
|---|---|---|
|Bright gallery|S3 Standard|Frequent access with millisecond retrieval|
|Self-adjusting room|Intelligent-Tiering|Automatic access-tier movement|
|High-speed editing bay|S3 Express One Zone|Low-latency single-AZ object storage in a directory bucket|
|Quiet shelf|Infrequent Access class|Lower storage price with retrieval and duration considerations|
|Underground vault|Glacier storage class|Rare-access or archival object storage|
|Elevator rule|S3 Lifecycle|Automated transition or expiration|
|Calling an album back|Restore request|Temporary access to an archived object|

---

## A Note From the Author

The elevator makes transitions look immediate and reversible. Lifecycle timing is asynchronous, and expiration can permanently delete data.

Pricing dimensions and storage-class features change. Verify current minimum durations, minimum billable sizes, retrieval options, availability design, and charges for the workload.

- [S3 storage classes](https://docs.aws.amazon.com/AmazonS3/latest/userguide/storage-class-intro.html)
- [S3 Lifecycle configuration](https://docs.aws.amazon.com/AmazonS3/latest/userguide/object-lifecycle-mgmt.html)

---

## The Last Bite

Cold memories can live on cheaper shelves.

They cannot promise hot retrieval unless the shelf was chosen for it.

> **Archive according to the wait the business can tolerate, not the price that looks smallest.**

---

**Next chapter:** *[Amazon EBS, Amazon EFS, and Amazon S3: Choose the Right Home](08-choose-the-right-home.md)*

The studio now owns a workbench, a shared darkroom, a gallery, and a vault. Three new projects arrive, and each must go to the right place.
