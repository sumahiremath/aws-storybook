---
description: "A controlled build starts from a known source revision, runs declared commands in a known environment, reports tests, and emits identified output."
tags:
  - "aws"
  - "deployment"
  - "devops"
  - "codebuild"
---

# AWS CodeBuild: The Central Build Kitchen

> A controlled build starts from a known source revision, runs declared commands in a known environment, reports tests, and emits identified output.

## The Business Goal

Marco and Imani built the same commit.

Their artifact checksums differed.

Marco's laptop used a newer package manager. Imani's cache contained an older transitive dependency. One machine had a native compiler. The other silently downloaded a prebuilt binary.

Both packages called themselves release 4.7.2.

If a production investigation began, the team could not reproduce either artifact confidently.

---

## The Story

Imani moved the build into a central kitchen that started clean for every order.

A checked-in `buildspec.yml` defined:

- Install commands
- Pre-build authentication
- Compilation and tests
- Post-build packaging
- Test reports
- Output artifacts
- Cache paths

CodeBuild launched a managed build environment, retrieved the source revision, assumed a narrowly scoped service role, ran the phases, and uploaded the output.

The build record tied source, commands, logs, reports, environment image, and artifact together.

Developers could still build locally for speed.

Only the central kitchen could stamp a release candidate.

---

## The Wrong Way

A CI build is not reproducible merely because it runs in the cloud.

Reproducibility still requires:

- Pinned dependency versions and lockfiles
- A controlled build image
- Declared environment inputs
- Stable commands
- Captured source revision
- Immutable output identity

Caches improve speed but can hide undeclared dependencies or produce stale behavior. A build must remain correct with an empty cache.

Printing environment variables during debugging can expose secrets. Build logs are durable operational data, not a private terminal.

---

## Meet the AWS Service

**AWS CodeBuild** is a managed build service that runs commands in provisioned build environments.

> **Core idea:** A CodeBuild project defines where source comes from, which environment runs it, which role it assumes, and where artifacts and logs go; the buildspec defines the commands and outputs.

CodeBuild can run independently or as a CodePipeline action.

---

## How It Works

### Build Project

A project configures:

- Source provider
- Build environment image and compute
- Service role
- Environment variables
- Buildspec location
- Artifacts
- Logs
- Cache
- Timeout and networking

Select compute resources appropriate for compilation and tests. Excess resources cost more; insufficient memory can fail or slow builds.

A VPC-enabled build can reach private resources, but network configuration must also provide any required access to package repositories, AWS services, or the internet.

### Buildspec

Buildspec version `0.2` commonly defines phases:

```yaml
version: 0.2
phases:
  install:
    commands:
      - npm ci
  build:
    commands:
      - npm test
      - npm run build
artifacts:
  files:
    - dist/**/*
```

Typical phases are `install`, `pre_build`, `build`, and `post_build`. A `finally` block can run cleanup or diagnostics even when commands fail.

The buildspec can also define:

- Primary and secondary artifacts
- Test report groups
- Cache paths and keys
- Environment variables from Parameter Store or Secrets Manager

Avoid storing plaintext secrets directly in project definitions or buildspec files.

### Service Role

The CodeBuild service role authorizes build actions such as:

- Reading source or input artifacts
- Fetching CodeArtifact dependencies
- Pulling or pushing ECR images
- Writing logs
- Uploading artifacts
- Reading approved parameters or secrets

Do not give the build role production deployment permissions unless the design explicitly combines those responsibilities. Separating build and deploy roles limits blast radius.

### Reports

CodeBuild report groups can collect unit and test framework results. The pipeline can fail when test commands fail and preserve evidence for investigation.

A green build report proves the tests ran and passed. It does not prove the tests were meaningful.

### Artifacts

When CodeBuild runs inside CodePipeline, input and output artifacts flow through the pipeline artifact store. A container workflow may instead push an image to ECR and emit a small definition file for the deploy action.

Record the artifact or image identity. Do not rebuild during the deployment stage.

### Cache

Local or S3-backed caches can speed dependency and build work. Keys should change when relevant lockfiles or compiler inputs change.

Treat cache restoration as an optimization. The build must declare every required dependency independently.

---

## Architectural Mapping

```text
source revision
      |
CodeBuild project
  environment image
  service role
  buildspec.yml
      |
 install -> test -> package
      |       |        |
   cache   reports   artifact / ECR image
```

Logs should avoid secrets and unnecessary customer data. Artifact buckets and ECR repositories need encryption, access control, lifecycle rules, and cross-account policies where promotion spans accounts.

---

## When to Use It

Use CodeBuild when:

- Builds and tests need a managed clean environment
- The team wants checked-in build commands
- Artifacts must be tied to source and logs
- The pipeline needs test reports and controlled roles
- Container images must be built and pushed to ECR

## When Not to Use It

Do not move an opaque local build script into CodeBuild and call it controlled. Declare inputs and pin tooling.

Do not place manual approval or multi-environment release orchestration inside one long build. That is pipeline work.

---

## Painkiller

> **Problem:** Developer machines produce different artifacts from the same source.  
> **Pain:** The release cannot be reproduced, trusted, or investigated.  
> **AWS solution:** Run declared buildspec phases in CodeBuild, use a scoped service role and controlled environment, publish reports, and emit one identified artifact for promotion.

---

## Knife Cut

> **CodeBuild creates and tests the artifact. It does not decide the artifact's journey through environments.**

---

## The Masthead

### What Actually Just Happened

|In the story|In CodeBuild|What it actually means|
|---|---|---|
|Central build kitchen|Build project|Managed build configuration|
|Kitchen instructions|`buildspec.yml`|Commands, phases, reports, artifacts, cache|
|Temporary worker badge|Service role|Permissions used by the build|
|Inspection sheet|Test report|Structured test result evidence|
|Sealed release kit|Output artifact|Build result consumed later|
|Prepared ingredient shelf|Cache|Speed optimization, not a dependency declaration|

---

## A Note From the Author

Build systems are attractive targets because they read source, dependencies, secrets, and signing material and can write deployable output. Apply least privilege, protect webhooks and repository access, review dependency sources, and avoid leaking secrets into artifacts or logs.

The exact build environment is part of the release evidence.

- [CodeBuild buildspec reference](https://docs.aws.amazon.com/codebuild/latest/userguide/build-spec-ref.html)
- [CodeBuild concepts](https://docs.aws.amazon.com/codebuild/latest/userguide/concepts.html)

---

## The Last Bite

The central kitchen produced one approved artifact.

Then a regional manager downloaded it, skipped staging, and installed it directly in production because the checklist arrived in an email.

> **A repeatable build does not create an ordered release process.**

---

**Next chapter:** *[AWS CodePipeline: The Release Conveyor](10-the-release-conveyor.md)*

Imani places the artifact on a release conveyor where stages, actions, approvals, and environment roles decide exactly how far it may travel.

