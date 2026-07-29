---
description: "A presigned URL delegates one narrow S3 request for a limited time without giving the recipient AWS credentials."
tags:
  - "aws"
  - "apis"
  - "sdk"
  - "s3"
---

# AWS SDK and Amazon S3: The Temporary Loading Dock Pass

> A presigned URL delegates one narrow S3 request for a limited time without giving the recipient AWS credentials.

## The Business Goal

A customer needed to upload a large menu photograph.

Marco could send the file through the application server, paying for an unnecessary middle hop. Or he could give the browser AWS access keys, which would be like giving every diner the warehouse master badge.

Neither was a loading-dock pass.

## The Story

The operations assistant prepared a temporary dock pass for one bucket, one object key, one operation, and a short time window. The customer used the pass to upload directly to the warehouse. The pass did not reveal the assistant's badge, and it did not let the customer browse the rest of the warehouse.

When the pass expired, the customer had to request another.

## Meet the AWS Service

An SDK can create an Amazon S3 presigned URL for supported operations such as `GetObject` or `PutObject`.

> **Core idea:** The signer delegates the ability to make a specific request under the signer's permissions until the URL expires.

## How It Works

### Signing Scope

The application uses credentials that already have the relevant S3 permission. The SDK signs the request parameters into a URL. S3 evaluates the request using the signing context and applicable policies.

The recipient does not receive reusable AWS credentials. Anyone who obtains the URL before expiry may be able to use it, so distribute it carefully.

### Upload Constraints

For uploads, define the key carefully and validate ownership, file type, size, and intended operation in the application workflow. Browser CORS configuration on the bucket is separate from the presigned permission.

Use multipart upload patterns when object size and reliability require them.

### Expiry and Revocation

The URL expires after its configured validity period, subject to the credentials and policy context used to sign it. It is not a durable sharing model. Changing permissions or invalidating underlying credentials can affect use, but do not design emergency revocation around assumptions; choose short expiry and a controlled issuance path.

## Architectural Mapping

```text
authenticated client -> application authorization -> SDK presigns -> temporary URL -> direct S3 transfer
```

The client receives the URL, not the signer's reusable credentials.

## When to Use It

Use presigned URLs for direct, short-lived client upload or download of a specifically authorized S3 object operation.

## When Not to Use It

Do not use one long-lived URL as a general API. Do not issue a URL for an uncontrolled object key or treat it as user authentication.

## Painkiller

> **Problem:** A client needs one object transfer but should not hold AWS credentials or force the application server to proxy large bytes.  
> **Pain:** Broad credentials expose the warehouse; proxying creates avoidable cost and bottlenecks.  
> **AWS solution:** Use an SDK to issue a short-lived, narrowly scoped S3 presigned request after application authorization.

## Knife Cut

> **A presigned URL delegates a request. It does not turn the recipient into an AWS principal with general access.**

## The Masthead

### What Actually Just Happened

|In the story|In AWS|What it actually means|
|---|---|---|
|Temporary dock pass|Presigned URL|Time-limited signed S3 request|
|Warehouse badge|Signer credentials|IAM-authorized identity creating the URL|
|One labeled pallet|Bucket/key/operation|Narrow request target|
|Pass expiry|URL expiration|Limited delegation window|

## A Note From the Author

Presigning is service- and operation-specific. It does not bypass bucket policy, encryption requirements, object ownership design, CORS, or application validation of the requested key.

- [Amazon S3 presigned URLs](https://docs.aws.amazon.com/AmazonS3/latest/userguide/using-presigned-url.html)

## The Last Bite

The customer reached the loading dock.

They never received the keys to the warehouse.

**Next chapter:** *[Amazon API Gateway and AWS SDK: The Byte Burger Contract](10-amazon-api-gateway-and-aws-sdk-the-restaurant-contract.md)*

Byte Burger now has a public counter, a disciplined internal assistant, and a narrow dock pass. Nia needs one final rule for choosing which boundary belongs to each request.
