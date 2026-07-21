# AMI: Bottle the Shop Setup, Not the Business

> An Amazon Machine Image captures the approved starting configuration used to launch repeatable EC2 instances.

## The Pain

The first boba shop took weeks to prepare.

The owner chose the operating system for the point-of-sale machine.

She installed the ordering application.

She installed the approved Ruby runtime.

She configured the receipt printer.

She added monitoring and security agents.

She created startup services.

She tested the entire setup until the shop opened cleanly every morning.

Then the business became popular.

A second location needed to open.

Someone pulled out a checklist.

```text
Install this package.
Download that runtime.
Copy this configuration file.
Start this service.
Change this setting.
Try to remember what Maya fixed last Tuesday.
```

The second shop looked almost like the first one.

Almost.

One package was missing.

The ordering application used an older dependency.

The monitoring agent never started.

The store opened successfully during testing and failed when the lunch crowd arrived.

The company did not have two identical shops.

It had one approved shop and one handcrafted interpretation of it.

Repeatability cannot depend on someone remembering every installation step.

---

## The Story

### The Shop Everyone Wants to Copy

The original boba shop is running beautifully.

Its point-of-sale software is installed.
Its menu application starts automatically.
Its payment integration is configured.
Its monitoring agent reports correctly.
Its operating system has the approved patches.
Its application runtime uses the approved version.

When the shop opens, its systems begin in a known state.

The owner wants five more locations with the same operating capability.

She does not merely need a photograph of the counter.

A photograph would not capture:

- which software was installed
- which runtime version was approved
- how the services were configured
- which agents start automatically
- which system files were required
- how the machine should boot

She needs a reusable starting model of the shop's operating setup.

Not today's customers.
Not the drinks currently being made.
Not the cash in the register.
Not the production database.

The setup.

The owner packages that approved starting configuration so it can be used again.

Now a new shop does not begin with an empty machine and a nervous checklist.

It begins from the same approved operating model.

That is an Amazon Machine Image.

---

## Meet the Amazon Machine Image

An Amazon Machine Image, or **AMI**, provides the software image needed to set up and boot an EC2 instance.

It includes:

- the operating system
- software captured in the image
- baseline configuration stored on the image volumes
- block-device mappings that describe which storage volumes EC2 should create and attach

Every EC2 instance launches from an AMI.

> **Core idea: An AMI is the approved starting image used to create a new EC2 instance.**

In the boba company:

```text
AMI
=
the approved operating package for a new shop
```

That operating package may include:

- Amazon Linux
- the approved Ruby version
- the boba ordering application
- web-server software
- monitoring and security agents
- system libraries
- startup services
- database drivers or clients

The actual production database usually lives outside the image in a shared data service such as Amazon RDS or Amazon Aurora.

The AMI gives the shop the ability to connect to the database.

It should not carry a private copy of the business inside every new shop.

---

## The Boba Mapping

| Boba company | AWS | Meaning |
|---|---|---|
| Approved shop operating package | AMI | Reusable operating-system and software image |
| Running shop | EC2 instance | Independent machine created from the AMI |
| Shop capacity | Instance type | CPU, memory, network, and hardware capacity |
| Environment-specific opening order | Launch template | Versioned parameters used to launch an instance |
| Opening-day instructions | User data | Commands or scripts run when the instance starts |
| Storage-room copy | EBS snapshot | Point-in-time copy of an EBS volume |
| Fleet manager | Auto Scaling group | Maintains the required number of instances |
| Customer dispatcher | Load balancer | Sends requests to healthy instances |

The distinction matters:

> **The AMI standardizes what every shop knows how to run.**
> **The launch template specializes how a particular shop should be launched.**

---

## How It Works

### Step 1: Prepare the Reference Shop

The company configures one EC2 instance.

It installs:

- the approved operating system
- required packages
- the application runtime
- the application
- monitoring and security agents
- baseline configuration
- startup services

It tests that instance.

The application starts correctly.

Monitoring reports correctly.

The instance can connect to its external dependencies.

The team now has a reference shop whose operating setup is approved.

### Step 2: Create the AMI

The team creates an AMI from the configured EC2 instance.

```text
Configured EC2 instance
        |
        | create image
        v
       AMI
```

The AMI captures the approved starting image.

It becomes the reusable shop model.

### Step 3: Launch New Shops

AWS launches new EC2 instances from that AMI.

```text
                    AMI
          approved operating package
                 /    |    \
                v     v     v
             EC2 1  EC2 2  EC2 3
```

Every new instance begins with the same starting software configuration.

