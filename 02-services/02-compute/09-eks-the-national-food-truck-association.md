# Amazon EKS: The National Food Truck Association

> Amazon ECS provides AWS-native container orchestration. Amazon EKS provides managed Kubernetes for teams that use the Kubernetes API and ecosystem.

---

## The Pain

Our food truck business is thriving on AWS.

Amazon ECS coordinates our trucks.

AWS Fargate can provide their campsites.

Life is good.

Then the business expands.

Some teams already operate trucks under Kubernetes rules. Others use Kubernetes tooling in several environments or have built company-wide practices around its APIs.

That creates the central question:

**Why would a team choose Kubernetes on AWS instead of Amazon ECS?**

---

## The Story

Imagine every city invents its own food truck rules.

One city requires blue permits.

Another requires green permits.

Another uses completely different inspection forms.

Moving operating knowledge between cities becomes painful.

The company wants a shared rulebook, vocabulary, and set of tools that many locations recognize.

---

## Meet Kubernetes

A national food truck association publishes a common handbook.

Participating cities use concepts such as:

- deployments
- pods
- services
- configuration and secrets
- health checks
- scheduling rules

Food truck teams can reuse those concepts and much of their tooling across Kubernetes environments.

That association is **Kubernetes**.

It is an open-source platform for orchestrating containerized workloads.

---

## Meet Amazon EKS

Amazon Elastic Kubernetes Service (EKS) is a managed Kubernetes service on AWS.

AWS operates the Kubernetes control plane, including its availability and integration with AWS infrastructure. Your team uses Kubernetes APIs and tools to declare and operate workloads.

Worker compute is a separate decision. Depending on the cluster design, workloads can run on managed node groups, self-managed EC2 nodes, AWS Fargate, or other supported EKS compute options.

> **Core idea:** EKS gives Kubernetes teams an AWS-managed control plane while preserving the Kubernetes API and ecosystem.

---

## From ECS Tasks to Kubernetes Pods

In Amazon ECS, a **task** is the running unit scheduled by ECS from a task definition.

In Kubernetes, a **Pod** is the smallest schedulable unit.

A Pod may contain one container or several tightly coupled containers that need to share parts of their runtime environment.

An ECS task can also contain multiple containers, so tasks and Pods serve a similar purpose: they group containers that should run together as one scheduled unit.

They are not identical objects. Each belongs to a different orchestration model with its own API, lifecycle, networking, and configuration semantics.

In the food truck story, both are operating trucks. They simply follow different rulebooks.

---

## How It Works

```text
Kubernetes Manifests and Tools
kubectl, Helm, GitOps controllers
              |
              v
     Amazon EKS Control Plane
Kubernetes API and cluster coordination
              |
              v
       Kubernetes Scheduler
              |
              v
Worker Capacity
EC2 nodes, managed node groups, or Fargate
              |
              v
            Pods
```

The control plane accepts desired-state declarations through the Kubernetes API. Kubernetes controllers and the scheduler work to place and maintain pods on compatible worker capacity.

---

## ECS vs. EKS

Both services orchestrate containers, but they expose different operating models.

| Amazon ECS | Amazon EKS |
|---|---|
| AWS-native orchestration model | Kubernetes orchestration model |
| ECS APIs and task definitions | Kubernetes APIs and workload resources |
| Lower Kubernetes-specific overhead | Access to the Kubernetes ecosystem |
| Strong fit for AWS-focused teams | Strong fit for teams standardized on Kubernetes |
| AWS-specific operational model | More portable concepts with environment-specific integration work |

Neither is universally better.

ECS offers a simpler AWS-native path for teams that do not need Kubernetes. EKS provides Kubernetes APIs, ecosystem compatibility, and alignment with organizations that have standardized on Kubernetes.

Choose the model that fits the team's skills, platform standards, ecosystem requirements, and operational tolerance.

---

## Why Teams Choose EKS

