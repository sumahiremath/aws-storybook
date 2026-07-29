---
description: "Infrastructure updates are safe only when teams understand which resources change in place, which are replaced, and which state must survive either path."
tags:
  - "aws"
  - "deployment"
  - "devops"
  - "cloudformation"
---

# AWS CloudFormation: The Renovation Preview

> Infrastructure updates are safe only when teams understand which resources change in place, which are replaced, and which state must survive either path.

## The Business Goal

Dev's template change was four characters long.

The CloudFormation change set was one line terrifying:

```text
ProductionOrdersTable  Modify  Replacement: True
```

The new property could not be applied to the existing physical resource. CloudFormation planned to create a replacement and remove the old one according to the resource's update behavior.

The template was valid.

Executing it blindly could still separate Byte Burger from its orders.

---

## The Story

Nia refused the renovation.

Dev split the change into safer phases. The new resource would be created first. Application code would understand both old and new forms. Data movement and verification would happen under an explicit plan. Only after traffic moved would the old resource become eligible for removal.

Then he added guardrails:

- A **change set** for preview
- A **stack policy** protecting critical resources during updates
- A **DeletionPolicy** retaining stateful resources if the stack was deleted
- Nested stacks for reusable network and application pieces
- CloudWatch rollback triggers for health-sensitive stack operations

The blueprint could now say not only what to build, but what must not be casually destroyed.

---

## The Wrong Way

A change set is not a simulation.

It compares the current stack with proposed template and parameter changes and shows modeled additions, modifications, removals, and replacements. It cannot guarantee runtime success or predict every application consequence.

CloudFormation rollback also has limits. It attempts to return stack resources to the previous known configuration after a failed operation. It cannot:

- Reverse data already written by application code
- Unsend events or notifications
- Make a breaking schema change backward compatible
- Recover a resource deliberately deleted outside the stack
- Guarantee that a custom resource's side effects are undone

“CloudFormation will roll it back” is not a migration strategy.

---

## Meet the AWS Service

CloudFormation provides update controls around the stack lifecycle.

> **Core idea:** Preview the modeled resource change, protect critical update targets, decide what survives deletion, and make application changes backward compatible before execution.

Stack events reveal the order and status of resource operations. If an update fails, CloudFormation usually attempts an automatic rollback unless configured otherwise.

---

## How It Works

### Change Sets

Creating a change set submits a new template or parameter set without applying it. CloudFormation compares the proposal with the stack.

Review:

- Resources added, modified, or removed
- Properties changing
- Whether replacement is expected
- IAM resources and capabilities
- Nested-stack changes when included

Only execution applies the change set.

Change sets can catch many obvious problems and now perform some early validation, but runtime constraints, permissions, custom logic, and service behavior can still fail later.

### Update and Replacement

Each resource property has update behavior documented in the CloudFormation reference. A change may require:

- No interruption
- Some interruption
- Replacement

Replacement often creates a new physical resource and then deletes the old one. Fixed names, dependencies, data, and quotas can complicate that sequence.

### Stack Policies

A stack policy is a JSON policy that limits update actions on designated stack resources. It is a fail-safe against accidental updates or deletion during stack updates.

It is not an IAM policy and does not decide who may call AWS APIs generally. IAM controls who can perform stack actions. The stack policy restricts what an allowed stack update may change.

### DeletionPolicy and UpdateReplacePolicy

`DeletionPolicy` controls what CloudFormation does to a resource when the resource is removed from the template or its stack is deleted. Common options include retaining it or, for supported resources, taking a snapshot.

`UpdateReplacePolicy` controls what happens to the old physical resource when an update replaces it.

These policies protect infrastructure state, not application consistency. Retaining a database prevents automatic deletion but may leave an orphan that the team must track, secure, and eventually clean up.

### Nested Stacks

Nested stacks let a parent template include reusable CloudFormation stacks. They can separate network, data, and application modules while keeping one hierarchy.

They improve reuse, but operations remain coupled through the parent. Update or delete nested stacks through the root workflow unless the design explicitly calls for independent stacks.

### Drift

Drift occurs when actual resource configuration differs from the stack's expected configuration, often because someone changed a resource outside CloudFormation.

Drift detection helps reveal supported differences. It does not automatically repair them, and not every property or resource supports the same drift visibility.

---

## Architectural Mapping

```text
current stack + proposed template
              |
              v
          change set
      add / modify / remove
       in-place / replace
              |
      review and approve
              |
           execute
              |
     success or rollback attempt
```

A safe data change may use expand-and-contract:

1. Add new compatible structures
2. Deploy code that can use both forms
3. Migrate and verify data
4. Move reads and writes
5. Remove old structures in a later release

That sequencing belongs to the application release plan, not CloudFormation alone.

---

## When to Use It

Use change sets for production stack updates, especially when stateful or security-sensitive resources may change. Use stack policies as a guardrail for critical resources. Use retention or snapshot policies when deleting the stack should not automatically delete important state.

Use nested stacks for cohesive reusable modules with a shared lifecycle.

## When Not to Use It

Do not treat retained resources as backups. Do not rely on stack rollback to undo destructive application migrations. Do not use one nested hierarchy for components that must deploy and fail independently.

---

## Painkiller

> **Problem:** A small template edit would replace a stateful production resource.  
> **Pain:** A syntactically valid update could destroy or detach critical data.  
> **AWS solution:** Preview with change sets, understand property update behavior, protect resources with stack policies, preserve state with lifecycle policies, and stage incompatible migrations.

---

## Knife Cut

> **A change set predicts modeled infrastructure actions. It does not predict every production consequence.**

---

## The Masthead

### What Actually Just Happened

|In the story|In CloudFormation|What it actually means|
|---|---|---|
|Renovation preview|Change set|Proposed stack changes before execution|
|Replace the walk-in freezer|Resource replacement|New physical resource replaces old one|
|Do-not-touch notice|Stack policy|Restricts update actions on protected resources|
|Keep equipment after closure|DeletionPolicy|Controls resource handling on removal/deletion|
|Keep old equipment after replacement|UpdateReplacePolicy|Controls old physical resource after replacement|
|Reusable room plans|Nested stacks|Templates composed into a parent hierarchy|
|Unrecorded remodeling|Drift|Actual configuration differs from stack expectation|

---

## A Note From the Author

Byte Burger can close a room and move inventory neatly. Production systems may process traffic throughout a migration. That is why backward-compatible application behavior, data migration, health observation, and rollback boundaries belong in the plan.

A retained resource also stops being managed by the deleted stack. Ownership, cost, and cleanup do not disappear.

- [Updating stacks with change sets](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/using-cfn-updating-stacks-changesets.html)
- [Protecting stack resources](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/protect-stack-resources.html)
- [CloudFormation DeletionPolicy](https://docs.aws.amazon.com/AWSCloudFormation/latest/TemplateReference/aws-attribute-deletionpolicy.html)

---

## The Last Bite

Nia approved the phased renovation.

The next branch of the franchise used mostly Lambda functions, API routes, queues, and tables. Dev's general blueprint could define all of them, but Marco wanted a serverless workflow that spoke in those shapes directly.

> **A higher-level blueprint can reduce repetition without changing the foundation underneath it.**

---

**Next chapter:** *[AWS SAM: The Serverless Store Plan](05-the-serverless-store-plan.md)*

Marco opens AWS SAM and discovers that shorthand still becomes CloudFormation before the store exists.

