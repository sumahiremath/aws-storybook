# Containers: The Food Truck That Carries the Kitchen

> Containers package an application and everything it needs to run, allowing the same workload to run consistently on any compatible host.

---

## The Pain

Your first boba shop is thriving.

Then a music festival invites you.

Opening another permanent shop makes no sense.

You only need to sell drinks for the weekend.
You try moving the shop.

The counters are bolted down.
The refrigerators are wired into the building.
The plumbing stays behind.

The building was never meant to travel.
The business needs a kitchen that moves.

---

## The Story

Instead of moving the building, the owner builds a food truck.

The truck already contains:

- the tea equipment
- the blenders
- the cups
- the syrups
- the menu
- the order counter

Drive it to a festival.
Drive it to the beach.
Drive it to a football game.

The kitchen works consistently anywhere a compatible host is waiting.

The parking lot changes.

The kitchen does not.

That is a container.

---

## Meet Containers

A container packages:

- the application
- required libraries
- runtime
- dependencies
- configuration needed by the application

Containers share the host operating system kernel instead of bringing an entire operating system like a virtual machine.

> **Core idea:** A container packages the kitchen, not the building.

---

## The Boba Mapping

| Boba Company | AWS / Containers | Meaning |
|---|---|---|
| Food truck | Container | Portable application package |
| Kitchen equipment | Application + dependencies | Everything needed to run |
| Parking lot | Host machine | Where the container runs |
| Fleet of trucks | Container service | Many containers running together |
| Recipe card | Container image | Immutable template |
| Running truck | Container | Executing image |

---

## Why Not Another Shop?

An EC2 instance gives the business an entire shop.

A container carries only the application kitchen.

Many containers can run on the same host because they share the host operating system kernel.

That smaller launch unit makes containers lighter and faster to start than full virtual machines.

---

## Container Image

Before building food trucks, the owner designs a standard truck.

Every future truck is built from that design.

That design is the **container image**.

```text
Container Image
       |
       | start
       v
Running Container
```

The image is read-only.

Running containers are created from it.

Most developers build and package container images with tools such as Docker. AWS does not require Docker specifically, but Docker popularized the image format and workflow supported by many AWS container services.

---

## Container Image vs. AMI

Yesterday, the boba company standardized its permanent shops.

Every new location started from the same approved machine image.

That was an **AMI**.

Today, the company wants the kitchen itself to travel.

That is a different unit.

### AMI: Rebuild the Whole Shop

An AMI describes how to launch an EC2 instance.

In the story, it is the blueprint for an entire shop.

It can include:

- the operating system
- installed software
- the application runtime
- monitoring agents
- startup services
- application code

The important point is not how much application work has already been done.

The important point is what gets launched.

An AMI launches a machine.

The whole shop opens.

### Container Image: Carry the Kitchen

A container image packages the application and the files it needs to run.

In the story, it is the portable kitchen inside the food truck.

It typically contains:

- the application
- required libraries
- the runtime
- the framework
- supporting binaries
- the startup command

But the kitchen does not create its own parking lot.

The running container still depends on a compatible host for:

- the operating system kernel
- compute
- networking
- permissions
- configured storage

The image carries the kitchen.

The host provides the ground beneath it.

### Side by Side

| AMI | Container image |
|---|---|
| Blueprint for the whole shop | Blueprint for the portable kitchen |
| Launches an EC2 virtual machine | Starts an application process as a container |
| Includes a full operating system | Shares the host operating system kernel |
| Usually larger | Usually smaller |
| Usually slower to launch | Usually faster to start |

> **An AMI packages the machine. A container image packages the application environment that runs on a compatible host.**

## How It Works

```text
Application Code
        |
Dependencies
        |
Runtime
        |
Configuration
        |
        v
 Container Image
        |
        v
 Running Container
        |
        v
 Host Machine
```

---

## Containers vs EC2

| EC2 Shop | Container Food Truck |
|---|---|
| Full building | Portable kitchen |
| Includes operating system | Shares host kernel |
| Minutes to prepare | Seconds to start |
| Larger footprint | Lightweight |
| Good for long-running servers | Great for portable workloads |

Neither replaces the other.

Containers often run **on EC2**.

EC2 provides the parking lot.

Containers provide the kitchen.

---

## AWS Container Services

AWS provides several ways to run containers.

### Amazon ECS

AWS helps manage fleets of containers.

### Amazon EKS

Runs Kubernetes for teams already using Kubernetes.

### AWS Fargate

You provide the food truck.

AWS provides the parking lot automatically.

No EC2 fleet to manage.

---

## Wrong Way

A blender breaks inside one food truck.

Instead of fixing the truck design, an engineer climbs inside.

```text
Install package.

Edit configuration.

Download one missing library.

"It works now!"
```

The truck finishes the festival.

The next truck arrives.

The fix is gone.

Containers are disposable.

Update the image, then replace the running container instead of treating that container like a permanent server.

### Do Not Leave the Cash Register in the Truck

Every night, the owner parks the truck.

If someone steals it, today's cash disappears.

Business data should not live only inside the truck's writable layer.

Persistent data belongs in durable storage or a managed data service designed for it.

The truck should carry the kitchen.

Not the business.

> **Move the kitchen. Do not move the cash register.**

---

## Architectural Mapping

```text
Food Truck
      |
      v
Container
      |
      v
Container Service
(ECS / EKS / Fargate)
      |
      v
EC2 or AWS-managed infrastructure
```

---

## When to Use Containers

Use containers when:

- applications should run consistently everywhere
- deployments should be fast
- many isolated workloads share infrastructure
- microservices are common
- scaling individual services is important

---

## When Not to Use Them

Containers are not automatically the best answer.

Simple applications may fit Lambda.

Traditional long-lived servers may fit EC2.

Choose the tool that matches the workload.

---

## Painkiller

> **Problem:** Applications behave differently across environments and are difficult to move.
> **Pain:** Installing runtimes and dependencies repeatedly causes inconsistent deployments.
> **AWS solution:** Package the application and its dependencies into a portable container that runs consistently on compatible hosts.

---

## Knife Cut

> **An AMI launches the machine. A container image launches the application on a compatible host.**

---

## The Masthead

### What Actually Just Happened

| Story | Technology | Meaning |
|---|---|---|
| Food truck | Container | Portable application |
| Standard truck design | Container image | Immutable template |
| Parking lot | Host | Where containers run |
| Fleet of trucks | ECS / EKS / Fargate | Container orchestration |

---

## A Note From the Author

The food truck is a memory aid, not a complete container runtime model.

An AMI can include a fully deployable application. A container image can also be large and complex. The defining distinction is the launch unit: an AMI launches a virtual machine with its own operating system, while a container image starts an application process that shares the host kernel.

A running container still depends on a compatible host, container runtime, networking, permissions, and configured storage.

Containers also receive a writable layer while running. That layer is generally disposable. Durable business data belongs in persistent storage or a managed data service, not only inside the container.

- [AWS container services documentation](https://docs.aws.amazon.com/containers/)

---

## The Last Bite

The owner stopped moving buildings.

Instead, she built kitchens that could travel anywhere.

Containers do the same thing for software.

They package the application once and let it run consistently wherever a compatible host is waiting.

---

**Next chapter:** *Amazon ECS: The Operations Manager Behind Your Food Truck Festival*

One food truck is useful.

Running hundreds of them requires an organizer.
