---
description: "A deployment is more than placing files. It is a compute-specific sequence for installing, validating, exposing, and, when necessary, replacing a version."
tags:
  - "aws"
  - "deployment"
  - "devops"
  - "codedeploy"
---

# AWS CodeDeploy: The Rollout Manager

> A deployment is more than placing files. It is a compute-specific sequence for installing, validating, exposing, and, when necessary, replacing a version.

## The Business Goal

The production action copied release 4.7.2 onto an EC2 instance and reported success.

The application process never restarted.

At another store, the process restarted before the new configuration arrived. At a third, traffic returned before the health endpoint became ready.

The file copy had succeeded.

The application deployment had failed.

---

## The Story

Imani handed the rollout manager two documents.

The first named the target group of stores.

The second, the AppSpec file, described the release and lifecycle instructions:

- Where files belonged
- Which scripts ran before and after installation
- When the application stopped and started
- How to validate service health

For Lambda, the plan named the new function version and traffic hooks. For ECS, it named the task definition, container, port, and validation hooks for the replacement task set.

CloudWatch alarms watched the rollout. If checkout errors entered `ALARM`, CodeDeploy stopped and initiated the configured rollback.

The manager was not just a courier.

It controlled the changeover.

---

## The Wrong Way

CodeDeploy behaves differently across compute platforms.

- **EC2/on-premises** can use in-place deployments or blue/green deployments.
- **Lambda** deployments shift traffic between function versions.
- **ECS** deployments through CodeDeploy use blue/green task sets and traffic routing.

Lambda and ECS do not use the CodeDeploy agent. EC2/on-premises deployments commonly rely on the agent installed on instances.

Lifecycle hook names and AppSpec structure differ by platform. Memorizing one EC2 hook sequence and applying it to Lambda creates the wrong mental model.

---

## Meet the AWS Service

**AWS CodeDeploy** automates deployments to supported compute platforms.

> **Core idea:** An application identifies the deployable system, a deployment group selects targets and policy, and an AppSpec file describes revision content and lifecycle behavior.

CodeDeploy can be invoked directly or by CodePipeline.

---

## How It Works

### Application

A CodeDeploy application is a logical container for deployment configuration associated with one compute platform:

- EC2/on-premises
- AWS Lambda
- Amazon ECS

The application is not the binary itself.

### Deployment Group

A deployment group identifies targets and settings.

For EC2 it can select instances through tags or Auto Scaling groups. For Lambda it identifies the alias and deployment configuration. For ECS it identifies the service and traffic-routing resources.

The group can include:

- Deployment configuration
- Service role
- Load balancer information
- CloudWatch alarms
- Automatic rollback settings
- Notification triggers

### AppSpec

The AppSpec file is unique to CodeDeploy.

For EC2/on-premises, it maps revision files to destinations and declares scripts for lifecycle events such as stopping, installing, starting, and validating the application.

For Lambda, it identifies the function version and alias, plus optional validation Lambda hooks before and after traffic.

For ECS, it identifies the task definition, load-balancer container and port, and optional Lambda validation hooks around traffic phases.

Hooks must be idempotent where reruns are possible and must return meaningful success or failure.

### EC2 In-Place

An in-place deployment updates existing instances. A deployment configuration controls how many must remain healthy, using options such as all-at-once, half-at-a-time, or one-at-a-time.

In-place is resource-efficient but modifies the current hosts. A bad install can leave an instance in a mixed state unless scripts and rollback are carefully designed.

### EC2 Blue/Green

Blue/green creates or uses replacement instances, deploys the revision there, and reroutes traffic after validation. Old instances can remain available for a configured period.

This improves isolation and rollback options but costs additional capacity and requires external state to remain compatible.

### Lambda

CodeDeploy can move an alias from the current published version to a new one using all-at-once, canary, or linear configurations.

Pre-traffic hooks can test the new version before customer traffic. Post-traffic hooks validate after the shift. CloudWatch alarms can stop and roll back the alias routing.

### ECS

CodeDeploy creates a replacement task set for the new task definition. Test traffic can exercise it before production routing moves. After successful validation and the configured wait, the original task set is terminated.

The AppSpec must name the correct task definition, container, and port used by the load balancer.

### Alarms and Rollback

A deployment group can monitor CloudWatch alarms and stop when an associated alarm activates.

Automatic rollback can be configured for deployment failure, alarm activation, or a stopped deployment. A rollback generally creates a new deployment of the last known good revision.

Rollback cannot reverse:

- Database migrations
- Messages already processed
- External calls
- Files modified outside the revision lifecycle
- Incompatible shared state

---

## Architectural Mapping

```text
CodePipeline deploy action
          |
          v
 CodeDeploy application
   deployment group
   AppSpec + revision
          |
  hooks -> traffic -> alarms
          |
 success or rollback deployment
```

The CodeDeploy service role needs permissions for the selected platform and routing resources. Hook functions and instance scripts need their own narrowly scoped application permissions.

---

## When to Use It

Use CodeDeploy when:

- EC2 instances need coordinated install and lifecycle scripts
- Lambda aliases need managed canary or linear shifts
- ECS services use CodeDeploy blue/green task-set routing
- Alarms and rollback should be attached to deployment groups
- The pipeline needs a specialized rollout provider

## When Not to Use It

Do not add CodeDeploy when the target service already provides the exact simple deployment behavior required and no CodeDeploy integration is needed.

Do not use lifecycle scripts to hide an uncontrolled configuration-management system.

---

## Painkiller

> **Problem:** The release action copies files but does not coordinate process state, readiness, traffic, or validation.  
> **Pain:** A technically successful transfer leaves applications stopped, half-configured, or exposed too early.  
> **AWS solution:** Use CodeDeploy applications, deployment groups, AppSpec declarations, platform-specific hooks, alarms, and rollback settings.

---

## Knife Cut

> **The pipeline chooses when deployment begins. CodeDeploy controls how supported compute targets change.**

---

## The Masthead

### What Actually Just Happened

|In the story|In CodeDeploy|What it actually means|
|---|---|---|
|Franchise release program|Application|Logical deployment container for a compute platform|
|Region of stores|Deployment group|Targets and rollout settings|
|Changeover runbook|AppSpec|Revision content and lifecycle declarations|
|Opening checks|Lifecycle hooks|Scripts or Lambda validations around deployment|
|Health bell|CloudWatch alarm|Signal that can stop deployment|
|Recall|Automatic rollback|New deployment of a prior known revision|

---

## A Note From the Author

The rollout manager appears to command every store the same way, but platform contracts are genuinely different. EC2 instances contain mutable hosts and an agent; Lambda traffic points to immutable versions; ECS blue/green replaces task sets.

Design validation around user outcomes. A process can be running while checkout is broken.

- [What is AWS CodeDeploy?](https://docs.aws.amazon.com/codedeploy/latest/userguide/welcome.html)
- [CodeDeploy AppSpec files](https://docs.aws.amazon.com/codedeploy/latest/userguide/application-specification-files.html)
- [Monitoring CodeDeploy with CloudWatch alarms](https://docs.aws.amazon.com/codedeploy/latest/userguide/monitoring-create-alarms.html)

---

## The Last Bite

The rollout mechanism was ready.

Nia still had to choose whether release 4.7.2 should reach every store at once, one store first, ten percent every interval, or a complete parallel fleet.

> **The same artifact can carry radically different risk depending on how quickly traffic meets it.**

---

**Next chapter:** *[AWS Deployment Strategies: How Many Stores Change Tonight?](12-how-many-stores-change-tonight.md)*

The franchise calculates blast radius, capacity, mixed-version time, and rollback speed for every release strategy.

