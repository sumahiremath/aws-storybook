# AWS Storybook

AWS Storybook is a narrative learning system for engineers and cloud learners who want to understand, remember, and explain AWS services through stories, mental models, and precise analogy boundaries.

Each article introduces a concrete human problem, opens a story that mirrors the problem structurally, and reveals the AWS service as the natural answer to the situation the story has already established. By the time a service is named, the reader has already identified the need.

---

## How it works

### Stories before services

Articles withhold the service name until the problem is fully visible. The reader arrives at the service, rather than being told about it.

### Mental models over feature lists

The goal of each article is a model the reader can carry into a conversation, a design discussion, or a whiteboard session — and retell accurately.

> A good explanation should make the concept easier to retell.

### Explicit analogy boundaries

Every service article includes **A Note From the Author** — a fourth-wall break that explains exactly where the analogy simplifies or omits real behavior, and where to read further. The analogy teaches the model. The note teaches the limits.

---

## What each article contains

**The Masthead** — a table that maps every story element to its AWS equivalent, with a column explaining what each mapping actually means technically.

**The Knife Cut** — a single line that draws the most important distinction in the article: the boundary between two commonly confused concepts.

**A Note From the Author** — an explicit correction of where the analogy oversimplifies, followed by links to official documentation for readers going deeper.

**The Last Bite** — a closing sentence that reinforces the central lesson without introducing anything new.

---

## Who this is for

- Engineers and cloud learners encountering an AWS service domain for the first time
- People who process new technical concepts through narrative and analogy before reaching for documentation
- Anyone who has read the AWS documentation and still cannot explain the difference between two similar services

AWS Storybook is not a replacement for AWS documentation, hands-on practice, or production architecture guidance. It is a first encounter — a way to build the mental model that makes everything else easier to absorb.

---

## Where to begin

Start with the infrastructure series if you are new to AWS. It establishes the mental models that the service articles build on.

If you already have a foundation, each service section begins with an orientation article that introduces the section's theme before the individual service articles.

```
01-infrastructure/        ← Start here if you are new to AWS
02-services/
  01-identity-security/   ← Complete
  02-compute/             ← Complete
  03-storage/             ← In progress
  06-application-integrations/  ← In progress
```

---

## Sections

### Infrastructure orientation (complete)
Mental models for the cloud itself: what AWS is, how managed services work, the shared responsibility model, event-driven thinking, and how to approach architecture.

### Identity and security (complete)
Cognito, IAM, STS, KMS, Parameter Store, Secrets Manager.

### Compute (complete)
EC2, AMI, Auto Scaling, Elastic Load Balancing, containers, ECS, task definitions, Fargate, ECR, EKS, AWS Batch, Lambda.

### Storage (in progress)
Storage orientation complete. Individual service articles in progress: EBS, EFS, S3, FSx, Glacier, Snow Family.

### Application integrations (in progress)
EventBridge complete. SNS, SQS, Step Functions in progress.

---

## License

Written content in this repository is licensed under Creative Commons Attribution-NonCommercial-ShareAlike 4.0 International.

Commercial reuse is not permitted without permission.

See [LICENSE.md](./LICENSE.md) for details.
