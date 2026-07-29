---
description: "AWS SAM gives serverless applications a concise template and local workflow, then transforms the definition into CloudFormation for deployment."
tags:
  - "aws"
  - "deployment"
  - "devops"
  - "sam"
---

# AWS SAM: The Serverless Store Plan

> AWS SAM gives serverless applications a concise template and local workflow, then transforms the definition into CloudFormation for deployment.

## The Business Goal

Marco's promotional-order service needed three Lambda functions, an API, a queue trigger, permissions, and deployment controls.

He could define every underlying resource directly in CloudFormation.

But each function repeated packaging, event, permission, and API wiring. The real application shape disappeared inside plumbing.

Then one developer tested the handler locally, another deployed a stale build directory, and staging ran code that was not in the current source tree.

The serverless blueprint was correct.

The developer path around it was not.

---

## The Story

Marco drew a smaller franchise plan.

Instead of spelling out every pipe, he declared:

- A serverless function
- Its source location and handler
- An API event
- A queue event
- Environment inputs
- A deployment preference

The AWS SAM transform expanded that shorthand into CloudFormation resources.

Then the SAM CLI gave the team one repeatable path:

```text
sam build -> sam local -> sam deploy
```

`sam build` gathered dependencies into the expected build structure. Local commands invoked functions or hosted API routes in containers. `sam deploy` packaged referenced artifacts and operated the CloudFormation deployment.

The test kitchen finally used the same plan as the production franchise.

---

## The Wrong Way

AWS SAM is not a separate infrastructure engine replacing CloudFormation.

The SAM template is transformed into CloudFormation. Stack behavior, IAM capabilities, resource replacement, change sets, and rollback boundaries still matter.

Local execution is also not production proof. It can improve feedback for handlers and API flows, but it does not perfectly reproduce managed networking, IAM evaluation, service quotas, concurrency, retries, or every event-source behavior.

Finally, the `.aws-sam/build` directory is generated output. Editing it creates changes that disappear at the next build. Change source and manifests, then rebuild.

---

## Meet the AWS Service

**AWS Serverless Application Model**, or AWS SAM, is an open-source framework for building serverless applications on AWS.

> **Core idea:** SAM combines serverless-focused infrastructure shorthand with a CLI for build, local test, package, sync, and deployment workflows.

Common resource types include:

- `AWS::Serverless::Function`
- `AWS::Serverless::Api`
- `AWS::Serverless::HttpApi`
- `AWS::Serverless::StateMachine`
- `AWS::Serverless::LayerVersion`

SAM can generate supporting permissions and event-source resources from the declared events, but generated permissions still require review.

---

## How It Works

### Template and Transform

A SAM template declares the transform and serverless resources.

```yaml
Transform: AWS::Serverless-2016-10-31

Resources:
  PriceFunction:
    Type: AWS::Serverless::Function
    Properties:
      CodeUri: src/
      Handler: app.handler
      Runtime: python3.13
      Events:
        PriceApi:
          Type: Api
          Properties:
            Path: /price
            Method: post
```

The transform expands this into CloudFormation resources before deployment.

### Build

`sam build` reads the template and dependency manifests, prepares functions and layers, and writes organized build artifacts under `.aws-sam`.

Container-based builds can help compile native dependencies in a runtime-compatible environment. Cached and parallel builds can speed iteration, but a clean release build remains important evidence.

### Local Test

Useful commands include:

- `sam local invoke` for a function and test event
- `sam local start-api` for local API routes
- `sam local start-lambda` for Lambda-compatible local endpoints

Store representative JSON events in source control. Include malformed input, duplicates, optional fields, retries, and events from each important integration.

### Deploy

`sam deploy --guided` can establish stack, Region, capabilities, parameters, and artifact storage configuration. A `samconfig` file can record named environment settings.

Use separate configuration environments for development and production values. Do not place secrets in the config file merely because it is convenient.

