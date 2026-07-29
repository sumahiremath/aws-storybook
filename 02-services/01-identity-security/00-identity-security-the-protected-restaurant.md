---
description: "Orient identity and security decisions around the distinct boundaries that authenticate, authorize, protect, and audit Byte Burger."
tags:
  - "aws"
  - "identity-security"
  - "security"
  - "orientation"
---

# AWS Identity and Security: The Protected Restaurant

## The Business Goal

At 8:40 on Monday morning, Byte Burger's owner opens the new online-ordering launch checklist.

Customers will create accounts. Store managers will refund orders. Delivery partners will send updates. The checkout service will call a payment provider. A public promotion will send strangers to the site.

At the bottom of the list, someone has written one final task:

> Make it secure.

That word is too small for the job.

Customers need to sign in. Applications need permission to call AWS services. Temporary staff need limited access. Secrets must not appear in source code. Encryption keys must be governed. Public traffic needs screening before it reaches the ordering counter.

One guard cannot do all of that well.

---

## The Story

Byte Burger does not build one giant security office. It builds a protected restaurant with several specialized boundaries.

The **guest list** recognizes customers. The **capes and temporary badges** decide which powers a person or workload may assume. The **Elder Wand vault** governs cryptographic authority. The **filing cabinet** stores configuration; the **private security guard** handles rotating secrets. The **front-door screen** filters suspicious web traffic before it burdens the counter.

These are intentionally different scenes because they answer different questions.

---

## Meet the AWS Service

This section covers Amazon Cognito, IAM, STS, KMS, Systems Manager Parameter Store, Secrets Manager, and AWS WAF.

> **Core idea:** Security begins by naming the boundary: identity, permission, temporary access, cryptographic authority, secret lifecycle, or public web-request filtering.

## How It Works

| Security question | Primary services | What the question means |
| --- | --- | --- |
| Who is the customer? | Cognito | Application authentication and tokens |
| What may this principal do? | IAM | AWS authorization through policies and roles |
| Can access be temporary? | STS | Short-lived credentials from role assumption or federation |
| Who may use encryption authority? | KMS | Key policy, IAM, grants, and cryptographic operations |
| Where does configuration or a secret live? | Parameter Store / Secrets Manager | Storage, access, rotation, and retrieval choices |
| Should this HTTP request reach the application? | WAF | Web-request inspection and filtering |

The services overlap at real boundaries. A Cognito-authenticated customer may call API Gateway; API authorization may validate a token; Lambda assumes an execution role; that role reads a secret and uses KMS; WAF filters unsuitable public requests before the API receives them.

---

## Architectural Mapping

```text
Public request → WAF screen → API/application
                         ↓
Customer identity → Cognito token → application authorization
Workload identity → IAM role / STS credentials → AWS resources
Configuration and secrets → Parameter Store / Secrets Manager → KMS-protected data
```

The diagram is layered on purpose. Passing one boundary does not automatically grant access through the others.

## When to Use It

Use this section as a map when a security problem feels vague. First identify the exact question, then choose the service that owns that boundary.

## When Not to Use It

Do not treat a successful login as complete authorization, encryption as complete access control, or WAF as a replacement for secure application behavior.

---

## Painkiller

> **Problem:** “Security” combines several unrelated responsibilities.  
> **Pain:** One broad control becomes over-permissioned, hard to operate, and easy to misunderstand.  
> **AWS solution:** Use distinct identity, permission, credential, key, secret, and request-filtering controls at their proper boundaries.

---

## Knife Cut

> **Authentication answers “who are you?” Authorization answers “may you do this?” Encryption answers “who may use this cryptographic protection?” WAF answers “should this web request reach the application?”**

---

## The Masthead

### What Actually Just Happened

| In the story | In AWS | Precise meaning |
| --- | --- | --- |
| Guest list | Cognito | Application identity and token issuance |
| Cape / badge | IAM role and STS | AWS permissions and temporary credentials |
| Wand vault | KMS | Controlled cryptographic key use |
| Filing cabinet / guard | Parameter Store / Secrets Manager | Configuration and secret lifecycle |
| Front-door screen | WAF | HTTP(S) request filtering |

---

## A Note From the Author

This is intentionally a collection of analogies, not one continuous fantasy or restaurant scene. Security is a layered discipline, and forcing all of it through one metaphor would blur the boundaries readers need to remember. The recurring structure—not one setting—is the unifier: identify the question, locate the boundary, apply least privilege, observe the outcome, and preserve a recovery path.

---

## The Last Bite

Good security is not one perfect guard. It is several clear boundaries, each answering one question well.

---

**Next chapter:** *[Amazon Cognito: The Tiny Company’s Guest List](01-cognito-guest-list.md)*

The first boundary is the person at the door: how does Byte Burger recognize an application customer without building an identity system from scratch?