Organizations may already operate Kubernetes across:

- AWS
- other cloud providers
- private data centers
- developer or edge environments

Using Kubernetes can let them reuse APIs, packaging tools, deployment patterns, and operational knowledge.

Amazon EKS lets those teams keep that Kubernetes model while integrating workloads with AWS networking, identity, storage, load balancing, observability, and compute services.

---

## Architectural Mapping

| In the story | In AWS | What it means |
|---|---|---|
| National association | Kubernetes project and ecosystem | Shared orchestration APIs and concepts |
| Association handbook | Kubernetes API resources | Desired-state workload definitions |
| AWS-hosted headquarters | Amazon EKS control plane | AWS-managed Kubernetes control plane |
| Local festival grounds | Worker compute | Capacity where pods run |
| Food trucks | Pods and containers | Scheduled application workloads |
| Local permits and utilities | AWS integrations | Networking, IAM, storage, and load balancing configuration |

---

## When to Use It

Use Amazon EKS when:

- the organization has standardized on Kubernetes
- workloads depend on Kubernetes APIs, controllers, or ecosystem tools
- platform teams already have Kubernetes operational expertise
- a managed Kubernetes control plane is preferred on AWS
- common Kubernetes practices must span several environments

---

## When Not to Use It

Consider Amazon ECS or another compute service when:

- the team does not need Kubernetes APIs or tooling
- minimizing platform complexity is more important than ecosystem flexibility
- the workload can be served by a simpler container or serverless model
- the organization lacks the capacity to operate Kubernetes workloads, add-ons, policies, and upgrades

---

## Painkiller

> **Problem:** Teams standardized on Kubernetes need to run that orchestration model on AWS.
> **Pain:** Building and maintaining the Kubernetes control plane adds availability, upgrade, security, and integration work.
> **AWS solution:** Use Amazon EKS for an AWS-managed Kubernetes control plane integrated with supported AWS compute and infrastructure options.

---

## Knife Cut

> **Amazon ECS is AWS-native container orchestration. Amazon EKS is managed Kubernetes on AWS.**

---

## The Masthead

### What Actually Just Happened

| In the story | In AWS | What it actually means |
|---|---|---|
| National rulebook | Kubernetes API | Common desired-state resource model |
| AWS association office | Amazon EKS | Managed Kubernetes service |
| Headquarters staff | EKS control plane | AWS-operated Kubernetes API and control-plane components |
| Festival grounds | Worker capacity | EC2- or Fargate-based capacity for pods |
| Food truck schedule | Kubernetes deployment | Declares and maintains application replicas |
| Operating truck | Pod | Smallest Kubernetes scheduling unit |

---

## A Note From the Author

The national-association metaphor makes Kubernetes sound like a universal permit that removes every local difference. Kubernetes standardizes important APIs and concepts, but moving a workload between environments can still require changes to identity, networking, storage classes, load balancers, ingress, observability, policies, and managed-service integrations.

EKS also does not eliminate Kubernetes operational complexity. AWS manages the control plane, while customers remain responsible for Kubernetes version planning, workload upgrades, add-ons, networking, security policies, access controls, application security, and—depending on the selected compute model—worker nodes and their lifecycle.

- [Amazon EKS concepts](https://docs.aws.amazon.com/eks/latest/userguide/what-is-eks.html)
- [Amazon EKS compute options](https://docs.aws.amazon.com/eks/latest/userguide/eks-compute.html)

---

## The Last Bite

Amazon ECS and Amazon EKS can both coordinate containers on AWS.

ECS gives teams an AWS-native operating model.

EKS gives Kubernetes teams a managed control plane and their familiar ecosystem.

The best rulebook is the one the organization can operate safely and consistently.

---

**Next chapter:** *AWS Batch: The Catering Company*

ECS and EKS orchestrate containerized applications that need to remain available.

AWS Batch asks a different question: what if the work is finite and the goal is to drain a backlog?
