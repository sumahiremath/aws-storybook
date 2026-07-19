# STS = Polyjuice Potion

## The Pain

Harry, Ron, and Hermione need to enter the Ministry.

Their own identities won't get them inside.

They don't need permanent new identities.

They need temporary ones.

---

## Meet Polyjuice Potion

Polyjuice Potion doesn't make you a different person forever.

It lets you **assume an IAM role**.

For a limited time.

When it wears off...

You become yourself again.

**That is AWS Security Token Service (STS).**

---

## AssumeRole

Harry doesn't gain new magical powers.

He gains access because everyone believes he is someone else.

Likewise...

STS doesn't create new permissions.

It lets you temporarily **assume an IAM Role**.

The permissions belong to the role.

Not the caller.

---

## Temporary Credentials

Polyjuice always expires.

You can't drink one potion and stay disguised forever.

STS credentials work exactly the same way.

```text
Access Key

Secret Access Key

Session Token

Expiration
```

After expiration...

Gone.

---

## Cross-Account Access

Imagine borrowing Polyjuice from another school.

For one mission...

You temporarily become one of their students.

That's cross-account AssumeRole.

---

## Role Chaining

### The Inception Potion

Sometimes the borrowed identity needs to borrow another identity.

```text
Harry
  |
  v
assumes Ministry Clerk
  |
  v
assumes Undersecretary
```

AWS calls this **role chaining**.

One role session uses its temporary credentials to call `AssumeRole` for another role.

The second role must trust the first role, and the first role must be allowed to assume it.

But chained magic has a shorter clock.

No matter how long the target role normally permits sessions to last, a role obtained through role chaining can last no more than one hour.

> **The more times you layer the potion without returning to a long-term identity, the sooner the borrowed form expires.**

---

## Least Privilege

You don't become Dumbledore.

You receive the permissions available to the role session.

No extra permissions are created. Session policies, permissions boundaries, organization guardrails, and explicit denies can further reduce what the session may do.

---

## Painkiller

> **Problem:** Applications and users sometimes need temporary access without creating permanent credentials.
> **Pain:** Long-lived access keys become security risks.
> **AWS solution:** Use STS to issue temporary credentials by assuming an IAM role.

---

## Knife Cut

IAM answers:

> **Who are you allowed to be?**

STS answers:

> **Which role are you temporarily acting as right now?**

---

## Why AWS Built STS

Without STS, temporary work would often require permanent credentials.

Applications would store long-lived access keys.

People would share credentials across accounts.

Short missions would leave permanent identities behind.

STS separates the permission definition from the credential lifetime.

IAM defines the role and its permissions.

STS issues temporary credentials when a trusted principal needs to use that role.

The mission ends.

The credentials expire.

---

## The Masthead

### What Actually Just Happened

| In the story | In AWS | What it actually means |
| --- | --- | --- |
| Polyjuice Potion | STS | Temporary credentials |
| Borrowed identity | `AssumeRole` | Use another IAM role |
| Potion duration | Session duration | Credentials expire |
| Returning to yourself | Expiration | Credentials become invalid |
| Borrowing a Ministry identity | Cross-account `AssumeRole` | Temporary cross-account access |
| Layered potion | Role chaining | One role session assumes another role; the chained session is limited to one hour |

---

## A Note From the Author

Polyjuice Potion captures the core intuition behind STS remarkably well, but like every story, it simplifies reality.

**You never permanently become someone else.** STS issues temporary security credentials. When they expire, they're useless. Unlike a long-lived IAM access key, they aren't meant to live forever.

**The permissions aren't yours.** When you assume a role, you're borrowing that role's permissions for a limited time. STS doesn't create new permissions. IAM still decides what the assumed role is allowed to do.

**Nobody copies identities.** Polyjuice physically transforms the drinker. STS doesn't. AWS says, "For a limited session, you may act as this IAM role," and issues temporary credentials that prove it.

**Trust still matters.** In Harry Potter, anyone with the right hair can brew Polyjuice. In AWS, a role must explicitly trust you before you can assume it. Without a trust relationship, the transformation never happens.

**Effective permissions can be narrower than the role policy.** Session policies, permissions boundaries, organization guardrails, resource policies, and explicit denies can all affect the final decision.

**The potion does not actually become unstable.** That is only the memory device. AWS deliberately limits chained role sessions to one hour; it is an authorization-session rule, not a degradation of the temporary credentials.

Use the story to remember the idea.

Use the documentation to build secure systems.

---

## The Last Bite

STS does not give you a permanent new identity.

It gives you temporary credentials for a trusted role and a bounded mission.

---

**Next chapter:** _The Elder Wand Answers to Its True Owner: KMS as Controlled Cryptographic Power_

STS explains how AWS lends temporary credentials without creating another permanent identity.

Next, we will explore how AWS KMS keeps cryptographic power protected and allows only authorized callers to use it.