After launch, each EC2 instance becomes its own running machine.

The AMI gave the shops a common birth.

It does not guarantee a common adulthood.

---

## What the AMI Contains

In the boba shop, the AMI captures the systems that every approved location should begin with.

That may include:

- the operating system
- the point-of-sale application
- the approved menu application
- the application runtime
- system libraries
- monitoring agents
- endpoint-security agents
- baseline configuration
- startup services
- expected root-volume layout

Technically, an AMI is also associated with characteristics such as:

- AWS Region
- processor architecture
- root-volume type
- virtualization type
- block-device mappings

The AMI must be compatible with the selected EC2 instance type.

An ARM-based image cannot run unchanged on an incompatible x86 instance type.

The shop model must fit the equipment AWS is asked to provide.

---

## What the AMI Should Not Contain

### Live Business Data

The approved shop model should describe how to open a shop.

It should not contain:

- today's customer orders
- active payment records
- session data
- temporary files
- personal customer information
- production secrets
- a copied production database

An AMI can technically capture files present on the source volumes included in the image.

That does not mean every file belongs there.

Persistent business data should usually live in systems designed for shared, durable state, such as:

- Amazon RDS
- Amazon Aurora
- Amazon DynamoDB
- Amazon S3
- purposefully managed EBS volumes
- other external data stores

Secrets should come from controlled services such as AWS Secrets Manager or AWS Systems Manager Parameter Store.

> **The AMI should reproduce the shop's capability, not yesterday's business activity.**

Bottle the setup.

Do not bottle the customers.

---

## One AMI, Different Environments

The same approved operating package may be used in multiple environments.

QA and production may both need:

- the same operating system
- the same Ruby version
- the same application
- the same monitoring agent
- the same startup services

But they may not need the same machine size, network, permissions, or storage.

```text
                         Same AMI
              OS + runtime + application + agents
                            |
                  +---------+---------+
                  |                   |
                  v                   v
          QA launch template   Production launch template
          smaller instance     larger instance
          QA network            production network
          QA IAM role           production IAM role
          QA security groups    production security groups
          smaller storage       larger storage
                  |                   |
                  v                   v
             QA instance       Production instance
```

The AMI keeps the operating package consistent.

The launch template supplies the environment-specific launch instructions.

This separation is the reason the concepts should not be blended together.

---

## AMI Versus Launch Template

Both participate in creating an EC2 instance.

They answer different questions.

### AMI

The AMI answers:

> **What operating system and software should this machine begin with?**

For the boba company, the AMI is the approved operating package.

It defines what every new shop knows how to run.

### Launch Template

The launch template answers:

> **How should AWS launch this particular kind of machine?**

A launch template can specify:

- AMI ID
- instance type
- key pair
- security groups
- network interfaces
- subnet-related launch settings
- storage configuration
- IAM instance profile
- user data
- tags

In the boba company, the launch template is the opening order for a particular kind of location.

```text
Use this approved shop model.

For QA:
- open a smaller shop
- connect it to the QA network
- assign the QA permissions
- attach smaller storage
- run the QA startup instructions

For production:
- open a larger shop
- connect it to the production network
- assign the production permissions
- attach larger storage
- run the production startup instructions
```

The AMI is one important input inside the launch template.

The launch template combines the image with the remaining instructions AWS needs to create the instance.

> **The AMI defines what the machine knows how to run.**

> **The launch template defines what kind of machine AWS should create, where it belongs, and how it should start.**

---

## Launch Template Versions

Launch templates support numbered versions.

A new version records a change to the launch instructions.

For example:

```text
Production launch-template version 11
- boba AMI v17
- r7g.large
- production security group
- production IAM role
- 50 GB storage
```

After the company approves a patched application image:

```text
Production launch-template version 12
- boba AMI v18
- r7g.large
- production security group
- production IAM role
- 50 GB storage
```

The AMI changed because the approved software package changed.

Later, the production workload needs more capacity:

```text
Production launch-template version 13
- boba AMI v18
- r7g.xlarge
- production security group
- production IAM role
- 50 GB storage
```

The instance type changed because the shop needed a larger body.

The operating package remained the same.

Launch-template versions are immutable.

AWS creates a new version rather than rewriting an existing one.

The company can now say:

> Production uses launch-template version 13.

That is safer than:

> Production uses whatever Bob most recently clicked.

---

## A New AMI Does Not Remodel Existing Shops

The company discovers a security vulnerability.

It patches the reference instance and creates a new AMI.

