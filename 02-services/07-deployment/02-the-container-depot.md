---
description: "A container image packages an application. A registry identifies and stores it. An orchestrator decides how copies of it should run."
tags:
  - "aws"
  - "deployment"
  - "devops"
  - "ecs"
  - "ecr"
---

# Amazon ECR and Amazon ECS: The Container Depot

> A container image packages an application. A registry identifies and stores it. An orchestrator decides how copies of it should run.

## The Business Goal

The pilot store passed every checkout test.

The second store failed immediately.

Both deployment records said they had received `release-4.7.2`.

Imani opened the container registry history. Someone had fixed a typo, rebuilt the image, and pushed the new bytes under the same tag.

The two stores wore the same label and ran different software.

No dashboard could explain the fleet until the team could answer which image each store had actually pulled.

---

## The Story

At the franchise depot, Nia watched workers attach labels to sealed kitchen modules.

The label `candidate` moved every morning. `release-4.7.2` was supposed to stay still, but policy had not enforced that promise.

Imani showed her two identifiers:

```text
tag:    release-4.7.2
digest: sha256:9f...c2
```

A tag was a convenient name. A digest was derived from image content.

The deployment record was changed to capture the digest. A new ECS task definition revision referenced the approved image and declared its CPU, memory, ports, environment, secrets, and logs.

The ECS service then replaced old tasks while maintaining its desired number of healthy copies.

The depot stored the image.

The service ran the fleet.

---

## The Wrong Way

“We use containers” does not describe a deployment architecture.

A container workflow still needs:

- A repeatable image build
- A registry
- Identity and vulnerability controls
- Runtime resource and network configuration
- A controller that starts replacements and maintains capacity
- Health checks, logs, alarms, and rollback

Using a mutable tag such as `latest` as the only release identity makes later diagnosis ambiguous. Even a semantic version tag is only immutable if the repository policy and team behavior keep it that way.

Putting credentials into image layers is worse: deleting the visible file in a later layer may not remove the secret from image history.

---

## Meet the AWS Services

**Amazon ECR** is a managed container image registry. **Amazon ECS** is a container orchestration service.

> **Core idea:** ECR answers “Which image can be pulled?” ECS answers “Which task definition should be running, and how many copies should remain healthy?”

An ECS **task definition** is a versioned JSON blueprint for one or more containers. It can specify:

- Image references
- CPU and memory
- Ports and commands
- Environment variables and secrets
- Logging
- Network mode
- Runtime platform
- Task and execution roles

An ECS **service** maintains a desired task count and can deploy a new task definition revision.

---

## How It Works

### Build and Push

CodeBuild or another build system creates the image from a Dockerfile or compatible build definition. The build authenticates to ECR and pushes the image.

The build role needs permission to obtain an authorization token and perform only the necessary repository operations. The ECS task execution role later needs permission to pull private images and publish logs when configured.

### Identify the Image

Tags make images easy to discuss. Digests make content precise.

Safer release records preserve:

- Source revision
- Image digest
- Build execution
- Test result
- Task definition revision

ECR can enforce tag immutability, scan images, apply lifecycle policies, and replicate repositories according to platform needs. Scanning is evidence, not a guarantee that the application is secure.

### Register a Task Definition Revision

Changing an image, resource allocation, environment value, or runtime option generally leads to a new task definition revision.

CPU and memory are not decorative metadata. A task that requests an unsupported combination or exceeds available capacity may remain pending or fail. A task with too little memory can be terminated under load.

### Update the ECS Service

The service scheduler starts tasks from the new revision and replaces unhealthy or stopped tasks. Deployment configuration controls minimum and maximum healthy capacity during a rolling deployment.

With load balancing, health checks help determine when new tasks can receive traffic and when old tasks can stop.

Rollback still requires a known earlier task definition and compatible external state.

### ECS, EKS, and Copilot

ECS provides AWS-native container orchestration. **Amazon EKS** provides a managed Kubernetes control plane for teams that need Kubernetes APIs and its ecosystem. The application team still manages Kubernetes workloads and many cluster-level choices.

**AWS Copilot** provides a developer-oriented workflow for defining and deploying container applications into named environments, commonly using ECS and related AWS resources. It can simplify repeated environment setup; it does not remove the underlying service contracts.

---

## Architectural Mapping

```text
source revision
      |
      v
 image build -> ECR repository
                   |
             tag + digest
                   |
       ECS task definition revision
                   |
             ECS service
              /   |   \
           task task task
```

Application code normally uses a **task role** for AWS API access. The ECS agent and platform use a **task execution role** for operations such as pulling an ECR image or sending logs. Confusing those roles can either break startup or grant the application unnecessary permissions.

---

## When to Use It

Use ECR and ECS when:

- The application is packaged as containers
- A service must maintain long-running copies
- CPU, memory, networking, and startup behavior need explicit control
- Multiple services need independent image and task revisions
- The team prefers AWS-native orchestration without operating Kubernetes

Consider EKS when Kubernetes compatibility and ecosystem requirements are genuine constraints.

## When Not to Use It

Do not containerize a small event handler solely for fashion when Lambda ZIP packaging is simpler. Do not choose EKS because it is the most flexible option if the team does not need or want Kubernetes operations.

Do not assume ECR deploys anything. A registry stores images; another service consumes them.

---

## Painkiller

> **Problem:** A reused image tag points different stores to different content.  
> **Pain:** Rollout evidence becomes ambiguous and rollback cannot reliably reproduce the old state.  
> **AWS solution:** Store images in ECR, preserve immutable digests, register explicit ECS task definition revisions, and let ECS services replace tasks under health and capacity controls.

---

## Knife Cut

> **A tag is a label. A digest is content identity. A task definition is runtime intent. A service keeps that intent running.**

---

## The Masthead

### What Actually Just Happened

|In the story|In AWS|What it actually means|
|---|---|---|
|Container depot|ECR|Registry for container images|
|Movable label|Image tag|Human-friendly reference that can be mutable|
|Tamper-evident seal|Image digest|Content-addressed image identity|
|Kitchen module plan|ECS task definition revision|Container image and runtime configuration|
|Fleet manager|ECS service|Maintains and deploys desired tasks|
|Store class|Copilot environment|Named environment in a container developer workflow|

---

## A Note From the Author

The depot analogy separates storage from orchestration. In real systems, an ECS deployment also depends on subnets, security groups, load balancer target groups, task and execution roles, service discovery, capacity providers, and health checks.

Image immutability does not mean the deployed application is stateless or reversible. It only makes the software input identifiable.

- [Amazon ECS task definitions](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/task_definitions.html)
- [Amazon ECR image tag mutability](https://docs.aws.amazon.com/AmazonECR/latest/userguide/image-tag-mutability.html)

---

## The Last Bite

Release 4.7.2 finally meant one set of bytes.

But the store receiving it still contained hand-installed queues, roles, tables, and API settings that differed from its neighbors.

> **An identical application cannot behave identically on infrastructure that was assembled from memory.**

---

**Next chapter:** *[AWS CloudFormation: The Franchise Blueprint](03-the-franchise-blueprint.md)*

Dev rolls out a blueprint so every store begins from the same declared design.

