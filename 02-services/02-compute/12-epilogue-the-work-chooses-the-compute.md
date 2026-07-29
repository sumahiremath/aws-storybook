---
description: "The Compute section began with one question: what kind of work needs to run?"
tags:
  - "aws"
  - "compute"
  - "epilogue"
---

# AWS Compute: The Work Chooses the Compute

> *“The wand chooses the wizard, Mr. Potter.”* — Ollivander

---

## The Business Goal

### Back to Bits Boba

The Compute section began with one question: what kind of work needs to run?

Some work needed a permanent shop with full control: EC2. Some needed identical shops behind one host: AMIs, Auto Scaling, and Elastic Load Balancing. Some needed portable kitchens: containers, ECS, Fargate, ECR, and EKS. Some needed a catering company to finish a finite backlog: Batch. And some needed a runner for one bounded errand: Lambda.

The runner became more real as the story continued. A custom boba order could wait at the counter, receive an order number, or sit on a ticket rail. A spill needed retries, an owned recovery shelf, and idempotent handling. The runner needed a kit, a clearance badge, and a safe shift change for a new recipe.

## The Decision Was Never “Serverless or Not?”

The question was never which service sounded most modern.

> **What shape is the work?**

| Work says… | Compute answer |
| --- | --- |
| “I need a machine I control.” | Amazon EC2 |
| “I need a fleet of identical application instances.” | EC2 + Auto Scaling + ELB |
| “I need a packaged application runtime.” | Containers + ECS/EKS/Fargate |
| “I need a finite backlog to finish.” | AWS Batch |
| “I react to an event and finish within a boundary.” | AWS Lambda |

## The Last Bite

You did not just learn a list of compute services. You learned to listen to the work before choosing the kitchen.

The work chooses the compute.

---

**Next section:** *[AWS Storage: The Memory Chooses Its Home](../03-storage/00-the-memory-chooses-its-home.md)*

Compute explains where work runs. The next Services section explains where its files, images, artifacts, and durable object data live. Architecture will become its own later top-level collection, where the series can compare monoliths, modular monoliths, request/response paths, events, orchestration, and choreography across the completed service stories.
