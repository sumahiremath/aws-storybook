---
description: "A release process is complete when the team can identify what is running, prove how it arrived, observe whether it works, and execute a credible recovery plan."
tags:
  - "aws"
  - "deployment"
  - "devops"
  - "epilogue"
---

# AWS Deployment: The Morning After the Rollout

> A release process is complete when the team can identify what is running, prove how it arrived, observe whether it works, and execute a credible recovery plan.

## The Business Goal

The pipeline was green.

The CloudFormation stacks were complete.

CodeDeploy reported success.

Elastic Beanstalk showed healthy instances.

Customers in one Region still waited eight seconds for checkout.

The release machinery had answered:

> “Did every requested deployment action finish?”

Nia needed a different answer:

> “Is the franchise serving customers correctly?”

---

## The Story

At the morning review, Imani rebuilt release 4.7.2's journey on one page.

The repository revision had triggered one pipeline execution. CodeBuild had produced one artifact and test report. The artifact had entered a test stack, passed a deployed API test, and received Leena's approval.

CloudFormation change sets had shown the infrastructure changes. Lambda aliases and ECS task definitions identified the runtime versions. AppConfig showed which menu configuration had reached each environment. CodeDeploy recorded the traffic shift and alarms.

When latency rose, the team could correlate the first slow request with the new version rather than guess which store had copied which file.

They returned traffic, preserved the failing version for investigation, and opened the trace.

Deployment had not prevented the fault.

It had made the fault bounded and explainable.

---

## The Wrong Way

The deployment season can leave a seductive but false conclusion:

> “If we use all these services, releases are safe.”

Safety comes from connected evidence:

- The artifact is identifiable
- Dependencies are declared
- Infrastructure changes are previewed
- Environments are explicit
- Tests challenge real boundaries
- Roles are scoped
- Traffic exposure is controlled
- Health signals represent user outcomes
- Old and new versions can coexist
- Recovery has been designed and practiced

A long pipeline with a mutable artifact and a meaningless health check is ceremony.

---

## Meet the Complete System

The release path now has distinct contracts:

> **Core idea:** Source, build, infrastructure, configuration, orchestration, rollout, and observation are separate responsibilities that must preserve one release identity.

### What Is Being Built?

- CodeArtifact supplies approved package dependencies.
- CodeBuild creates and tests the deployable output.
- Lambda ZIPs, layers, or container images express the function package.
- ECR stores container images by tags and digests.

### Where Will It Run?

- CloudFormation declares AWS resources.
- SAM provides serverless shorthand and developer tooling.
- CDK composes constructs and synthesizes CloudFormation.
- Elastic Beanstalk provides a managed application environment.
- ECS task definitions and services declare container runtime intent.

### How Will It Move?

- CodePipeline orchestrates source, build, test, approval, and deployment actions.
- CodeDeploy performs platform-specific rollouts.
- Service-native deployment controls update Lambda, ECS, API, and environment targets.

### How Much Exposure Is Safe?

- All-at-once favors speed.
- Rolling limits each replacement batch.
- Canary limits first exposure.
- Linear increases exposure over time.
- Blue/green and immutable strategies use fresh capacity for isolation.

### What Can Change Without Rebuilding?

- API stages and custom domains route clients to explicit API environments.
- Lambda aliases point stable names at published versions.
- AppConfig validates and gradually deploys runtime configuration.
- Amplify branches and Copilot environments give frontend and container workflows named environment boundaries.

---

## How It Works

```text
source revision
      |
      v
repeatable build ---- approved dependencies
      |
identified artifact / image digest
      |
test environment + test events + reports
      |
infrastructure change preview
      |
approval and environment role
      |
canary / rolling / blue-green deployment
      |
versioned telemetry + alarms + bake time
      |
complete, stop, or roll back
```

The identity thread matters. If staging tested digest A and production pulled whatever tag `latest` meant later, the thread broke. If production was rebuilt with a different endpoint, the thread broke. If an approver cannot see the artifact and change set, the thread weakened.

### The Security Thread

Every release service has a principal:

