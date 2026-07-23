# Amazon ECR: The Warehouse of Approved Food Truck Blueprints

> Container tools build the food truck blueprint. Amazon ECR stores and distributes it so Amazon ECS can deploy it reliably.

---

## The Pain

Our deployment pipeline is almost complete:

- A container build produces an image.
- A task definition describes how it should run.
- Amazon ECS orchestrates the tasks.
- AWS Fargate can provide the compute capacity.

But one important question remains:

**Where does the container image live?**

Amazon ECS should not depend on an image sitting on a developer's laptop.

---

## The Story

Imagine your company owns five hundred food trucks.

Every time engineering finishes a new design, they leave the blueprint in the factory parking lot.

Restaurant managers drive across town hoping they pick up the correct one.

Some grab yesterday's version.

Some grab today's version.

Someone accidentally grabs an unfinished prototype.

Chaos follows.

The company needs a central warehouse for approved blueprints.

---

## Meet Amazon ECR

Amazon Elastic Container Registry (ECR) is a managed container-image registry.

Unlike general-purpose object storage, ECR organizes container images into repositories and integrates with AWS identity, encryption, scanning, and container services.

The warehouse can provide:

- private or public repositories
- image tags and content digests
- IAM-controlled push and pull access
- encryption for stored image data
- configurable vulnerability scanning
- lifecycle policies for cleaning up images

> **Core idea:** ECR gives container images a managed registry from which authorized systems can push and pull them.

---

## How It Works

```text
Build Container Image
        |
        v
Authenticate to Amazon ECR
        |
        v
Push Image to Repository
        |
        v
Reference Image in Task Definition
        |
        v
Amazon ECS Starts a Task
        |
        v
Runtime Pulls Image from ECR
```

ECR becomes the image source for the deployment. Permissions still determine who can push an image and whether the task execution role can pull it.

---

## The Warehouse Shelves

ECR repositories organize related images.

```text
payments:v1
payments:v2
payments:v3

orders:v8

inventory:v15
```

Each pushed image has a content digest derived from its manifest. The digest identifies specific image content.

If you need different application content, build and push another image rather than changing a running container by hand.

---

## Every Blueprint Gets a Label

Images can have one or more tags:

```text
payments:v1.0
payments:v1.1
payments:v2.0
payments:latest
```

A tag is a human-friendly reference to an image.

Tags make image references easier to read, but their behavior depends on repository settings.

---

## What Does `latest` Mean?

Many beginners assume `latest` always means the newest image.

It does not.

`latest` is an ordinary tag. A build or release process can move it to different image content when tag overwrites are allowed.

For predictable production deployments, teams commonly use explicit version tags or immutable image digests:

```text
payments:v3.2.7

payments@sha256:abc123...
```

An ECR repository can also be configured for tag immutability to prevent existing tags from being overwritten.

---

## Why Retention Matters

Suppose version 13 introduces a bug.

The warehouse may still contain:

- `payments:v11`
- `payments:v12`
- `payments:v13`

Immutable image content and retained version references make rollback possible only while the old image still exists. If version 12 remains available, you can register a task-definition revision that references it and deploy that revision through the ECS service.

Nothing needs to be rebuilt, but the rollback still requires an ECS deployment and enough retained image history.

If an ECR lifecycle policy deletes `payments:v12` and its image, that rollback path is gone. Lifecycle policies should therefore preserve images for as long as the team may need to recover to them.

---

## How Everything Connects

A task definition does not contain the application image data.

It references an image by registry URI and tag or digest.

```text
Amazon ECR Repository
payments:v3 or image digest
        |
        v
Task Definition Revision
        |
        v
Amazon ECS Service
        |
        v
EC2 or Fargate Task
```

When the task starts, the container runtime retrieves the referenced image.

For a private ECR repository, the **ECS task execution role** acts like the truck's warehouse badge. It grants the ECS or Fargate infrastructure permission to authenticate with ECR and pull the image before the application starts. This is different from the task role, which grants permissions to the application running inside the container.

---

## AMIs and Container Images

| EC2 world | Container world |
|---|---|
| AMI | Container image |
| AMI ID | Image digest or tag reference |
| Launch template | ECS task definition |
| EC2 instance | Running ECS task |

Both AMIs and container images support repeatable launches, but they package different units. An AMI defines a machine image for EC2. A container image packages an application filesystem and runs on a compatible container host.

---

## Architectural Mapping

| In the story | In AWS | What it means |
|---|---|---|
| Food truck blueprint | Container image | Packaged application content |
| Warehouse | Amazon ECR | Managed container registry |
| Warehouse section | ECR repository | Collection of related images |
| Shelf label | Image tag | Human-readable image reference |
| Blueprint fingerprint | Image digest | Content-addressed image identity |
| Operating manual | Task definition | Runtime configuration that references the image |

---

## When to Use It

Use Amazon ECR when:

- AWS workloads need a managed container registry
- ECS, EKS, Lambda, or other systems need to pull container images
- image access should integrate with IAM
- vulnerability scanning or lifecycle policies should be applied to repositories
- deployment pipelines need a central image destination

---

## When Not to Use It

Consider another registry or storage service when:

- organizational policy requires an existing external registry
- the artifact is not a supported container image or OCI artifact
- the workload depends on registry features ECR does not provide
- a public distribution workflow belongs in a different established registry

---

## Painkiller

> **Problem:** Containerized deployments need a reliable place to store and retrieve application images.
> **Pain:** Images scattered across laptops and servers create inconsistent, unauditable deployments.
> **AWS solution:** Use Amazon ECR as a managed registry for versioned container-image content and controlled image distribution.

---

## Knife Cut

> **The build creates the blueprint. Amazon ECR stores it. The task definition points to it. Amazon ECS deploys it.**

---

## The Masthead

### What Actually Just Happened

| In the story | In AWS | What it actually means |
|---|---|---|
| Approved blueprint | Container image | Packaged application layers and manifest |
| Warehouse | Amazon ECR | Managed registry API and storage |
| Warehouse section | Repository | Namespace and policy boundary for related images |
| Movable shelf label | Mutable image tag | Friendly reference that can point to different content |
| Blueprint fingerprint | Image digest | Stable reference to specific image content |
| Warehouse badge | ECS task execution role | Lets ECS or Fargate authenticate with private ECR and pull the image |

---

## A Note From the Author

The warehouse metaphor can make ECR sound like an approval system. ECR stores and distributes images, but an image is not automatically safe or approved merely because it was pushed. Teams must configure permissions, scanning, signing or provenance controls, deployment policy, and lifecycle rules appropriate to their supply chain.

Tags are labels rather than permanent versions unless tag immutability is enabled. Digests identify specific image content more precisely. Vulnerability scanning reports findings but does not automatically repair an image, and lifecycle policies can delete older images that a rollback might otherwise need.

- [Amazon ECR repositories](https://docs.aws.amazon.com/AmazonECR/latest/userguide/Repositories.html)
- [Image tag mutability](https://docs.aws.amazon.com/AmazonECR/latest/userguide/image-tag-mutability.html)
- [Amazon ECR image scanning](https://docs.aws.amazon.com/AmazonECR/latest/userguide/image-scanning.html)

---

## The Last Bite

Container images need a dependable home before they become running tasks.

Amazon ECR gives those images an organized registry, readable references, and controlled distribution.

The warehouse stores the blueprint.

The deployment still decides which blueprint is trusted enough to serve customers.

---

**Next chapter:** *Amazon EKS: The National Food Truck Association*

ECR stores the approved food truck blueprint.

EKS asks why some organizations want those trucks to follow the Kubernetes rulebook on AWS.
