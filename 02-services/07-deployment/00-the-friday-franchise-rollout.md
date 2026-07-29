---
description: "Writing a change proves that it can exist. Deployment proves that the same known change can reach real users safely."
tags:
  - "aws"
  - "deployment"
  - "devops"
  - "orientation"
---

# AWS Deployment: The Friday Franchise Rollout

> Writing a change proves that it can exist. Deployment proves that the same known change can reach real users safely.

## The Business Goal

Byte Burger had grown into two hundred locations.

Headquarters had a new point-of-sale release ready for Friday: faster mobile checkout, a redesigned kitchen screen, and a limited-time menu.

At 8:00 a.m., Marco emailed the new code to every regional manager.

By lunch, twelve stores had installed it. Four had copied an old configuration file. One lacked a library Marco had installed on his laptop. Three restarted every register at once. A store in Phoenix discovered that the new kitchen screen expected an API endpoint that existed only in the test environment.

No two stores had received quite the same release.

The code was finished.

The deployment had never truly begun.

---

## The Story

Imani, the release engineer, placed a sealed box on Nia's desk.

“This is release 4.7.2,” she said. “Not an email. Not a folder someone can edit. One built, tested, numbered kit.”

The kit would first enter a test kitchen. Automated orders would exercise it. A staging Byte Burger location would run the exact artifact that production would receive. Leena would approve the promotion, but approval would not rebuild anything.

Then one pilot store would receive a small share of real traffic.

If its checkout errors rose, an alarm would stop the rollout and send traffic back to the known version. If it stayed healthy, the same artifact would move through store batches until the fleet converged.

Nia looked at the rollout board:

```text
source -> build -> test -> approve -> pilot -> batches -> complete
                 same numbered artifact -------------------->
```

For the first time, deployment looked less like copying files and more like controlling change.

---

## The Wrong Way

Teams often call several different moments “deployment”:

- A developer merges source code
- A build creates a package
- An artifact is installed in an environment
- Traffic begins reaching the new version
- A feature becomes visible to customers

Those moments can be connected, but they are not identical.

A **build** converts source and dependencies into a deployable artifact. A **deployment** places a version into an environment. A **release** exposes that version or feature to users. A configuration or feature flag can separate deployment from release.

Rebuilding separately for staging and production also weakens the evidence. Even if both builds start from the same commit, dependency resolution or tool changes can produce different output. The safer promotion model moves one identified artifact forward.

---

## Meet the AWS Services

AWS provides services for different parts of the path.

> **Core idea:** Packaging defines what moves, infrastructure as code defines where it runs, CI/CD defines how it moves, and a deployment strategy defines how much user traffic is exposed.

- **AWS CodeArtifact** stores versioned software packages used as dependencies.
- **AWS CodeBuild** runs build and test commands in managed build environments.
- **AWS CodePipeline** coordinates source, build, test, approval, and deploy actions.
- **AWS CodeDeploy** manages rollouts to supported compute platforms.
- **AWS CloudFormation**, **AWS SAM**, and **AWS CDK** define and deploy infrastructure.
- **AWS AppConfig** validates and gradually deploys runtime configuration.
- **AWS Elastic Beanstalk** provides a managed application platform and deployment policies.
- **Amazon ECR** stores container images; **Amazon ECS** deploys container task revisions.

No single service turns unsafe code into a safe release. The application team still owns tests, permissions, health signals, data compatibility, and rollback design.

---

## How It Works

### Identify the Change

A source revision should be traceable to the build it produced. Branches and labels help organize work, but an approved release needs a durable identity: a version, artifact checksum, or container digest.

### Build Once

The build installs declared dependencies, compiles if necessary, runs early tests, and creates output with the directory structure the runtime expects.

The output might be:

- A Lambda ZIP archive
- A Lambda container image
- A web application bundle
- An Elastic Beanstalk source bundle
- A container image and ECS task definition revision
- A CloudFormation or SAM template plus referenced artifacts

### Test at Several Distances

Unit tests examine small pieces. Integration tests exercise dependencies. Deployed tests call development endpoints. Event-driven tests include duplicates, retries, malformed payloads, and partial failure.

