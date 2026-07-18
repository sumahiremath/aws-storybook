# How to Think Like an Architect

## The Pain

The first time people open the AWS Console, they see hundreds of services.

Their first question becomes:
> Which one do I use?

That is the wrong question.

Architects don't start with services.
They start with problems.

---

## Imagine Hiring People

Suppose you need help building a house.

You don't begin by asking:
> Should I hire an electrician?

You ask:
> What problem am I trying to solve?

Need plumbing?
Hire a plumber.

Need wiring?
Hire an electrician.

Need roofing?
Hire a roofer.

The profession follows the problem.

AWS works exactly the same way.

---

## Never Start With the Service

Bad thinking:
> I want to use Lambda.

Good thinking:
> I need code that only runs when something happens.

Then Lambda becomes obvious.

---

Bad thinking:
> I want DynamoDB.

Good thinking:
> I need millisecond lookups at massive scale.

---

Bad thinking:
> I want Kubernetes.

Good thinking:
> I need portable container orchestration.

---

## Every AWS Service Exists Because of Pain

Ask these questions first.

### Need compute?

Someone must execute code.

↓

EC2

Lambda

Fargate

ECS

EKS

---

### Need storage?

Need to save something.

↓

S3

EBS

EFS

---

### Need a database?

Need structured data.

↓

RDS

Aurora

DynamoDB

ElastiCache

---

### Need communication?

Systems must talk.

↓

SQS

SNS

EventBridge

---

### Need networking?

People must reach your application.

↓

Route 53

CloudFront

API Gateway

Global Accelerator

VPC

---

### Need identity?

Someone must prove who they are.

↓

IAM

Cognito

STS

---

### Need security, cryptography, or secrets?

Data and credentials must be protected.

↓

KMS

Secrets Manager

---

## The Architect's Questions

Every time you see an AWS problem ask:

What is hurting?

What am I storing?

What needs to communicate?

Who needs access?

Does something happen because of an event?

Does this need to run continuously?

Where is the bottleneck?

What are the cost, latency, durability, scale, compliance, and operational ownership requirements?

---

## Architectural Mapping

```
Business Problem
        │
        ▼
Identify the Pain
        │
        ▼
Choose the Category
        │
        ▼
Choose the AWS Service
        │
        ▼
Build the Solution
```

Notice the service is the **fourth** step.

Not the first.

---

## Knife Cut

AWS isn't a toolbox.

It's a hospital.

You don't walk into a hospital saying:

> I want surgery.

You describe your symptoms.

The doctor recommends the treatment.

Architects work the same way.

They diagnose before prescribing.

---

## Last Bite

People who memorize AWS services forget them.

People who understand the problems each service solves can usually predict the right answer, even when AWS launches a brand-new service.

---

**Next section:** _AWS Services_

Now that you know how an architect thinks, it is time to meet the services that solve those problems.

From here, we will explore AWS one service at a time: what pain it removes, how it works, and when to choose it.
