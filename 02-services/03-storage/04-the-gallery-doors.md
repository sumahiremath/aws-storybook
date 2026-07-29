---
description: "Keep the bucket private by default, grant only the required actions, and use temporary signed access when outsiders need one object."
tags:
  - "aws"
  - "storage"
  - "s3"
---

# Amazon S3: The Gallery Doors

> Keep the bucket private by default, grant only the required actions, and use temporary signed access when outsiders need one object.

## The Business Goal

An assistant needed to share one approved portrait with one client for ten minutes.

His first idea was to open the gallery to everyone.

It would certainly let the client in.

It would also let the internet try every other door.

The requested access was narrow. The proposed permission was enormous.

---

## The Story

Mira gave staff badges that described what each role could do.

The gallery itself also kept a door policy describing which visitors it accepted.

A large red safety bar prevented accidental public opening. For a client, Mira issued a signed guest pass naming one object, one operation, and an expiration.

The pass did not make the gallery public.

It temporarily exercised the authority of the person who signed it.

---

## The Wrong Way

Do not make a bucket public merely to serve private downloads or browser uploads.

Do not put long-lived AWS credentials in a browser or mobile application.

Do not assume an `Allow` in one policy defeats an explicit `Deny`, an organization control, a permissions boundary, or missing KMS permission.

S3 access is the result of policy evaluation, not a single toggle.

---

## Meet the AWS Service

> **Core idea:** S3 requests are authorized through IAM and resource policies; presigned URLs delegate a specific signed request for a limited time.

Identity policies attach permissions to principals. Bucket policies attach resource-based permissions to a bucket. S3 Block Public Access adds guardrails against public exposure.

Object Ownership controls can keep the bucket owner responsible for objects and disable ACL-based ownership complications. Modern designs generally prefer policies over ACLs.

---

## How It Works

### Staff Badges

#### IAM Identity Policies

An application role might receive `s3:GetObject` for:

```text
arn:aws:s3:::portrait-gallery/clients/123/*
```

Listing the bucket is a different action on the bucket resource. Least privilege must account for both the action and the correct ARN shape.

### The Gallery's Door Policy

#### Bucket Policy

A bucket policy can grant cross-account access, require transport conditions, restrict principals, or deny requests that violate a rule.

An explicit deny wins over an allow. Troubleshooting must consider all applicable policy layers.

### The Red Safety Bar

#### S3 Block Public Access

Block Public Access settings help prevent public policies or ACLs from exposing data. They can operate at account and bucket levels.

They do not replace careful policy design. They are an additional guardrail.

### The Locked Cabinets

#### Encryption

S3 encrypts new uploads at rest by default with S3-managed encryption. SSE-KMS uses an AWS KMS key and adds KMS authorization and audit considerations.

An application can have S3 permission and still receive an access error when it lacks the required KMS permission or the key policy does not permit the operation.

TLS protects data in transit. Bucket policies can deny requests that do not use secure transport.

### The Signed Guest Pass

#### Presigned URLs

A trusted backend uses its AWS credentials to sign a time-limited `GET` or `PUT` request. The client uses the URL without receiving AWS credentials.

The URL cannot grant more access than the signing principal possesses. Its practical lifetime can also be limited by the underlying temporary credentials and policy conditions.

For uploads, using an existing key can replace the current object at that key. The backend should generate controlled keys and validate the resulting object before trusting it.

### The Browser Reception Desk

#### CORS

Cross-Origin Resource Sharing tells a browser whether frontend code from one origin may call S3 at another origin.

CORS is not authorization. A request can satisfy CORS and still be denied by IAM—or be authorized but blocked by the browser because CORS is absent.

---

## Architectural Mapping

```text
Client -> application backend -> presigned URL
   |                              |
   `------------------------------v
                         one signed S3 request
                                  |
                   IAM + bucket policy + guardrails
                                  |
                              S3 object
```

The backend decides the key and operation before signing. The browser sends or receives the bytes directly, which avoids routing large objects through the backend.

---

## When to Use It

Use:

- Application roles for workloads calling S3
- Bucket policies for resource-level and cross-account rules
- Block Public Access as a public-exposure guardrail
- SSE-KMS when key control and KMS auditing requirements justify it
- Presigned URLs for temporary direct upload or download

## When Not to Use It

Presigned URLs are bearer-style capabilities. Anyone who obtains a valid URL can use it within its constraints. Use another authorization flow when every request needs fresh application-level policy evaluation or immediate revocation behavior.

---

## Painkiller

> **Problem:** Staff, applications, and clients need different slices of gallery access.  
> **Pain:** Public buckets and distributed credentials create excessive exposure.  
> **AWS solution:** Combine least-privilege policies, public-access guardrails, encryption, and narrowly signed temporary requests.

---

## Knife Cut

> **CORS answers “may this browser make the call?” IAM answers “may this principal perform the action?”**

---

## The Masthead

### What Actually Just Happened

|In the story|In AWS|What it actually means|
|---|---|---|
|Staff badge|IAM role policy|Actions an application identity may perform|
|Door policy|Bucket policy|Resource-based rules for the bucket and objects|
|Safety bar|Block Public Access|Guardrail against public exposure|
|Locked cabinet|S3 encryption|Protection for stored object data|
|Guest pass|Presigned URL|Time-limited signed request using signer authority|
|Reception rule|CORS|Browser cross-origin permission, not AWS authorization|

---

## A Note From the Author

A presigned URL is not a session with a guard who can reconsider context halfway through. Treat it as sensitive while valid and keep its scope and lifetime narrow.

Authorization can involve IAM policies, bucket policies, access point policies, KMS key policies, permissions boundaries, session policies, service control policies, and explicit denies.

- [S3 access control](https://docs.aws.amazon.com/AmazonS3/latest/userguide/access-management.html)
- [S3 presigned URLs](https://docs.aws.amazon.com/AmazonS3/latest/userguide/using-presigned-url.html)

---

## The Last Bite

The client needed one photograph.

Mira gave one temporary pass—not the gallery keys.

> **Narrow access is a design, not a public bucket with a short explanation.**

---

**Next chapter:** *[Amazon S3: The Retouching Lab](05-the-retouching-lab.md)*

Clients can now deliver large RAW packages safely. The lab must receive them, verify them, and react without processing the same arrival twice.