SAM deploys through CloudFormation, so a production workflow should review the resulting changes and stack events.

### Gradual Lambda Deployments

`AutoPublishAlias` can publish a new Lambda version and update a named alias. A `DeploymentPreference` can request a canary or linear rollout.

SAM creates the supporting CodeDeploy resources. Pre-traffic and post-traffic hooks can validate the new version. CloudWatch alarms can cause CodeDeploy to stop and roll back the traffic shift.

The callers must invoke the alias-qualified function for the traffic policy to matter.

### Stack Updates Across Environments

The same SAM application can be deployed to separate stacks with environment-specific parameters and configuration profiles.

Promotion should still identify the code and template being moved. Running `sam deploy` from an unclean local directory is not a controlled release process.

---

## Architectural Mapping

```text
source + template + test events
             |
         sam build
             |
   generated build directory
       /             \
 sam local         sam deploy
 local containers       |
                 CloudFormation stack
                        |
                Lambda alias rollout
```

SAM CLI credentials act through the caller's AWS permissions. Deployed resources use their own execution roles. The deployment role and the application role should not be conflated.

---

## When to Use It

Use SAM when:

- The application is primarily serverless
- Concise event and function definitions improve clarity
- Local Lambda and API testing helps the developer loop
- CloudFormation remains the desired deployment foundation
- Lambda versions, aliases, and gradual rollout should be declared with the application

## When Not to Use It

Do not force a broad, non-serverless platform into SAM merely because it can include ordinary CloudFormation resources. Plain CloudFormation or CDK may provide a clearer model.

Do not rely on local emulation as the only integration test.

---

## Painkiller

> **Problem:** Serverless infrastructure repeats low-level wiring, while developers build and test through inconsistent local steps.  
> **Pain:** Application intent is obscured and staging can receive stale or differently packaged code.  
> **AWS solution:** Declare serverless resources in SAM, build with the SAM CLI, exercise local test events and APIs, and deploy the transformed application through CloudFormation with optional gradual Lambda rollout.

---

## Knife Cut

> **SAM is the serverless authoring and developer workflow. CloudFormation remains the deployment engine.**

---

## The Masthead

### What Actually Just Happened

|In the story|In AWS SAM|What it actually means|
|---|---|---|
|Serverless franchise plan|SAM template|Concise serverless infrastructure definition|
|Expanded construction plan|SAM transform output|CloudFormation resources generated from shorthand|
|Prepare the kit|`sam build`|Resolve and organize deployable artifacts|
|Counter rehearsal|`sam local`|Container-based local invocation or API testing|
|Open the store|`sam deploy`|Package artifacts and deploy a CloudFormation stack|
|Numbered menu sign|`AutoPublishAlias`|Publish version and move a Lambda alias|
|Pilot orders|`DeploymentPreference`|CodeDeploy-managed traffic shift and hooks|

---

## A Note From the Author

SAM reduces syntax, not responsibility. Review generated IAM permissions, keep environment values deliberate, inspect CloudFormation changes, and test managed-service behavior after deployment.

The local container is closer to Lambda than a bare local process, but it is not the AWS control plane or every downstream service.

- [How AWS SAM works](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/what-is-sam-overview.html)
- [Building with AWS SAM](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/using-sam-cli-build.html)
- [Gradual deployments with AWS SAM](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/automating-updates-to-serverless-apps.html)

---

## The Last Bite

Marco could now read the serverless plan without wading through every generated pipe.

Dev looked at the franchise's repeated security, networking, monitoring, and service patterns and wanted something else: reusable components with tests and programming-language tools.

> **Shorthand reduces repetition in a template. Constructs can package design decisions into reusable building blocks.**

---

**Next chapter:** *[AWS CDK: The Blueprint Factory](06-the-blueprint-factory.md)*

Dev opens AWS CDK and builds a blueprint factory—but the factory's final product is still a CloudFormation template.