Local tests are fast, but managed environments provide evidence that local emulation cannot.

### Promote Through Environments

Development, test, staging, and production should be explicit environments, not accidental variations on one shared account or endpoint. Environment-specific values are supplied at deployment or runtime while the approved application artifact remains unchanged.

### Control Exposure

An all-at-once deployment is fast but broad. Rolling deployments change batches. Canary and linear deployments expose traffic gradually. Blue/green maintains old and new environments so routing can move between them.

### Observe and Reverse

Deployment health must include more than “the command succeeded.” Error rate, latency, throttles, failed health checks, queue backlog, and business signals can reveal a bad release.

Rollback returns traffic or compute to a previous known version. It does not unsend notifications, reverse orders, or automatically undo an incompatible database migration.

---

## Architectural Mapping

```text
Repository
    |
    v
CodePipeline -> CodeBuild -> test environment -> approval
                                      |
                                      v
                              deploy known artifact
                                      |
                       canary / rolling / blue-green
                                      |
                              alarms and rollback
```

The pipeline needs an IAM role and each action needs only the permissions required for its job. Artifact stores and container repositories need access controls and encryption. Secrets should come from managed secret stores, not source code or casually printed build variables.

---

## When to Use It

Automate the release path when:

- The same change moves through more than one environment
- Repeated manual steps produce drift or mistakes
- Production needs approvals, tests, or auditability
- Rollout risk requires traffic shifting and alarms
- Several developers must produce reproducible artifacts
- Infrastructure and application changes must move together

## When Not to Use It

A tiny prototype may not need a sophisticated multi-stage production pipeline. It still benefits from declared dependencies, repeatable packaging, and a way to identify what is running.

Automation should match risk. More stages do not help when each stage repeats a meaningless test.

---

## Painkiller

> **Problem:** Headquarters distributes source and instructions instead of one known release.  
> **Pain:** Stores build different output, apply steps out of order, and discover incompatibilities in production.  
> **AWS solution:** Build an identified artifact, promote it through controlled environments, deploy it with an appropriate traffic strategy, and connect health alarms to rollback.

---

## Knife Cut

> **A commit identifies source. An artifact identifies what was built. A deployment identifies where it runs. A release identifies who receives it.**

---

## The Masthead

### What Actually Just Happened

|In the story|In AWS|What it actually means|
|---|---|---|
|Recipe source|Repository revision|Traceable input to the build|
|Sealed release kit|Build artifact or image digest|Known output promoted between environments|
|Test kitchen|Development or test environment|Place to exercise deployed behavior|
|Pilot store|Canary target|Limited first exposure|
|Store batches|Rolling or linear deployment|Risk spread across increments|
|Health bell|CloudWatch alarm|Signal that can stop or reverse a rollout|
|Recall|Rollback or redeployment|Return to a previous known deployed state|

---

## A Note From the Author

Byte Burger's franchise explains coordination and blast radius. Real deployments cross service-specific contracts. A Lambda alias shifts invocations between published function versions; an ECS service replaces tasks from a new task definition; an EC2 deployment may update instances in place; AppConfig distributes configuration rather than application binaries.

Treat “rollback” carefully. Application code may be reversible while database schemas, messages, payments, and external side effects are not. Safe releases often require backward-compatible changes and explicit recovery procedures.

- [Introduction to DevOps on AWS](https://docs.aws.amazon.com/whitepapers/latest/introduction-devops-aws/introduction.html)
- [AWS CodePipeline concepts](https://docs.aws.amazon.com/codepipeline/latest/userguide/concepts.html)

---

## The Last Bite

At 5:00 p.m., Nia did not ask, “Did everyone receive the files?”

She asked, “Which version is serving customers, what evidence says it is healthy, and how quickly can we return?”

> **Deployment is not movement alone. It is controlled, observable, reversible change.**

---

**Next chapter:** *[AWS Deployment Packages: The Sealed Release Kit](01-the-sealed-release-kit.md)*

Before release 4.7.2 can enter a test kitchen, Imani must answer a deceptively simple question:

> What, exactly, is inside the box?