```text
ami-boba-v1
     |
     | patch and rebuild
     v
ami-boba-v2
```

New instances can launch from `ami-boba-v2`.

Existing instances launched from `ami-boba-v1` do not automatically transform.

The AMI is used when an instance is created.

Changing the image changes future shops.

It does not walk into already-running shops and replace their systems.

The team must decide how to update the existing fleet:

- patch instances in place
- replace them with instances launched from the new AMI
- roll out the image gradually
- use an Auto Scaling instance refresh
- use another deployment or replacement process

> **Changing the shop model changes the next shop, not the shops already serving customers.**

Otherwise, someone updates an AMI, closes the laptop, and assumes production became secure through optimism.

---

## AMIs Are Replaced, Not Edited in Place

A custom AMI represents a particular image.

When the baseline changes, teams generally create a new AMI.

```text
boba-web-v17
boba-web-v18
boba-web-v19
```

Or:

```text
boba-web-2026-07-01
boba-web-2026-07-15
boba-web-2026-07-19
```

The company can test a new image before approving it for production launches.

Older AMIs may later be deprecated, disabled, or deregistered.

The newest image is not automatically the safest image.

It is the newest candidate until it has been tested.

---

## The Golden AMI

Some organizations maintain a **golden AMI**.

This is an approved baseline containing organization-wide requirements such as:

- hardened operating-system settings
- approved patches
- monitoring agents
- endpoint protection
- logging configuration
- compliance tooling
- standard runtime packages

In the boba company, the golden image is the company-approved starter shop before a particular product team installs its menu application.

It answers:

> **What should every new shop contain before the application team adds its own business software?**

Application teams may build application-specific AMIs on top of that baseline.

A golden image reduces inconsistency.

It does not eliminate maintenance.

The image still needs an owner, a rebuild process, security updates, testing, versioning, and cleanup.

Gold still tarnishes when nobody owns the polishing cloth.

---

## Parameter Store Can Point to the Approved AMI

A launch template can reference a Systems Manager parameter instead of hard-coding an AMI ID.

```text
/boba/prod/approved-ami
=
ami-0123456789example
```

The parameter acts like the approved-model card in the operations office.

It tells future launches which AMI currently represents the approved shop model.

When a new AMI passes testing, the parameter can be updated.

Eligible future launches can then resolve to the new image.

The same lifecycle rule remains:

> **Updating the approved-model card changes future launches. It does not rebuild shops already running.**

---

## AMI Versus EBS Snapshot

The two concepts are related, but they are not interchangeable.

An EBS snapshot is a point-in-time copy of an EBS volume.

An AMI is a launchable machine image that tells EC2 how to create and boot an instance.

For EBS-backed AMIs, the AMI references snapshots used to create the required volumes.

In the boba company:

```text
EBS snapshot
=
a preserved copy of one storage room

AMI
=
the approved operating model for opening the shop,
including which storage rooms should be created
```

A snapshot reproduces volume data.

An AMI creates a bootable starting machine.

> **A snapshot copies storage. An AMI launches a shop.**

An AMI may participate in a recovery strategy.

It is not a complete backup strategy for databases, secrets, networking, external services, and all application data.

---

## AMI Versus User Data

The AMI captures the starting software image.

User data supplies instructions that typically run when the instance boots.

The company can bake more into the AMI:

```text
AMI
=
shop arrives with the approved systems already installed
```

This can make startup faster and more predictable.

Or it can configure more at launch:

```text
User data
=
opening-day instructions handed to the setup crew
```

This provides flexibility, but startup becomes more dependent on scripts, repositories, package servers, and external services behaving correctly.

A good design deliberately decides what belongs in the reusable image and what should be supplied at launch.

Do not bake secrets into the image merely because copying them is convenient.

Convenience is how credentials become archaeological artifacts.

---

## Architectural Mapping

```text
Reference EC2 instance
configured and tested
        |
        | create AMI
        v
Amazon Machine Image
OS + software + block-device mappings
        |
        | selected by launch template
        v
Launch Template
AMI + size + network + security + storage + IAM + user data
        |
        | launch
        v
New EC2 Instance
running shop with its own lifecycle
```

The process is:

1. Configure and test a reference EC2 instance.
2. Create an AMI from the approved starting setup.
3. Reference that AMI from a launch template.
4. Add the environment-specific launch parameters.
5. Launch the EC2 instance.
6. Allow the instance to begin its own independent lifecycle.

---

## When to Use a Custom AMI

Use a custom AMI when:

