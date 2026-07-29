---
description: "Byte Burger is secure not when it has the most controls, but when every request meets the right control at the right boundary."
tags:
  - "aws"
  - "identity-security"
  - "security"
  - "epilogue"
---

# AWS Identity and Security: Many Boundaries, One Restaurant

> Byte Burger is secure not when it has the most controls, but when every request meets the right control at the right boundary.

## The Business Goal

After the security section, a reader can name Cognito, IAM, STS, KMS, Parameter Store, Secrets Manager, and WAF. Naming them is not yet a design.

The dangerous mistakes happen between them: treating a token as AWS permission, putting secrets in configuration, granting a Lambda every action, assuming encryption allows decryption, or using a web firewall as authorization.

---

## The Story

At closing time, the restaurant manager traces one order.

The customer entered through the guest list. The public request passed the front-door screen. The checkout function wore a staff badge with only the permissions it needed. It retrieved a rotating payment credential from the guard, and KMS governed the cryptographic authority behind protected data.

No individual control claimed to secure the whole restaurant. Together, their boundaries made the path understandable.

---

## Meet the AWS Service

The security services in this section form a layered application path:

| Layer | Primary service | Core responsibility |
| --- | --- | --- |
| Public request filtering | WAF | Inspect and control HTTP(S) requests |
| Customer identity | Cognito | Sign-up, sign-in, federation, tokens |
| AWS workload permission | IAM and STS | Roles, policies, temporary credentials |
| Configuration and credentials | Parameter Store / Secrets Manager | Store and retrieve values; rotate secrets where needed |
| Cryptographic authority | KMS | Govern key use and encryption operations |

## How It Works

Design a request path by asking, in order:

1. Can this web request reach the public application?
2. Who is the customer or calling workload?
3. Which exact action on which resource is permitted?
4. Which secret or configuration value is needed, and how is it protected or rotated?
5. Which key policy, IAM permission, grant, or encryption context governs cryptographic use?
6. Which logs, metrics, and audit events will reveal denial, misuse, or unexpected change?

The answer is usually multiple controls with narrow responsibility, not one broad policy.

---

## Architectural Mapping

| Story path | AWS boundary | Common confusion avoided |
| --- | --- | --- |
| Customer enters | Cognito | Identity is not AWS-resource permission |
| Request passes screen | WAF | Filtering is not business authorization |
| Runner receives badge | IAM/STS | Temporary credentials are not a new human identity |
| Runner receives secret | Secrets Manager / Parameter Store | A secure value is not automatically rotated |
| Vault approves key use | KMS | Encryption is governed access, not possession |

## When to Use It

Use this model when reviewing an application request end-to-end, a new integration, or an `AccessDenied` failure.

## When Not to Use It

Do not add every control to every path. Apply the controls required by the resource, trust boundary, data sensitivity, and public exposure.

---

## Painkiller

> **Problem:** Security failures often occur in the gaps between identity, permission, secrets, encryption, and web traffic.  
> **Pain:** A broad control obscures ownership and creates excessive access.  
> **AWS solution:** Build a path of small, observable boundaries with clear responsibilities.

---

## The Masthead

### What Actually Just Happened

| In the story | In AWS | Precise technical meaning |
| --- | --- | --- |
| Protected order path | layered controls | Each boundary evaluates a different concern |
| Narrow staff badge | least-privilege IAM role | Minimum actions and resources needed |
| Guard’s rotating credential | Secrets Manager | Secret retrieval and lifecycle management |
| Change record | CloudTrail/CloudWatch | Audit and operational evidence |

---

## A Note From the Author

The restaurant path is deliberately linear; real applications have multiple callers, accounts, regions, resource policies, SCPs, VPC boundaries, and third-party identity providers. The useful discipline remains the same: state the principal, action, resource, data sensitivity, trust boundary, and evidence path before granting access.

---

## The Last Bite

Security becomes manageable when every control has one clear job and every request has an explainable path.

---

**Next section:** *[AWS Compute: The Wand Chooses the Wizard](../02-compute/00-compute-wizards-wand.md)*

Security has established who or what may act and which boundaries govern the request. Now Byte Burger must choose where the authorized work will actually run.
