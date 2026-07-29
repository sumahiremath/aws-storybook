---
description: "Use browser-based and local developer tools to inspect and operate AWS deliberately, while treating generated output as code that still requires review and tests."
tags:
  - "aws"
  - "deployment"
  - "devops"
  - "cloudshell"
  - "cli"
  - "amazon-q-developer"
---

# AWS CloudShell, AWS CLI, and Amazon Q Developer: The Franchise Control Desk

## The Business Goal

Shreya is investigating a failed rollout from a borrowed laptop. She needs an approved way to inspect the account, query deployment evidence, and make a narrowly authorized repair—without installing a private toolbox or asking for permanent credentials.

## The Story

The franchise control desk gives an authorized manager a temporary workstation inside the building. The same operating instructions can also be used from a carefully configured desk outside the building. An assistant can draft a checklist or test case, but the manager still reads it, tests it, and owns the decision.

## Meet the Tools

CloudShell is a browser-accessible shell environment for working with AWS from the console using the signed-in user’s permissions. The AWS CLI is the command-line interface that lets code, scripts, and operators call AWS APIs using configured credentials and Region context.

Amazon Q Developer can assist with explanations, code, and tests. It does not grant permissions, validate a design, or make generated output safe to deploy by itself.

## How It Works

Before a command changes anything, establish the account, Region, identity, intended resource, and evidence needed afterward. Prefer roles and temporary credentials; keep commands repeatable; avoid printing secrets; and treat a production mutation as an audited operational action, not an exploratory experiment.

Use CloudShell when a browser-based AWS environment is useful. Use the CLI in local automation when the credential chain, configuration, and review process are explicit. Use Q Developer to accelerate a well-defined task, then review and test the result in the same pipeline as handwritten code.

## The Last Bite

The control desk makes AWS easier to operate. It does not remove the need to know whose authority is being used or what the command will change.

**Next section:** *[Amazon CloudWatch, AWS X-Ray, and AWS CloudTrail: The Operations Room](../08-observability/00-observability-operations-room.md)*

The release has reached production and the control desk can inspect it. Now the team needs evidence that explains what the new version is doing for real customers.
