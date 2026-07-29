---
description: "Store durable objects in buckets and let applications address them by key through an API."
tags:
  - "aws"
  - "storage"
  - "s3"
---

# Amazon S3: The Gallery of Objects

> Store durable objects in buckets and let applications address them by key through an API.

## The Business Goal

The Bellweather album was finished, but clients were still entering the studio's shared darkroom to view it.

That meant mounting a filesystem, navigating project folders, and risking access to working files.

Clients did not need a place to edit photographs.

They needed a place to request finished photographs.

---

## The Story

Mira opened a gallery.

Every finished photograph entered a named building, received a precise catalogue label, and carried a card describing its content type and other metadata.

A visitor did not wander through a disk. The visitor requested:

> “Bring me the object with this catalogue key.”

The gallery returned the photograph—or said that key did not exist.

The storage model had changed from shared files to addressed objects.

---

## The Wrong Way

An S3 key can contain slashes, and consoles display those slashes like folders. That does not make a general purpose S3 bucket a mounted hierarchical filesystem.

Renaming a large "folder" is not merely changing one directory entry. Applications typically copy objects to new keys and delete the old objects.

Design the key namespace for API access, listing, lifecycle, events, and human comprehension—not because it secretly behaves like a local disk.

---

## Meet the AWS Service

> **Core idea:** Amazon S3 stores an object's bytes and metadata under a key inside a bucket.

Buckets exist in an AWS Region and their names must satisfy the applicable naming rules. A general purpose bucket is a container and policy boundary. An object key identifies an object within that bucket.

AWS manages the distributed storage infrastructure and durability. You manage names, access, encryption choices, lifecycle, replication, event handling, and application behavior.

---

## How It Works

### The Gallery Building

#### Bucket

A bucket groups objects and carries configuration such as policies, versioning, lifecycle rules, notifications, and encryption settings.

Bucket names are not the same thing as object keys. Permissions may apply to the bucket, the objects, or both, depending on the action.

### The Catalogue Label

#### Object Key

The key is the object's name inside the bucket:

```text
weddings/bellweather/final/portrait.jpg
```

The apparent prefixes help organize listing and policy patterns. They are part of one key namespace.

### The Photograph and Its Card

#### Object Data and Metadata

The object contains bytes plus system-defined and user-defined metadata. Content type, cache behavior, tags, checksums, and custom metadata influence how applications manage or deliver it.

Metadata is not a database query engine. If an application needs rich search across millions of photographs, it commonly keeps searchable attributes in a database or search service and stores the binary object in S3.

### The Gallery Requests

#### S3 APIs

Common application operations include:

- `PutObject` to create or replace an object
- `GetObject` to retrieve object data
- `HeadObject` to retrieve metadata without the object body
- `DeleteObject` to delete or place a delete marker, depending on versioning
- `ListObjectsV2` to list keys

The AWS SDK handles request signing, retries, and response parsing, but application code must still handle errors and pagination.

### A Catalogue That Reflects the Latest Intake

#### Strong Consistency

After a successful `PUT` or `DELETE`, subsequent `GET` and `LIST` requests are strongly consistent. An overwrite is atomic for a single key: a reader receives the old object or the new object, not a half-written blend.

Strong consistency does not coordinate two different keys as one transaction.

---

## Architectural Mapping

```text
Application
   |
   | signed S3 API request
   v
Bucket + object key
   |
   +--> object bytes
   `--> metadata, tags, version information
```

A successful SDK response is the application's boundary for the operation. Timeouts and retries still require care: if a client loses the response, it may need to determine whether the operation succeeded before repeating a nontrivial workflow.

---

## When to Use It

Use S3 when:

- Applications store and retrieve objects through APIs
- Data includes images, videos, documents, logs, backups, or static assets
- Massive scale and high durability matter
- Objects need lifecycle, versioning, replication, events, or controlled sharing

## When Not to Use It

Choose a filesystem when applications require mounted file semantics, file locking, or in-place edits. Choose a database when the main requirement is transactions and flexible record queries.

---

## Painkiller

> **Problem:** Finished assets need durable, scalable, API-addressable storage.  
> **Pain:** Exposing a working filesystem couples clients to internal paths and permissions.  
> **AWS solution:** S3 stores each asset as an object addressed by a key inside a bucket.

---

## Knife Cut

> **A key may look like a path, but an S3 object is requested through an object API—not opened as an ordinary filesystem inode.**

---

## The Masthead

### What Actually Just Happened

|In the story|In AWS|What it actually means|
|---|---|---|
|Gallery building|S3 bucket|Regional container and configuration boundary|
|Catalogue label|Object key|Identifier unique within a bucket|
|Photograph|Object data|Stored bytes|
|Catalogue card|Metadata and tags|Information used to describe and manage an object|
|Gallery request|S3 API operation|Signed request to create, retrieve, inspect, list, or delete|

---

## A Note From the Author

S3 provides strong read-after-write consistency, including `GET` and `LIST` behavior after successful writes. That does not make a sequence of operations across multiple objects transactional.

Service quotas, object-size rules, request rates, costs, network paths, and permissions still shape production designs.

- [What is Amazon S3?](https://docs.aws.amazon.com/AmazonS3/latest/userguide/Welcome.html)

---

## The Last Bite

The client did not enter the darkroom.

The client asked the gallery for one named object.

> **S3 begins when the application needs an object address, not a mounted drive.**

---

**Next chapter:** *[Amazon S3: The Gallery Doors](04-the-gallery-doors.md)*

The gallery can hold every photograph. Now Mira must decide who may reach each one.

