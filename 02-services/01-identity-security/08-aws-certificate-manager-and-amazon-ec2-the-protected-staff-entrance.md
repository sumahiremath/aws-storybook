---
description: "Separate proof of a server’s identity from controlled administrative machine access, and manage each credential as a lifecycle."
tags:
  - "aws"
  - "identity-security"
  - "security"
  - "acm"
  - "private-ca"
  - "ssh"
  - "ec2"
---

# AWS Certificate Manager and Amazon EC2: The Protected Staff Entrance

## The Business Goal

Byte Burger needs customers to know that `order.byteburger.com` is truly its ordering site. It also needs a tightly controlled way for an operator to reach an EC2 kitchen station when an exceptional repair is necessary. Those are not the same trust problem.

## The Story

At the public entrance, the restaurant presents a verifiable sign that proves the building’s identity before a customer shares an order. At the staff entrance, an approved operator presents a key that matches the lock. Neither credential should be copied into a notebook, shared casually, or forgotten until it expires.

## Meet the Credentials

TLS certificates authenticate a server to clients and protect traffic in transit. AWS Certificate Manager manages public certificates for supported AWS integrations; AWS Private CA can issue private certificates where an organization needs its own trusted internal identity system.

An SSH key pair is different: its public key is placed on a compute host and the private key proves an operator possesses the matching credential. It is an administrative access mechanism, not an application secret, API token, or TLS certificate.

## How It Works

Use the smallest suitable trust path:

- Validate domain control and attach the right certificate to the endpoint that serves TLS.
- Plan renewal and replacement before certificate expiration; use managed renewal where the integration supports it.
- Generate, store, rotate, and revoke administrative keys deliberately. Limit who can use them and which hosts accept them.
- Prefer short-lived, auditable access mechanisms when the workload supports them; do not leave a widely shared private key as the default repair path.

## Knife Cut

> A certificate proves **the server to the client**. An SSH key proves **the administrator to the host**.

## A Note From the Author

The entrance compresses two different trust systems into one scene. ACM managed renewal depends on certificate eligibility and supported AWS integration. EC2 key pairs do not rotate themselves, recover a lost private key, or decide who should retain administrative access; those remain customer responsibilities.

- [AWS Certificate Manager managed renewal](https://docs.aws.amazon.com/acm/latest/userguide/managed-renewal.html)
- [Amazon EC2 key pairs](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-key-pairs.html)

## The Last Bite

The protected entrance works only when the restaurant knows which identity is being proved, to whom, and for how long.

**Next chapter:** *[AWS Identity and Security: Many Boundaries, One Restaurant](09-identity-security-many-boundaries-one-restaurant.md)*

The restaurant now has identities, permissions, secrets, keys, filters, certificates, and administrative credentials. The final chapter follows one request through those distinct boundaries without confusing one control for another.
