---
description: "A pipeline turns release policy into an ordered workflow: which revision enters, which evidence it must produce, who may approve it, and which environment receives it next."
tags:
  - "aws"
  - "deployment"
  - "devops"
  - "codepipeline"
---

# AWS CodePipeline: The Release Conveyor

> A pipeline turns release policy into an ordered workflow: which revision enters, which evidence it must produce, who may approve it, and which environment receives it next.

## The Business Goal

The artifact was reproducible.

Its journey was not.

One regional manager deployed before integration tests completed. Another reran the build for production. A third approved release 4.7.2 but accidentally downloaded 4.7.1 from the shared bucket.

Every individual knew the checklist.

No system enforced its order or preserved the identity of the artifact crossing each step.

---

## The Story

Imani built a conveyor through headquarters.

The entrance accepted one source revision. The build station produced one artifact. The test station consumed that artifact. A production gate displayed the exact revision, reports, and change preview awaiting Leena's approval.

After approval, the deploy action assumed the production role and moved the same artifact into the pilot.

Each station recorded success or failure. A failed action stopped the execution. The artifact did not wander through email, desktops, or renamed folders.

Nia could point at one pipeline execution and answer:

- What triggered it?
- Which source revision entered?
- Which artifacts moved?
- Which tests ran?
- Who approved it?
- Which deployment action followed?

The checklist had become executable.

---

## The Wrong Way

CodePipeline does not compile code, evaluate application health, or deploy every target by itself. It calls action providers that perform those jobs.

A pipeline can automate a dangerous process perfectly. Examples include:

- Rebuilding separately for production
- Approving without visible evidence
- Reusing a mutable image tag
- Sharing one broad IAM role across every stage
- Running tests that never fail
- Deploying infrastructure without reviewing replacement

“Automated” describes execution, not quality.

---

## Meet the AWS Service

**AWS CodePipeline** models a software release workflow as stages containing actions.

> **Core idea:** A pipeline orchestrates actions and passes artifacts; each action provider owns its specific work.

Action categories include:

- Source
- Build
- Test
- Deploy
- Approval
- Invoke

Actions within a stage can run serially or in parallel according to run order. Stages are connected by transitions.

---

## How It Works

### Source

A source action retrieves a revision from a supported repository or source location and emits a source artifact.

Repository events or configured triggers can start the pipeline after a commit, branch update, tag, or other qualifying change. Preserve the source revision with the execution.

Not every commit belongs in production. Branch and tag filters should match the release policy.

### Stages and Actions

A typical path is:

```text
Source -> Build -> Test -> Staging -> Approval -> Production
```

Stages represent logical gates or environments. Actions perform work within them.

A build action might invoke CodeBuild. A deploy action might update CloudFormation, start CodeDeploy, deploy a SAM application, or update another supported target.

Parallel actions are useful for independent tests. Serial order is required when one action consumes another's output.

### Artifacts

Actions declare input and output artifacts. CodePipeline stores and transfers them through an artifact store, commonly an S3 bucket.

An artifact name inside the pipeline is a logical handoff, not the same thing as a CodeArtifact package.

Cross-Region pipelines require appropriate artifact stores. Cross-account actions require carefully configured roles, bucket permissions, encryption key access, and trust relationships.

### Manual Approval

A manual approval action pauses the pipeline until an authorized person approves, rejects, or the approval times out.

The approval should present useful context:

- Source revision and release notes
- Test and security results
- Infrastructure change preview
- Staging endpoint
- Known migration and rollback plan

Approval is a risk decision, not a ceremonial click.

### Transitions and Execution Modes

Disabling an inbound transition prevents executions from entering a stage. This is useful during a freeze or incident, but queued and superseding behavior must be understood so an old change is not released accidentally.

Pipelines also define how overlapping executions behave. Select an execution mode appropriate to whether releases may overlap, queue, or supersede one another.

### Rollback and Retry

A failed action can be retried after its cause is corrected. Some pipeline and stage rollback capabilities can rerun a previous successful stage state using its source revisions and variables.

That is not the same as a compute-platform traffic rollback. The target deployment service and application state still determine what reversal means.

### IAM

The pipeline service role orchestrates actions. Action roles perform target-specific work.

Production should be a distinct trust boundary. A cross-account deployment role can allow the pipeline to deploy approved artifacts without giving developers standing production credentials.

Use least privilege for artifact buckets, KMS keys, source access, build actions, stack operations, and deployment targets.

---

## Architectural Mapping

```text
repository trigger
       |
       v
[Source] -> [Build] -> [Test] -> [Staging] -> [Approve] -> [Deploy]
 artifact   artifact   reports    evidence      human       target
       \_______________________________________________________/
                    one pipeline execution
```

Monitor pipeline and action failures with EventBridge and CloudWatch. Preserve logs in the services that perform the work, such as CodeBuild, CloudFormation, and CodeDeploy.

---

## When to Use It

Use CodePipeline when:

- Releases have repeated ordered stages
- One artifact must move through several environments
- Builds, tests, approvals, and deployments use different providers
- Cross-account production promotion needs controlled roles
- Teams need execution history and automated triggers

## When Not to Use It

Do not create a pipeline stage for every tiny command if a single cohesive build action owns them. Do not place long-running business workflows in a release pipeline; use an application orchestration service for application work.

---

## Painkiller

> **Problem:** People perform a correct release checklist in inconsistent order and sometimes choose the wrong artifact.  
> **Pain:** Tests, approvals, and deployments lose their relationship to one source revision.  
> **AWS solution:** Use CodePipeline stages and actions to orchestrate providers, pass identified artifacts, enforce gates, and assume scoped environment roles.

---

## Knife Cut

> **CodePipeline coordinates the journey. CodeBuild creates the package. CodeDeploy controls supported rollouts.**

---

## The Masthead

### What Actually Just Happened

|In the story|In CodePipeline|What it actually means|
|---|---|---|
|Release conveyor|Pipeline|Automated release workflow|
|Station|Stage|Logical gate or environment|
|Worker at a station|Action provider|Service performing build, test, approval, or deploy work|
|Sealed box handoff|Input/output artifact|Files passed between actions|
|Manager's gate|Manual approval|Authorized pause before continuation|
|Store access badge|Action or cross-account role|Scoped permissions for target work|

---

## A Note From the Author

Pipeline history is valuable only when downstream services preserve useful details. Correlate the pipeline execution with the CodeBuild run, CloudFormation stack events, CodeDeploy deployment ID, application version, and image digest.

The repository is the source entrance, not necessarily an AWS-branded repository. Current release design should focus on revision identity, triggers, access, and branching policy rather than assuming one source provider.

- [CodePipeline concepts](https://docs.aws.amazon.com/codepipeline/latest/userguide/concepts.html)
- [CodePipeline pipeline structure](https://docs.aws.amazon.com/codepipeline/latest/userguide/reference-pipeline-structure.html)

---

## The Last Bite

The conveyor delivered release 4.7.2 to the production gate in perfect order.

The last action still needed to answer:

> Which machines or versions change, what happens before traffic arrives, and what signal sends us back?

> **Orchestration gets the artifact to the rollout. It does not define the rollout's mechanics.**

---

**Next chapter:** *[AWS CodeDeploy: The Rollout Manager](11-the-rollout-manager.md)*

The rollout manager opens the AppSpec file and coordinates lifecycle hooks, deployment groups, alarms, and rollback.