- Build role
- Pipeline role
- Cross-account deployment role
- CloudFormation service role
- CodeDeploy service role
- Lambda execution role
- ECS task and task execution roles
- EC2 instance profile

Separate orchestration permissions from application runtime permissions. Protect artifact stores and KMS keys. Retrieve secrets without printing them or baking them into images.

### The Recovery Thread

A credible rollback identifies:

- The known prior application version
- The traffic pointer to move
- The infrastructure state to restore
- The configuration version to restore
- Data and event compatibility
- External side effects that require reconciliation

Sometimes the safest response is roll-forward: deploy a compatible fix because shared state no longer supports the old version.

---

## Architectural Mapping

|Question|Primary mechanism|Failure when omitted|
|---|---|---|
|What source changed?|Repository revision|No traceable input|
|What was built?|Artifact, version, image digest|Staging and production may differ|
|What infrastructure changes?|Template and change set|Surprise replacement or drift|
|What evidence passed?|Test reports and deployed tests|Approval without proof|
|Who allowed promotion?|Pipeline gate and role|Uncontrolled production access|
|Who saw the release first?|Deployment strategy|Unbounded blast radius|
|What declares harm?|Metrics, logs, traces, alarms|Slow or subjective detection|
|How do we return?|Prior versions and recovery plan|Rollback exists only as a button|

---

## When to Use It

Use the full discipline in proportion to impact. A customer-facing payment release deserves more independent evidence and gradual exposure than a disposable development tool.

The principles remain valuable at every size:

- Declare inputs
- Identify output
- Separate environments
- Automate repeated work
- Limit exposure
- Observe outcomes
- Plan recovery

## When Not to Use It

Do not duplicate every enterprise gate in a low-risk prototype. Do not keep a stage because it once seemed mature. Remove controls that add delay without evidence, and strengthen controls that reveal genuine risk.

---

## Painkiller

> **Problem:** Individual deployment services report success while customers still experience a broken release.  
> **Pain:** The team confuses completed infrastructure actions with application health.  
> **AWS solution:** Preserve release identity across build, infrastructure, configuration, pipeline, and rollout; connect version-aware health signals to a tested recovery path.

---

## Knife Cut

> **Deployment success is a control-plane event. Release success is an observed customer outcome.**

---

## The Masthead

### What Actually Just Happened

|Franchise question|AWS evidence|Why it matters|
|---|---|---|
|Which recipe?|Source revision|Identifies input|
|Which sealed kit?|Artifact checksum or image digest|Identifies built output|
|Which store plan?|Template and stack version|Identifies infrastructure|
|Which menu card?|AppConfig version|Identifies runtime configuration|
|Which stores first?|Deployment strategy and execution|Identifies exposure|
|Did customers succeed?|Metrics, logs, traces, business alarms|Identifies actual health|
|Can we return?|Previous versions plus compatibility plan|Makes recovery credible|

---

## A Note From the Author

The franchise rollout is linear on the page. Real releases may deploy multiple services and templates in parallel, each with independent failure and compatibility boundaries.

Keep the release model simple enough to explain during an incident. If the team cannot identify the running version, locate its logs, or state what rollback changes, additional automation has not solved the operational problem.

- [AWS CodePipeline concepts](https://docs.aws.amazon.com/codepipeline/latest/userguide/concepts.html)
- [Introduction to DevOps on AWS](https://docs.aws.amazon.com/whitepapers/latest/introduction-devops-aws/introduction.html)

---

## The Last Bite

Nia did not celebrate because every box had turned green.

She celebrated because the franchise knew what had changed, limited who experienced it first, recognized harm quickly, and returned without guessing.

> **The purpose of deployment is not to move fast. It is to make change routine without making failure mysterious.**

---

**Next chapter:** *[AWS CloudShell, AWS CLI, and Amazon Q Developer: The Franchise Control Desk](15-aws-cloudshell-cli-and-amazon-q-developer-the-franchise-control-desk.md)*

The release is explainable, but Shreya still needs a safe, authorized desk from which to inspect the account and make a narrowly controlled repair.

