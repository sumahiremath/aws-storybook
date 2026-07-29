---
description: "Elastic Beanstalk manages a web or worker application platform around your code while leaving application behavior, data, configuration, and release choices with your team."
tags:
  - "aws"
  - "deployment"
  - "devops"
  - "elastic-beanstalk"
---

# AWS Elastic Beanstalk: The Turnkey Franchise

> Elastic Beanstalk manages a web or worker application platform around your code while leaving application behavior, data, configuration, and release choices with your team.

## The Business Goal

A three-person franchise team had a conventional web application.

They needed instances, scaling, a load balancer, health checks, logs, deployment policies, platform updates, and environment configuration.

They could assemble every resource themselves.

Doing so would make platform operations their largest product.

The team did not need a custom restaurant.

It needed a reliable one that arrived ready to operate.

---

## The Story

Nia offered them a turnkey franchise.

The team uploaded an application source bundle and selected a supported platform. Elastic Beanstalk created an **environment** and orchestrated the surrounding AWS resources.

The team still chose instance and scaling settings, environment properties, networking, health behavior, and deployment policy.

For release 4.7.2, they created an application version and deployed it first to staging. Production used traffic splitting: a new group of instances received a small share of traffic, remained under health observation, and then replaced the old group.

The platform handled much of the machinery.

The team still owned whether checkout was correct.

---

## The Wrong Way

Elastic Beanstalk is not “serverless,” and it does not make infrastructure disappear.

It creates and manages resources such as EC2 instances, Auto Scaling groups, load balancers, security groups, and monitoring according to the environment configuration.

Changing those resources manually can create configuration drift or be overwritten by later environment operations.

Elastic Beanstalk health can show that instances and web requests look healthy. It cannot decide whether a business outcome—such as charging the correct amount—is valid.

---

## Meet the AWS Service

**AWS Elastic Beanstalk** is a managed service for deploying and scaling web applications and services.

> **Core idea:** You provide an application version and environment configuration; Elastic Beanstalk orchestrates the application platform and offers deployment policies.

An **application** is a logical collection. An **application version** is a labeled, deployable source bundle. An **environment** runs one application version on one platform configuration.

---

## How It Works

### Application Versions

A source bundle contains application code and platform-appropriate files. Elastic Beanstalk stores version information and deploys selected versions to environments.

Labels identify application versions. Keep the label tied to one reviewed source bundle and preserve source revision metadata.

Old versions and source bundles can accumulate cost and clutter. Apply lifecycle management consistent with rollback and audit needs.

### Environments

Common environment tiers are:

- **Web server environment:** serves HTTP requests, often behind a load balancer
- **Worker environment:** processes background work, commonly through an SQS-managed workflow

Environments can be development, staging, or production. CNAMEs and environment URLs help clients reach them, and environment swapping can support some blue/green patterns.

Do not assume environment URLs are permanent identity without planning DNS, certificates, and client behavior.

### Configuration

Configuration can include:

- Instance types and capacity
- Auto Scaling
- Load balancer settings
- Environment properties
- Health reporting
- Logs
- VPC and security settings
- Platform and managed updates

Environment properties are exposed to the application. Do not treat them as a carefree secret vault. Use appropriate secret services and retrieval patterns.

`.ebextensions` configuration files can customize resources and options for supported platform workflows. Platform hooks can run custom commands during deployment. Customization adds power and maintenance responsibility; prefer native options when they express the requirement.

### Docker Platforms

Elastic Beanstalk can run Docker-based applications on supported platforms. The team supplies the appropriate container definition or image references.

Beanstalk remains the environment manager around the application. ECR image access, container health, logs, environment configuration, and underlying capacity still matter.

### Deployment Policies

Elastic Beanstalk supports:

- **All at once:** updates all instances together; may cause a short outage
- **Rolling:** updates batches; temporarily reduces capacity
- **Rolling with additional batch:** launches extra capacity first
- **Immutable:** launches a full new group and keeps original instances untouched until health succeeds
- **Traffic splitting:** launches a new group and sends a configured percentage of traffic to it for evaluation before full shift

The default and availability vary with environment type and creation path. Select deliberately.

### Health and Rollback

Enhanced health reporting combines instance and request information. Deployment settings decide when a batch is healthy enough to continue.

If a deployment fails, redeploy a known previous application version. Immutable and traffic-splitting policies preserve the old group during validation, giving a cleaner return path than modifying the same instances.

Application data changes remain outside that guarantee.

---

## Architectural Mapping

```text
application version + environment configuration
                    |
            Elastic Beanstalk
        /        |        |        \
     EC2     Auto Scaling   load balancer   health
                    |
          deployment policy
```

Elastic Beanstalk uses service roles and instance profiles to manage the environment and give instances approved AWS access. Do not attach broad permissions merely to make deployment succeed.

---

## When to Use It

Use Elastic Beanstalk when:

- A conventional web or worker application fits a supported platform
- The team wants AWS to orchestrate common infrastructure
- Application versions and environment-level deployment policies are useful
- The team needs more control than a highly abstract hosting product but less platform assembly than raw EC2

## When Not to Use It

Do not choose Beanstalk when the application requires a platform shape it does not model cleanly. ECS, EKS, Lambda, or direct infrastructure may better fit specialized container, Kubernetes, event-driven, or custom host requirements.

Do not heavily mutate Beanstalk-managed resources outside its configuration model.

---

## Painkiller

> **Problem:** A small team spends more effort assembling and operating a standard application platform than delivering its application.  
> **Pain:** Load balancing, scaling, health, platform updates, and rollout machinery become undifferentiated work.  
> **AWS solution:** Deploy versioned source bundles to Elastic Beanstalk environments, configure the managed platform, and choose an explicit deployment policy.

---

## Knife Cut

> **Elastic Beanstalk manages the platform around your application. It does not manage the correctness inside your application.**

---

## The Masthead

### What Actually Just Happened

|In the story|In Elastic Beanstalk|What it actually means|
|---|---|---|
|Franchise brand|Application|Logical collection of versions and environments|
|Numbered operating kit|Application version|Deployable source bundle with label|
|One operating restaurant|Environment|Running application version and managed resources|
|Counter restaurant|Web server tier|HTTP-serving application environment|
|Prep center|Worker tier|Background work processing environment|
|Turnkey equipment plan|Platform and configuration|Managed infrastructure settings|
|Pilot customers|Traffic splitting|Canary traffic to fresh instances|

---

## A Note From the Author

“Managed” shifts responsibility; it does not erase it. Elastic Beanstalk manages orchestration according to configuration, while the team owns application code, dependency security, data, IAM choices, environment settings, health criteria, and cost.

Review which resources Beanstalk will replace during platform and configuration updates. Stateful application data should not live only on disposable instances.

- [Elastic Beanstalk concepts](https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/concepts.html)
- [Elastic Beanstalk deployment policies](https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/using-features.rolling-version-deploy.html)

---

## The Last Bite

By Friday evening, every franchise team had a deployment path appropriate to its platform.

Release 4.7.2 was live.

At 6:04 p.m., checkout latency began climbing in one Region.

The deployment was complete.

The work was not.

> **A release is not healthy because the deployment service says “Succeeded.”**

---

**Next chapter:** *[AWS Deployment: The Morning After the Rollout](14-the-morning-after-the-rollout.md)*

The franchise turns to logs, metrics, traces, alarms, and root-cause analysis to understand what changed after customers arrived.

