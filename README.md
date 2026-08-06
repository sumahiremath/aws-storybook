# AWS Storybook

**By Suma Manjunath**

> A narrative learning experience for engineers who want to understand AWS the way systems are actually built: a business needs something, the first reasonable approach reaches a limit, and an engineering decision becomes necessary.

AWS Storybook teaches AWS through operational stories, not service memorization.

It does not begin with a product definition. It begins with a business goal: launch online ordering, protect a customer account, survive a promotion, release a change safely, or recover an incomplete order. The technical constraint arrives next. AWS becomes useful because the situation earns it.

The goal is a mental model a reader can retell accurately in a design review, an incident, a tradeoff discussion, or a whiteboard conversation.

> **Project status:** This repository contains the current published edition of AWS Storybook. Its stories, technical details, and navigation will continue to evolve as AWS and the book grow.

## Why this exists

Most cloud learning material starts with a service and lists its features.

AWS Storybook starts with the moment before the service becomes obvious:

```text
Business goal
      ↓
Reasonable first approach
      ↓
Constraint, pressure, or failure
      ↓
Technical decision
      ↓
AWS service and its real responsibilities
```

Each article makes the boundary explicit:

- What problem does the service solve?
- What does the developer still own?
- What can fail, duplicate, throttle, become stale, or cost money?
- What evidence proves the system is healthy or explains why it is not?
- Where does the analogy help, and where does it stop being reliable?

This is not documentation painted with a metaphor. The story must make the architecture necessary.

## Who this is for

AWS Storybook is for:

- engineers who want durable mental models before reaching for documentation, code, or a console;
- cloud learners who are tired of memorizing service names without understanding when they matter;
- technical leaders, mentors, and study groups who need better ways to explain architectural tradeoffs;
- builders who want AWS concepts tied to production behavior, ownership, failure, and cost.

It is not a shortcut around hands-on work. It is the connective tissue that helps documentation, labs, production reviews, and operational ownership make sense together.

## Repository map

```text
01-infrastructure/   Understand the cloud before choosing a service
02-services/         Learn AWS services through complete operational stories
03-architecture/     Learn when an architectural decision is right,
                     when it expires, and when complexity should be removed
```

### 01 - Infrastructure

The foundations: what AWS is, how managed services divide responsibility, how events change systems, and how to think before selecting technology.

### 02 - Services

The service collection teaches the contracts that make an application work in production. Its recurring worlds are **Bits Boba** for compute, **Mira's photography studio** for storage, and **Byte Burger** for customer-facing systems, operations, and failure.

The photography studio is an intentional third world. Images naturally move from a fast workbench to shared files, object galleries, protected versions, and archives, making the changing job of data visible before the collection returns to Byte Burger's application truth.

These businesses are used to make the engineering problem clear: what does Bits Boba or Byte Burger actually need? Other analogies appear when they make the story narration sharper, because some AWS concepts cannot be forced into one business metaphor without bending the truth.

Each section builds a practical mental model around the decisions, responsibilities, and failure modes engineers encounter when using AWS.

| Section | Central question |
| --- | --- |
| Identity and Security | Who is making this request, what may they do, and which boundary protects it? |
| Compute | What shape of work needs to run? |
| Storage | Where should files and durable objects live? |
| Database | Where should each kind of truth live? |
| APIs and SDKs | How does an application make and govern a request? |
| Application Integration | How should independent parts exchange work, facts, and state? |
| Deployment | How does a change reach production safely? |
| Observability and Optimization | How do we prove what happened and make the next shift better? |
| Networking and Content Delivery | How do requests find the right entrance and travel on intended paths? |
| Systems Under Pressure | What happens when traffic, failure, and change arrive together? |

The final service capstone follows Byte Burger through a Saturday promotion. It connects security, APIs, Lambda, queues, databases, caching, deployment, observability, networking, retries, and recovery into one customer journey.

### 03 - Architecture

`03-architecture` is the published Architecture collection. It is not a service catalogue, and it is not an argument that every successful company must become microservices.

It follows Byte Burger from a small local restaurant toward a multinational business, asking harder questions:

- When is a monolith the simplest correct answer?
- When has a premature service split become a distributed liability?
- When does a modular monolith restore speed and clarity?
- What observable business pressure earns a service extraction, queue, event, workflow, cache, or regional design?
- How will the team know the decision is no longer valid?
- If assumptions change, how can complexity be removed safely?

The working principle is simple:

> **Microservices are not the destination. Appropriate boundaries are.**

## How an article works

Every article opens with **The Business Goal**. The reader meets the person, team, or business outcome first, not an AWS definition.

The rest of the article typically includes:

- **The Story**: a reasonable approach and the pressure that exposes its limit;
- **Meet the AWS Service**: the smallest accurate explanation of what becomes necessary;
- **How It Works**: the developer-facing mechanics, permissions, lifecycle, failure behavior, and tradeoffs that matter;
- **The Masthead**: a precise story-to-AWS mapping;
- **A Note From the Author**: where the analogy breaks and what production still requires;
- **The Last Bite**: the memorable lesson;
- **Next chapter**: the consequence that opens the next question.

## Start here

- For the complete book sequence: use the [`AWS Storybook Reading Order`](./READING-ORDER.md).
- New to AWS: begin with [`01-infrastructure`](./01-infrastructure/).
- Choosing or operating an AWS service: enter [`02-services`](./02-services/).
- Studying how Byte Burger's architecture evolves: enter [`03-architecture`](./03-architecture/).

## License

Written content in this repository is licensed under Creative Commons Attribution-NonCommercial-ShareAlike 4.0 International. Commercial reuse is not permitted without permission. See [LICENSE.md](./LICENSE.md).
