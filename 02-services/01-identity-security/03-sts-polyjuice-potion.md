
## STS = Polyjuice Potion

### The Pain

Harry, Ron, and Hermione need to enter the Ministry.

Their own identities won't get them inside.

They don't need permanent new identities.

They need temporary ones.

---

## Meet Polyjuice Potion

Polyjuice Potion doesn't make you a different person forever.

It lets you **assume someone else's identity**.

For a limited time.

When it wears off...

You become yourself again.

**That is AWS Security Token Service (STS).**

---

### AssumeRole

Harry doesn't gain new magical powers.

He gains access because everyone believes he is someone else.

Likewise...

STS doesn't create new permissions.

It lets you temporarily **assume an IAM Role**.

The permissions belong to the role.

Not the caller.

---

### Temporary Credentials

Polyjuice always expires.

You can't drink one potion and stay disguised forever.

STS credentials work exactly the same way.

```
Access Key

Secret Access Key

Session Token

Expiration
```

After expiration...

Gone.

---

### Cross Account

Imagine borrowing Polyjuice from another school.

For one mission...

You temporarily become one of their students.

That's cross-account AssumeRole.

---

### Least Privilege

You don't become Dumbledore.

You become exactly the person whose identity you borrowed.

No extra permissions.

Only theirs.

---

## Painkiller

> **Problem:** Applications and users sometimes need temporary access without creating permanent credentials.
> **Pain:** Long-lived access keys become security risks.
> **AWS Solution:** Use STS to issue temporary credentials by assuming an IAM role.

---

## Knife Cut

IAM answers:

> **Who are you allowed to be?**

STS answers:

> **Who are you pretending to be right now?**

---

## Masthead

|Story|AWS|Meaning|
|---|---|---|
|Polyjuice Potion|STS|Temporary credentials|
|Borrowed identity|AssumeRole|Use another IAM role|
|Potion duration|Session Duration|Credentials expire|
|Returning to yourself|Expiration|Credentials become invalid|
|Borrowing ministry identity|Cross-account AssumeRole|Temporary cross-account access|

---
# A Note From The Author

Polyjuice Potion captures the core intuition behind STS remarkably well, but like every story, it simplifies reality.

**You never permanently become someone else.** STS issues temporary security credentials. When they expire, they're useless. Unlike a long-lived IAM access key, they aren't meant to live forever.

**The permissions aren't yours.** When you assume a role, you're borrowing that role's permissions for a limited time. STS doesn't create new permissions. IAM still decides what the assumed role is allowed to do.

**Nobody copies identities.** Polyjuice physically transforms the drinker. STS doesn't. AWS simply says, "For the next hour, you may act as this IAM role," and issues temporary credentials that prove it.

**Trust still matters.** In Harry Potter, anyone with the right hair can brew Polyjuice. In AWS, a role must explicitly trust you before you can assume it. Without a trust relationship, the transformation never happens.

Use the story to remember the idea.

Use the documentation to build secure systems.