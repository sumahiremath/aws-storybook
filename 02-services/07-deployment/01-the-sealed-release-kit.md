---
description: "Source code is a recipe. A deployment package is the versioned, runtime-compatible result that another environment can actually execute."
tags:
  - "aws"
  - "deployment"
  - "devops"
  - "concept"
---

# AWS Deployment Packages: The Sealed Release Kit

> Source code is a recipe. A deployment package is the versioned, runtime-compatible result that another environment can actually execute.

## The Business Goal

The first pilot store rejected release 4.7.2 before it served one order.

The Lambda function that calculated promotional prices started, imported a library, and failed.

Marco stared at the error. The library worked on his laptop.

That was precisely the problem. His laptop contained months of accumulated tools and packages. The release ZIP contained only the function file.

The store had received the recipe without one of its ingredients.

---

## The Story

Imani emptied the release box onto a stainless-steel table.

There were four separate things:

1. Marco's source code
2. Third-party and internal dependencies
3. Configuration describing the handler and runtime
4. The infrastructure template that would install it

She rebuilt the package in a clean environment. Every dependency came from a declared manifest. The function handler sat at the directory level Lambda expected. Native libraries were built for the target Linux runtime and processor architecture.

Then she numbered the output and sealed it.

No one would add a file on the way to production.

---

## The Wrong Way

Copying a development directory often includes too much and still misses what matters. Local caches, tests, credentials, editor files, and platform-specific binaries can enter the bundle while a required runtime library stays outside it.

Another common confusion is the word **artifact**.

- A **software package dependency** is a versioned library consumed during a build. CodeArtifact can store these packages.
- A **build artifact** is output produced by the build and passed to later release steps. CodePipeline commonly moves these through an artifact store.
- A **container image** is a packaged filesystem and metadata stored in a registry such as ECR.

They participate in one release, but they are not interchangeable.

---

## Meet the AWS Services

Lambda supports two primary deployment package types: ZIP archives and container images.

> **Core idea:** Choose the package form that fits the runtime and build needs, then keep its contents reproducible and versioned.

**AWS CodeArtifact** stores software packages in repositories and domains. Package managers such as npm, Maven, NuGet, and pip can retrieve approved public or private versions from it.

**Lambda layers** are versioned ZIP archives containing supplementary code or data. Lambda extracts layer content under `/opt` in the execution environment.

Layers can reduce duplicated dependency packaging and separate shared components from function code, but they create another version relationship that must be tested.

---

## How It Works

### Lambda ZIP Archives

A ZIP package contains function code and, unless the runtime already provides them or layers supply them, dependencies.

Directory structure matters. Lambda must be able to find the handler and import packages from the runtime search path. Permissions and executable bits can matter for custom binaries.

ZIP packaging is a strong fit for:

- Small and conventional functions
- Supported managed runtimes
- Straightforward dependencies
- Fast build and upload workflows

Large uploaded ZIPs may be placed in Amazon S3 and referenced during deployment rather than sent inline.

### Lambda Layers

A layer can contain:

- Shared libraries
- A custom runtime
- Configuration data
- Monitoring or instrumentation components

A function references specific layer versions. Updating a layer does not magically mutate every deployed function into a tested combination. Publish a new layer version, update function configuration deliberately, and test the pairing.

For some compiled languages, bundling dependencies directly with the function can be simpler and more efficient than using layers.

### Lambda Container Images

A Lambda container image packages code, dependencies, operating-system libraries, and runtime components with more control over the build.

Use it when:

- Dependencies are large or operationally complex
- A custom runtime or system library is important
- The organization already has a container build workflow
- Local container tooling improves reproducibility

The image must satisfy Lambda's container requirements and be stored in ECR. A function created with one package type cannot simply be changed in place to the other package type; create the appropriate function resource and migration plan.

### Approved Dependencies with CodeArtifact

CodeArtifact repositories can connect to upstream repositories and cache requested packages. Teams can publish internal packages and use origin controls to constrain where new versions come from.

Lockfiles and exact versions still matter. A private repository does not make an unpinned dependency reproducible.

The build role needs permission to authenticate and read only the required repositories. Publishing roles should be narrower than consuming roles.

### Environment Configuration

Do not bake production endpoints or secrets into a package merely to make it “complete.”

The artifact should contain code and stable runtime assets. Environment-specific values can enter through:

- Deployment parameters
- Lambda environment variables
- AppConfig
- Systems Manager Parameter Store
- Secrets Manager

Keep secrets out of source, ZIP contents, container layers, and build logs.

---

## Architectural Mapping

```text
source + dependency manifest
          |
          v
  clean build environment ----> CodeArtifact packages
          |
     test and package
       /          \
 Lambda ZIP     container image -> ECR
   + layers
       \          /
        identified release
```

The same package should move from test to production. Environment values change around it, not inside it.

---

## When to Use It

Use a ZIP when the function and its dependencies fit a conventional Lambda build. Use layers for genuinely shared, independently versioned supplementary content. Use a container image when system dependencies, custom runtimes, or existing container workflows justify the additional machinery.

Use CodeArtifact when teams need controlled, shareable package versions and upstream management.

## When Not to Use It

Do not introduce a layer solely to make a ZIP look smaller if it complicates testing and version ownership. Do not use a Lambda container image because the word “container” sounds more portable; Lambda remains the runtime contract.

Do not mistake a dependency repository for the place where a pipeline stores every deployable artifact.

---

## Painkiller

> **Problem:** The pilot receives source without the exact dependencies and runtime layout used during testing.  
> **Pain:** Imports fail, native libraries mismatch, and each environment quietly builds something different.  
> **AWS solution:** Build clean, declared Lambda ZIP or container packages; version shared layers; retrieve approved dependencies from CodeArtifact; inject environment configuration separately.

---

## Knife Cut

> **Dependencies are inputs. The build artifact is output. Configuration decides how that output behaves in an environment.**

---

## The Masthead

### What Actually Just Happened

|In the story|In AWS|What it actually means|
|---|---|---|
|Recipe|Source code|Human-authored application input|
|Approved pantry|CodeArtifact repository|Versioned packages for builds|
|Compact kit|Lambda ZIP|Code and dependencies in expected layout|
|Shared equipment crate|Lambda layer version|Supplementary content loaded under `/opt`|
|Complete kitchen module|Lambda container image|Container-packaged function stored in ECR|
|Store instructions|Environment configuration|Values supplied outside the immutable artifact|

---

## A Note From the Author

The sealed box is a useful memory device, but a real release can reference several immutable objects: an application artifact, a template, a Lambda layer version, and a container digest. Reproducibility comes from identifying the complete set and preventing unreviewed substitution.

Runtime compatibility matters more than where a package was assembled. Native dependencies built for the wrong operating system or processor architecture can fail even when every filename is present.

- [Configuring Lambda functions](https://docs.aws.amazon.com/lambda/latest/dg/lambda-functions.html)
- [Packaging Lambda layers](https://docs.aws.amazon.com/lambda/latest/dg/packaging-layers.html)
- [AWS CodeArtifact concepts](https://docs.aws.amazon.com/codeartifact/latest/ug/codeartifact-concepts.html)

---

## The Last Bite

The rebuilt function passed.

Then the next store pulled the image tagged `release-4.7.2` and received different bytes from the pilot.

The box was sealed.

The label was not.

> **A release name is useful only when it points to one unchanging thing.**

---

**Next chapter:** *[Amazon ECR and Amazon ECS: The Container Depot](02-the-container-depot.md)*

Nia follows the container from the build kitchen to the image depot—and discovers why a friendly label is not an identity.
