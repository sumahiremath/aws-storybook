---
description: "Infrastructure as code replaces remembered setup steps with a versioned declaration of desired resources and their relationships."
tags:
  - "aws"
  - "deployment"
  - "devops"
  - "cloudformation"
---

# AWS CloudFormation: The Franchise Blueprint

> Infrastructure as code replaces remembered setup steps with a versioned declaration of desired resources and their relationships.

## The Business Goal

The container now matched across stores.

The stores did not.

Phoenix sent logs to one group. Portland had a queue with the wrong retention. Austin's execution role still contained a permission copied from an experiment. Two API stages pointed at different Lambda aliases.

Each store manager had followed the same installation document.

Each had interpreted it slightly differently.

Nia could reproduce the application. She could not reproduce the place where it ran.

---

## The Story

Dev unrolled a franchise blueprint across the table.

It did not say:

> “Create a queue, remember its URL, then paste that URL into the function.”

It declared both resources and the relationship between them.

One blueprint described:

- The queue
- The function
- The function's role
- The event connection
- The API
- The environment-specific inputs
- The outputs another stack would need

CloudFormation would inspect dependencies, call the relevant service APIs, record events, and manage the resources as a **stack**.

Store managers would stop improvising installation.

They would supply approved parameters to the same reviewed template.

---

## The Wrong Way

A shell script can automate commands, but it often describes a sequence rather than the complete desired state. If a command partially succeeds or a resource already exists, rerunning the script may produce another outcome.

Manual console work is even harder to review, repeat, and compare.

Infrastructure as code is not automatically safe. A template can consistently create an insecure role, public resource, or destructive replacement. It makes intent reviewable and repeatable; the team must still make the intent correct.

---

## Meet the AWS Service

**AWS CloudFormation** provisions and manages AWS resources from JSON or YAML templates.

> **Core idea:** A template declares logical resources and relationships. A stack is a deployed instance of that template in an AWS account and Region.

CloudFormation determines creation order from references and dependencies. It tracks logical resource IDs in the template and maps them to physical resources created by AWS services.

The CloudFormation service needs authorization to perform the actions required by the stack. Teams can use a service role so stack operations consistently use defined permissions.

---

## How It Works

### Resources

The `Resources` section is the only required major template section. Each logical resource has a type, such as `AWS::SQS::Queue`, and properties.

```yaml
Resources:
  OrderQueue:
    Type: AWS::SQS::Queue
    Properties:
      VisibilityTimeout: 60
```

The logical ID `OrderQueue` remains the template's identity for the resource. CloudFormation creates or updates the physical queue according to property behavior.

### Parameters

Parameters accept deployment-time inputs such as environment names, instance sizes, or approved image references.

Use parameters for values that genuinely vary. Too many parameters can turn a template into an untestable configuration language.

`NoEcho` hides parameter values from some displays, but it is not a complete secret-management system. Prefer dynamic references or integrations with managed secret stores for sensitive values.

### References and Intrinsic Functions

`Ref` and functions such as `Fn::GetAtt`, `Fn::Sub`, and `Fn::Join` connect resource values without manual copying.

When a function references a queue ARN, CloudFormation can infer a dependency and create the queue before configuring the consumer.

### Mappings and Conditions

Mappings provide lookup tables, often for known values by Region or environment class. Conditions decide whether resources or properties should exist for a deployment.

For example, production might create an alarm and deletion protection while a short-lived development stack does not.

Conditions are evaluated from parameters, pseudo parameters, mappings, and functions. They should express legitimate environment differences, not hide uncontrolled drift.

### Outputs

Outputs publish useful stack values such as an API URL or queue ARN. Other stacks can import exported values, though cross-stack exports also create update and deletion coupling.

Outputs are not a safe place for secrets.

### Templates, Stacks, and Environments

The same template can create separate development, test, and production stacks with different approved parameters.

That does not mean every environment must be identical in size. It means differences are declared, reviewed, and reproducible.

---

## Architectural Mapping

```text
template.yaml
  Parameters ------ environment inputs
  Mappings -------- approved lookup values
  Conditions ------ intentional differences
  Resources ------- desired AWS resources
  Outputs --------- published handoff values
        |
        v
  CloudFormation stack
        |
  service API calls and stack events
```

CloudFormation stack events are the first place to inspect when resource creation or update fails. The underlying service may provide the more specific reason, such as an invalid property, missing permission, or quota.

---

## When to Use It

Use CloudFormation when:

- Resources must be recreated consistently
- Infrastructure changes need code review and history
- Environments should differ only through declared inputs
- Application and infrastructure releases must coordinate
- Rollback and stack event history are valuable

## When Not to Use It

Do not place rapidly changing business data in a CloudFormation template. Do not use stack parameters as a substitute for a proper secrets or configuration service.

Avoid one enormous stack when independent components have different lifecycles and ownership. Avoid excessive cross-stack exports that prevent teams from changing or deleting stacks independently.

---

## Painkiller

> **Problem:** Each store follows a setup document and accumulates unique infrastructure.  
> **Pain:** Identical artifacts encounter different permissions, queues, endpoints, and logging.  
> **AWS solution:** Declare resources, relationships, and intentional environment differences in CloudFormation templates, then deploy them as managed stacks.

---

## Knife Cut

> **A template is the declaration. A stack is one deployed instance. A logical ID is CloudFormation's handle on a physical resource.**

---

## The Masthead

### What Actually Just Happened

|In the story|In CloudFormation|What it actually means|
|---|---|---|
|Franchise blueprint|Template|Versioned resource declaration|
|Store-specific inputs|Parameters|Values supplied during stack operation|
|Regional lookup chart|Mappings|Static key-to-value lookup|
|Optional equipment|Conditions|Resources or properties created selectively|
|Equipment and wiring|Resources and references|AWS resources and dependencies|
|Opening-day handoff|Outputs|Values exposed after deployment|
|One restaurant installation|Stack|Managed resource collection|

---

## A Note From the Author

The blueprint sounds static, but CloudFormation manages lifecycle. A property update may happen without interruption, require interruption, or replace the physical resource. That behavior belongs to each resource type and property.

Templates should be validated and reviewed, but successful syntax validation does not prove the architecture or update is safe.

- [CloudFormation template anatomy](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/template-anatomy.html)
- [CloudFormation resource and property reference](https://docs.aws.amazon.com/AWSCloudFormation/latest/TemplateReference/aws-template-resource-type-ref.html)

---

## The Last Bite

The next staging stack matched the blueprint exactly.

Then Dev changed one innocent-looking property and the preview marked the production database:

```text
Replacement: True
```

> **Repeatable change is not the same thing as harmless change.**

---

**Next chapter:** *[AWS CloudFormation: The Renovation Preview](04-the-renovation-preview.md)*

Before Nia renovates two hundred stores, she learns to preview replacement, protect critical resources, and plan what rollback cannot restore.

