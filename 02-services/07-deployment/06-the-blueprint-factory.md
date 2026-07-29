---
description: "AWS CDK lets teams compose infrastructure from software constructs, but synthesis must still produce a reviewed CloudFormation deployment contract."
tags:
  - "aws"
  - "deployment"
  - "devops"
  - "cdk"
---

# AWS CDK: The Blueprint Factory

> AWS CDK lets teams compose infrastructure from software constructs, but synthesis must still produce a reviewed CloudFormation deployment contract.

## The Business Goal

The franchise had six application teams.

Each needed a queue, encryption, alarms, logs, least-privilege roles, and a deployment pipeline. Each copied the last team's template and edited it.

Within three months, four versions of “the standard queue” existed. One forgot the dead-letter queue. Another retained logs forever. A third gave the function access to every queue because the copied policy was easier than tracing one ARN.

The franchise had infrastructure as code.

It did not have reusable infrastructure decisions.

---

## The Story

Dev built a blueprint factory.

He created a `FranchiseOrderWorker` construct. A team supplied the handler, timeout, and alarm threshold. The construct created the queue, dead-letter queue, function, permissions, logs, and alarms according to the franchise standard.

Several constructs formed a **stack**. Several stacks formed a **CDK app**.

When Dev ran `cdk synth`, the factory did not deploy programming-language objects to AWS. It produced CloudFormation templates and assets.

When he ran `cdk diff`, the team reviewed the expected infrastructure changes. When the approved app deployed, the CDK toolkit used CloudFormation to operate the stacks.

The factory made good defaults easy.

CloudFormation still made them real.

---

## The Wrong Way

CDK is not permission to hide architecture inside clever abstractions.

A construct that creates twenty resources without exposing meaningful choices can surprise consumers. A construct that exposes every property adds no useful opinion. Good constructs create a stable boundary around a recurring capability.

Programming-language conditionals run during synthesis. They are different from CloudFormation conditions evaluated during deployment. If synthesis depends on local state, time, an unpinned lookup, or environment variables, two developers can produce different templates from the same source.

Generated templates deserve review. “The CDK produced it” is not a security argument.

---

## Meet the AWS Service

**AWS Cloud Development Kit**, or AWS CDK, is a framework for defining cloud infrastructure with supported programming languages.

> **Core idea:** Constructs form a tree inside an app; stacks define deployment units; synthesis converts them into CloudFormation templates.

Construct levels commonly describe:

- **L1 constructs:** direct CloudFormation resource representations
- **L2 constructs:** higher-level resources with ergonomic defaults and helper methods
- **L3 patterns:** opinionated combinations representing an architecture pattern

Every abstraction eventually resolves to resources and properties that CloudFormation can deploy.

---

## How It Works

### App

The app is the root of the construct tree. It contains one or more stacks.

Context and environment information can influence synthesis. Commit stable context where appropriate and avoid accidental dependence on one developer's machine.

### Stack

A CDK stack maps to a CloudFormation stack. Stack boundaries affect deployment order, blast radius, permissions, cross-stack references, and lifecycle.

Use separate stacks when components need independent deployment and ownership. Avoid splitting so aggressively that every change creates tangled cross-stack exports.

### Constructs

Constructs receive a scope, ID, and properties. Their path in the construct tree contributes to generated logical IDs.

Moving resources between constructs or renaming construct IDs can change generated logical IDs and cause CloudFormation to interpret the resource as new. Review diffs for replacement.

### Synthesis

`cdk synth` produces a cloud assembly containing CloudFormation templates and asset metadata.

Assets can include Lambda code or container build inputs. Asset publishing becomes part of deployment. The bootstrap resources in an environment provide storage and roles used by the toolkit.

### Diff and Deploy

`cdk diff` compares synthesized templates with deployed stacks and highlights expected changes. Like a change set, it is evidence, not a proof of runtime success.

`cdk deploy` synthesizes, publishes required assets, and deploys stacks through CloudFormation.

In a controlled pipeline, synthesis and deployment should use pinned dependencies and an identified build environment. Production should not depend on whatever CDK library version happens to be installed globally on a laptop.

### Testing

CDK assertions can inspect the synthesized template:

- Does the queue have encryption?
- Does the function timeout match the standard?
- Is a broad IAM action present?
- Was an alarm created?

Unit assertions test infrastructure intent. They do not prove deployed resources work together. Integration tests still belong after deployment.

---

## Architectural Mapping

```text
CDK app
  ├─ stack
  │   ├─ construct
  │   └─ construct
  └─ stack
       |
    cdk synth
       |
CloudFormation templates + assets
       |
  diff / change review
       |
CloudFormation deployment
```

The pipeline's synthesis and deployment roles should be scoped. Construct libraries are dependencies too; pin, review, and retrieve them through the approved package workflow.

---

## When to Use It

Use CDK when:

- Teams benefit from reusable infrastructure components
- Programming-language composition improves maintainability
- Shared constructs can encode safe organizational defaults
- Template assertions provide useful early tests
- CloudFormation remains an acceptable deployment engine

## When Not to Use It

Do not use CDK to hide an architecture no one understands. Do not create a custom construct library before a stable pattern has emerged. A short CloudFormation or SAM template may be clearer for a small application.

---

## Painkiller

> **Problem:** Teams copy infrastructure templates and gradually lose security, logging, and reliability defaults.  
> **Pain:** Every “standard” application becomes a different architecture.  
> **AWS solution:** Compose CDK apps from reusable constructs, test the synthesized intent, review the diff, and deploy the resulting templates through CloudFormation.

---

## Knife Cut

> **CDK is how you author and compose. CloudFormation is what ultimately deploys and manages the resources.**

---

## The Masthead

### What Actually Just Happened

|In the story|In AWS CDK|What it actually means|
|---|---|---|
|Blueprint factory|CDK app|Root collection of infrastructure stacks|
|Reusable store module|Construct|Component representing resources and configuration|
|One construction project|Stack|CloudFormation deployment unit|
|Print the plans|`cdk synth`|Generate CloudFormation templates and asset metadata|
|Compare renovations|`cdk diff`|Preview synthesized changes|
|Build the stores|`cdk deploy`|Publish assets and deploy stacks|

---

## A Note From the Author

The factory metaphor suggests identical output, but CDK code can make synthesis context-sensitive. Pin dependencies, control context, and synthesize in a known environment.

Higher-level constructs may create IAM roles, security groups, log groups, and public endpoints on the application's behalf. Inspect the synthesized template and know which defaults are acceptable.

- [AWS CDK apps](https://docs.aws.amazon.com/cdk/v2/guide/apps.html)
- [AWS CDK constructs](https://docs.aws.amazon.com/cdk/v2/guide/constructs.html)

---

## The Last Bite

The franchise could now produce the same blueprint repeatedly.

Release 4.7.2 still failed in staging when an external payment provider returned a timeout the unit tests had never modeled.

> **Repeatable infrastructure can reproduce a failure perfectly. Tests must challenge the application before customers do.**

---

**Next chapter:** *[AWS Deployment Testing: The Test Kitchen](07-the-test-kitchen.md)*

The test kitchen begins serving duplicate events, expired tokens, slow payments, and malformed orders on purpose.

