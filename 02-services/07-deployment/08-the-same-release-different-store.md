---
description: "Promote one approved application version. Supply environment-specific endpoints, identities, and configuration without secretly rebuilding it."
tags:
  - "aws"
  - "deployment"
  - "devops"
  - "appconfig"
---

# AWS AppConfig: The Same Release, Different Store

> Promote one approved application version. Supply environment-specific endpoints, identities, and configuration without secretly rebuilding it.

## The Business Goal

The staging build passed.

For production, Marco changed the payment URL, rebuilt the artifact, and deployed it.

The production bits had never passed staging.

Worse, the new menu needed to be disabled in one Region after a supplier delay. The menu flag lived inside code, so changing one value required another complete application deployment.

Nia had one release in name and several releases in practice.

---

## The Story

Imani drew three stores:

```text
DEV       TEST       PROD
same application version
different approved environment values
```

The API had separate stages. The production custom domain mapped customers to the intended stage without exposing an internal stage URL.

Lambda used published versions and stable aliases such as `test` and `prod`. The frontend created branch-associated preview environments. Container environments referenced the approved image identity.

The limited-time menu moved into AppConfig. A validator rejected malformed configuration before deployment. The change reached a small percentage of targets first. CloudWatch alarms watched checkout failures during the rollout and bake time.

The software no longer had to move merely because one business setting changed.

---

## The Wrong Way

Environment variables are not a complete environment strategy.

They are useful for small runtime values, but they can become an unversioned dumping ground. Sensitive values may leak through logs or configuration views. Large, dynamic configuration can require restarts or redeployment.

Stage variables can pass stage-specific strings or Lambda alias names, but they are not secret storage and should not become hidden application logic.

Using one mutable `prod` label without recording the version behind it also destroys traceability. Stable names are valuable only when their targets are observable.

---

## Meet the AWS Services

AWS services provide environment boundaries at different layers.

> **Core idea:** Versioned artifacts remain fixed; environment pointers and configuration select how those artifacts behave and where users reach them.

- **API Gateway stages** expose deployments with stage-specific settings.
- **API Gateway custom domains** give clients stable hostnames mapped to API stages.
- **Lambda versions** are immutable snapshots of code and qualifying configuration.
- **Lambda aliases** are named pointers to published versions and can split traffic between two versions.
- **AWS AppConfig** validates and deploys configuration or feature flags gradually.
- **AWS Amplify** can associate frontend deployments with branches.
- **AWS Copilot** can organize container applications into named environments.

---

## How It Works

### API Gateway Stages

An API deployment is a snapshot of API configuration made available through a stage. Stages can have their own logging, throttling, variables, and other settings.

Development, test, and production stages can expose different backends or configurations. Keep the differences declared and controlled.

Custom domain names map friendly hostnames and base paths to API stages. Clients can keep calling a stable production URL while the team changes the deployment behind the mapping.

### Stage Variables

Stage variables are stage-specific name-value pairs. An integration can use a variable such as a Lambda alias name so the `test` stage invokes a test alias and `prod` invokes a production alias.

They are not encrypted secret values. Use Secrets Manager or another appropriate service for secrets.

### Lambda Versions and Aliases

Publishing a Lambda version creates an immutable snapshot of code and qualifying configuration. `$LATEST` remains mutable.

An alias such as `prod` points to a published version. Updating the alias moves the stable name. A weighted alias can direct a percentage of invocations to a second published version.

Event sources and permissions should target the alias when alias-based isolation or traffic shifting is intended.

### AppConfig

AppConfig separates runtime configuration and feature flags from application deployment.

A safe flow includes:

1. Store or reference configuration
2. Validate syntax and semantics
3. Start a deployment to an environment
4. Increase target exposure according to the strategy
5. Monitor CloudWatch alarms
6. Observe a final bake time
7. Roll back automatically if configured alarms enter `ALARM`

Applications retrieve current configuration, commonly through the AppConfig Agent, which caches values and reduces direct retrieval work.

AppConfig supports all-at-once, linear, and canary strategies. A feature flag can decouple **deployment** from **release**: code can be present while the feature remains disabled.

### Branch and Container Environments

Amplify can build and deploy frontend branches into distinct environments or previews. Use branch protection and promotion rules so a preview is not mistaken for an approved production release.

Copilot environments provide named infrastructure contexts for container services. Image identity, secrets, roles, and scaling remain part of the environment contract.

---

## Architectural Mapping

```text
approved version 4.7.2
      /       |        \
  dev alias test alias prod alias
      |       |        |
  dev stage test stage prod stage <- custom domain
                       |
                  AppConfig
              validated menu flag
```

Keep environment permissions separate. A development pipeline should not casually deploy to production. Cross-account promotion commonly uses scoped roles and a production approval boundary.

---

## When to Use It

Use explicit service environments when:

- Dev, test, and production need separate endpoints and permissions
- One artifact should be promoted without rebuilding
- Clients need stable names while versions change
- Feature exposure should be independent from code deployment
- Configuration needs validation, gradual rollout, alarms, and rollback

## When Not to Use It

Do not create dozens of permanent environments without ownership and cleanup. Do not put secrets in stage variables, source-controlled config, or frontend builds.

Do not use a feature flag to support two architectures forever. Old paths create operational and testing cost.

---

## Painkiller

> **Problem:** Environment values are compiled into separately rebuilt artifacts, and every configuration change requires a full deployment.  
> **Pain:** Production runs untested bits and small business changes carry application-level blast radius.  
> **AWS solution:** Promote immutable versions, route through stages and aliases, use stable custom domains, and deploy validated runtime configuration separately with AppConfig.

---

## Knife Cut

> **Deployment puts code in place. Configuration changes behavior. Release decides who experiences it.**

---

## The Masthead

### What Actually Just Happened

|In the story|In AWS|What it actually means|
|---|---|---|
|Numbered recipe edition|Lambda version|Immutable published snapshot|
|Store sign|Lambda alias|Named pointer to version or weighted pair|
|Practice and live counters|API Gateway stages|Named deployed API environments|
|Customer-facing address|Custom domain mapping|Stable hostname routed to a stage|
|Regional menu card|AppConfig configuration|Runtime behavior deployed separately|
|Preview storefront|Amplify branch|Branch-associated frontend environment|

---

## A Note From the Author

Configuration can break production as thoroughly as code. Treat it as versioned, reviewed, validated deployment material with owners and alarms.

Lambda versions include code and qualifying configuration, but not every surrounding resource. An alias rollback cannot revert an API mapping, database schema, or AppConfig change unless those are coordinated separately.

- [Manage Lambda versions](https://docs.aws.amazon.com/lambda/latest/dg/configuration-versions.html)
- [Lambda weighted aliases](https://docs.aws.amazon.com/lambda/latest/dg/configuring-alias-routing.html)
- [What is AWS AppConfig?](https://docs.aws.amazon.com/appconfig/latest/userguide/what-is-appconfig.html)

---

## The Last Bite

The same release candidate now passed development, test, and staging without being rebuilt.

Marco still created the package on his laptop.

When Imani rebuilt the same commit, the checksum differed.

> **A promotion path is only trustworthy when the build at its entrance is reproducible.**

---

**Next chapter:** *[AWS CodeBuild: The Central Build Kitchen](09-the-central-build-kitchen.md)*

The central build kitchen starts from a clean room, follows a checked-in buildspec, and produces the artifact every later stage will trust.