- many EC2 instances need the same starting software configuration
- operating systems require standard hardening
- required packages should already be installed
- startup speed and predictability matter
- Auto Scaling must launch consistent replacements
- application servers should be replaced rather than manually repaired
- security or compliance baselines must be standardized
- disaster recovery needs a known machine image
- teams want to reduce configuration drift
- deployments use immutable-infrastructure practices

AMIs are especially useful when machines are disposable but their starting configuration must be dependable.

---

## When Not to Use It

Do not treat an AMI as the primary answer when:

- the workload does not need EC2
- a container image is the natural packaging format
- Lambda or another managed service removes the server
- the only goal is to back up one EBS volume
- live application data changes constantly
- configuration must be injected dynamically at runtime
- secrets would need to be embedded in the image
- the team has no process for rebuilding and patching stale images
- manual image creation causes more drift than it prevents

An AMI can standardize a machine.

It cannot rescue an undisciplined release process wearing an automation hat.

---

## Painkiller

> **Problem:** Teams need to create multiple EC2 instances with the same operating system and software configuration.
> **Pain:** Building each server manually causes slow launches, missed steps, inconsistent dependencies, and configuration drift.
> **AWS solution:** Use an AMI as the reusable operating package, then use a launch template to supply the environment-specific instructions for creating each EC2 instance.

---

## Knife Cut

> **The AMI standardizes the software. The launch template specializes the machine.**

The AMI says:

```text
Every approved shop starts with:
- this operating system
- this runtime
- this application
- these agents
- these baseline services
```

The launch template says:

```text
For this environment:
- use this machine size
- connect to this network
- apply these permissions
- attach this storage
- run these startup instructions
```

The running EC2 instance is the shop produced from both.

Bottle the shop setup.

Do not bottle the business.

---

## The Masthead

### What Actually Happened

| In the story | In AWS | What it means |
|---|---|---|
| Approved shop operating package | AMI | Reusable image containing the operating system and captured software configuration |
| Running shop | EC2 instance | Independent machine created from the AMI |
| Shop capacity | Instance type | CPU, memory, network, and hardware capacity |
| Environment-specific opening order | Launch template | Versioned launch parameters including AMI, instance type, network, security, IAM, storage, and user data |
| Revised opening order | Launch-template version | Immutable new version of the launch parameters |
| Opening-day instructions | User data | Commands or scripts supplied when the instance starts |
| New approved shop model | New AMI | Replacement image containing updated software or configuration |
| Golden starter shop | Golden AMI | Organization-approved baseline image |
| Storage-room copy | EBS snapshot | Point-in-time copy of EBS volume data |
| Approved-model card | Parameter Store AMI reference | Centrally managed pointer used by eligible future launches |
| Fleet manager | Auto Scaling group | Maintains the desired number of instances |
| Customer dispatcher | Load balancer | Routes requests to healthy instances |

---

## A Note From the Author

The boba analogy deliberately separates the operating package from the physical capacity of the shop.

An AMI does not define every property of the future EC2 instance.

It provides the operating-system and software image needed to set up and boot the instance, together with block-device mappings.

The launch template provides the rest of the launch configuration, such as:

- instance type
- networking
- security groups
- IAM instance profile
- key pair
- storage overrides
- tags
- user data

An AMI is Region-specific, though it can be copied to another Region.

Instances launched from the same AMI are not permanently synchronized.

Updating an AMI does not patch already-running instances.

Updating a Parameter Store value to reference a new AMI affects eligible future launches, not existing machines.

Creating an AMI from a running source instance also requires operational care. Applications may need to flush data, pause writes, or otherwise reach a consistent state before imaging.

Finally, EBS-backed AMIs consume storage through their underlying snapshots. Deregistering an AMI and deleting associated snapshots are separate lifecycle actions.

The story makes repeatability intuitive.

Production image management still requires ownership, testing, security updates, rollout strategy, versioning, and cleanup.

---

## The Last Bite

The first boba shop proved the business could run.

The AMI turns its approved operating setup into a reusable starting image.

The launch template combines that image with the instructions for a particular environment.

Together, they create the next EC2 instance.

> **An EC2 instance is the running shop.**

> **An AMI is the approved operating package.**

> **A launch template is the environment-specific opening order.**

One shop gives you compute.

An AMI gives you repeatability.

A launch template tells AWS which repeatable shop to open.

The fleet comes next.

---

**Next chapter:** _Auto Scaling: When the Boba Trucks Roll In_

The AMI gives every truck the same approved operating package.

The launch template defines the production truck AWS should create.

Auto Scaling decides how many of those trucks should be running when the crowd arrives.
