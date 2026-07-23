# EC2: The Boba Shop That Stays Open

> Amazon EC2 gives your application a provisioned virtual machine that remains available while it is running, whether customers are lining up or nobody walks through the door.

## The Pain

Your application needs somewhere to run.

Not for one quick job.
Not only when a file arrives.

It may need to:
- serve web requests all day
- maintain long-running processes
- run software with operating-system dependencies
- keep connections open
- host an application that was built for a traditional server
- give engineers control over the machine underneath the code

You could buy a physical server.

Then you would need to choose the hardware, wait for delivery, install it, power it, cool it, repair it, and eventually replace it.

That is a lot of building ownership for a team that merely wants somewhere to run software.

The application needs a shop.

It does not need to pour the concrete.

---

## The Story

### The Boba Shop That Stays Open

A boba shop opens at 9 a.m.

Before the first customer arrives, the lights are on.

The tea is brewing.
The refrigerators are cold.
The blenders are plugged in.
The point-of-sale system is waiting.

At noon, the line curls around the counter.
At 3 p.m., nobody is there.

The shop does not disappear between customers.
Its space, machines, and staff capacity remain available.

That readiness has value.

It also has a cost.

The shop pays for the space while customers are buying drinks.

It pays for the space while one person slowly studies the menu.
It pays for the space while the chairs sit empty.

The shop is not created for each order and destroyed afterward.

It is provisioned before the work arrives.

That is Amazon EC2.

---

## The Wrong Way

The owner sees only three customers during a quiet afternoon and declares:

> “This shop is too powerful. Remove half the refrigerator and one-third of the counter.”

At 5 p.m., a school bus pulls into the parking lot.

Now the shop cannot keep up.

The opposite mistake is just as expensive.

The owner leases a flagship store with twelve blenders, six counters, and an industrial freezer because someday the business might become popular.

Most days, the equipment hums for nobody.

Choosing EC2 is not merely deciding to have a server.

You must choose how much server the workload needs.

Too small, and the application struggles.

Too large, and you pay for an impressive amount of unused silence.

---

## Meet Amazon EC2

Amazon Elastic Compute Cloud provides virtual servers called ****instances****.

You choose a machine configuration, launch it inside AWS, install or run your software, and manage it much as you would manage another server.

> ****Core idea:**** EC2 gives you provisioned compute with control over the operating system and the software running on it.

AWS manages the physical data center and the underlying hardware.

You manage what happens inside the virtual machine.

That includes responsibilities such as:

- selecting an operating system
- installing application software
- applying guest operating-system patches
- configuring the application
- controlling access
- monitoring resource use
- choosing storage
- deciding when the instance should run
- replacing or scaling the instance when the workload changes

AWS removed the building.

It did not remove shop management.

---

## How It Works

### The Boba Shop

#### EC2 Instance

An EC2 instance is a virtual server running in AWS.

It provides resources such as:

- virtual CPUs
- memory
- network capacity
- storage options

While the instance is in the `running` state, that compute capacity is provisioned for your workload.

The application may be busy.

It may be idle.

It may be completely stuck because someone deployed a bug at 4:57 p.m.

The instance still exists.

EC2 usage is billed while the instance is running, even when it is idle. A stopped instance does not incur instance-usage charges, although attached resources such as EBS volumes can continue to incur charges. ([AWS Documentation](https://docs.aws.amazon.com/us_en/AWSEC2/latest/UserGuide/ec2-instance-lifecycle.html?utm_source=chatgpt.com "Amazon EC2 instance state changes - Amazon Elastic Compute Cloud"))

This is why “always-on” is useful as a mental model, but not a literal requirement.

An EC2 instance can be stopped.

While it is running, however, the shop remains provisioned before the next customer arrives.

---

### The Size of the Shop

#### Instance Type

Not every boba shop needs the same equipment.

A kiosk in a quiet office building might need:

- one counter
- one blender
- a small refrigerator

A downtown flagship location might need:

- several counters
- industrial tea brewers
- more refrigeration
- faster order processing
- a larger network connection

EC2 instance types provide different combinations of:

- CPU
- memory
- networking
- storage capabilities
- specialized hardware such as GPUs

AWS groups instance types into families optimized for different kinds of work.

A compute-intensive workload may care most about CPU.

An in-memory cache may need large amounts of RAM.

A machine-learning workload may need GPU acceleration.

A high-throughput service may care about network performance.

The correct instance type is not the largest one you can afford.

It is the one whose resource shape matches the workload.

```text

Small shop

    |

    | workload grows

    v

Larger shop

```

Moving one workload to a more powerful instance type is ****vertical scaling****.

You are making one shop larger.

You are not opening more shops.

Vertical scaling moves the business into a larger shop. But eventually, one giant shop becomes expensive, fragile, or physically limited. To serve a much larger crowd, the company needs a repeatable way to open several identical shops. That begins with an AMI.

---

### The Store’s Starting Setup

#### Amazon Machine Image

Before the boba shop opens, it needs a starting setup:

- an operating system
- installed software
- configured tools
- storage mappings
- startup instructions

An Amazon Machine Image, or AMI, provides the information EC2 needs to launch an instance.

The AMI is the approved starting model for the shop.

It is not the live business happening inside it.

It does not represent today’s line of customers, current orders, or every changing piece of business data.

That distinction deserves its own story.

For now, remember:

> The EC2 instance is the running shop. The AMI is the starting model used to create it.

---

### The Front Door

#### Security Group

A boba shop needs rules about who may enter and which doors may be used.

Customers may enter through the front door.

Employees may use a staff entrance.

The storage room should not be open to everyone wandering past the building.

A security group acts as a virtual firewall for an EC2 instance.

Its rules control allowed inbound and outbound network traffic.

For example:

- allow HTTPS traffic from customers
- allow administrative access only from a trusted network
- allow the application to connect to a database
- deny access that has not been explicitly permitted by a rule

Security groups are stateful. When traffic is allowed in one direction, the corresponding response traffic is allowed back through the tracked connection. ([AWS Documentation](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-security-groups.html?utm_source=chatgpt.com "Amazon EC2 security groups for your EC2 instances - Amazon Elastic Compute Cloud"))

A security group does not make an application secure by itself.

It controls network doors.

You still need secure software, appropriate IAM permissions, operating-system hardening, patching, secrets management, and monitoring.

A good lock does not repair a reckless shop manager.

---

### The Permanent Storage Room

#### Amazon EBS

The shop closes overnight, but the inventory records and equipment configuration should still exist in the morning.

Amazon Elastic Block Store provides persistent block storage that can be attached to EC2 instances.

An EBS volume behaves roughly like a virtual disk.

It can contain:

- the operating system
- application files
- logs
- database files
- other data requiring persistent block storage

For EBS-backed instances, stopping the EC2 instance releases the compute capacity while the EBS volumes can remain. The instance can later be started again using its persistent storage. EBS charges can continue while the instance is stopped. ([AWS Documentation](https://docs.aws.amazon.com/us_en/AWSEC2/latest/UserGuide/ec2-instance-lifecycle.html?utm_source=chatgpt.com "Amazon EC2 instance state changes - Amazon Elastic Compute Cloud"))

```text

EC2 instance stops

        |

        +---- Compute released

        |

        +---- EBS volume remains

```

Stopping the shop turns off the working space.

It does not necessarily throw away the storage room.

---

### One Temporary Storage Caveat

Some EC2 instance types also provide temporary local storage called ****instance store****. It can be useful for caches and scratch data, but it does not survive events such as stopping or terminating the instance. Treat it like ice prepared for today’s shift, not the shop’s permanent storage room.

> ****EBS is the storage room. Instance store is the working ice on the counter.****

---

### Closing Without Demolishing

#### Stop

An EBS-backed EC2 instance can be stopped.

When stopped:

- the operating system shuts down
- the compute capacity is released
- instance-usage billing stops
- attached EBS volumes remain and may continue to incur storage charges
- data held only in RAM is lost
- instance-store data is lost
- the instance can be started again later

When the instance starts again, AWS generally places it on underlying host capacity and boots it using its persistent EBS root volume.

Its automatically assigned public IPv4 address can change. Pointing a domain directly at that temporary address can therefore break when the shop reopens. Stable designs commonly use an Elastic IP, a load balancer, or DNS that points to a stable endpoint. ([AWS Documentation](https://docs.aws.amazon.com/us_en/AWSEC2/latest/UserGuide/ec2-instance-lifecycle.html?utm_source=chatgpt.com "Amazon EC2 instance state changes - Amazon Elastic Compute Cloud"))

Stopping is closing the shop while preserving its durable setup.

It is not serving customers.

It is also not demolishing the business.

---

### Demolishing the Shop

#### Terminate

Termination is different.

When you terminate an EC2 instance, the instance is permanently deleted.

It cannot be restarted.

The root EBS volume is deleted by default unless its deletion behavior has been configured differently. Other attached volumes may have their own deletion settings.

Termination is not closing for the night.

It is surrendering the lease and removing the shop.

Before termination, you must know which data exists elsewhere and which storage resources should survive.

Cloud resources are easy to delete.

That does not make their data easy to reconstruct.

---

## Architectural Mapping

```text

Customers

    |

    v

Security group

Allowed network entrances

    |

    v

EC2 instance

Provisioned virtual server

    |

    +---- CPU and memory

    |

    +---- Operating system

    |

    +---- Application

    |

    +---- EBS persistent storage

```

A request reaches the EC2 instance only when the surrounding network path and security rules allow it.

The operating system receives the traffic.

The application performs the work.

The instance consumes CPU, memory, network, and storage resources while doing so.

AWS operates the physical infrastructure beneath the instance.

The customer still operates the workload inside it.

---

## What AWS Manages and What You Manage

|Responsibility|Boba story|Owner|

|---|---|---|

|Physical data center|The land and building infrastructure|AWS|

|Physical servers|The underlying building structure and utilities|AWS|

|Virtualization layer|Dividing the building into rentable shops|AWS|

|EC2 instance selection|Choosing the size of the shop|Customer|

|Guest operating system|Managing the shop’s internal systems|Customer|

|OS and application patches|Maintaining the equipment and procedures|Customer|

|Application deployment|Installing the boba operation|Customer|

|Security-group rules|Deciding which network doors are open|Customer|

|Data protection|Protecting recipes, records, and inventory|Customer|

|Monitoring and scaling|Watching demand and adding capacity|Customer|

EC2 gives you control because some applications need control.

That same control creates work.

---

## When to Use It

Use EC2 when the workload benefits from one or more of these characteristics:

- it runs continuously or for long periods
- it needs control over the operating system
- it requires custom system packages, agents, or kernel-level capabilities
- it uses software designed to run on a traditional server
- it maintains long-running processes or connections
- it needs a specialized instance type, GPU, or hardware capability
- it has predictable utilization that can justify provisioned capacity
- the team is prepared to patch, secure, monitor, and operate servers
- another managed compute model does not provide the required control

EC2 is not outdated merely because serverless services exist.

A permanent shop is sensible when customers arrive throughout the day and the business needs control over the entire operation.

---

## When Not to Use It

Consider another compute model when:

- code runs only in response to occasional events
- the workload is short-lived and bounded
- the team does not need operating-system control
- most of the server would sit idle
- the application already fits a managed container platform
- a managed AWS service can remove substantial operational work
- the team cannot reliably patch, secure, and monitor the instance
- rapid horizontal scaling is required but the application remains tightly bound to one machine
- the application stores critical state only on the local instance

Lambda may fit event-driven, bounded work.

ECS or EKS may fit containerized workloads.

Fargate may fit containers when the team does not want to manage the underlying EC2 fleet.

A managed service may be better when running the underlying software is not part of your product’s value.

Do not lease a full boba shop to make one drink every Thursday.

---

## Painkiller

> **Problem:** An application needs flexible compute without the delay and expense of purchasing physical servers.
> **Pain:** Owning hardware creates procurement, data-center, maintenance, and replacement work before the application can even run.
> **AWS solution:** Use Amazon EC2 to launch resizable virtual servers while retaining control over the operating system, software, networking, and storage configuration.

---

## Knife Cut

> ****Lambda appears when work arrives. EC2 keeps the shop available before the work arrives.****

Lambda supplies bounded compute in response to an invocation.

EC2 supplies a provisioned machine that remains available while it is running.

One is summoned work.

The other is rented space.

Neither is universally better.

The workload chooses.

---

## The Masthead

### What Actually Just Happened

| In the story                  | In AWS                  | What it actually means                                                                          |
| ----------------------------- | ----------------------- | ----------------------------------------------------------------------------------------------- |
| Permanent boba shop           | EC2 instance            | A provisioned virtual server                                                                    |
| Shop size and equipment       | Instance type           | A selected combination of CPU, memory, network, storage, and specialized hardware               |
| Approved starting store model | AMI                     | The information required to launch an instance with an operating system and configured software |
| Front-door rules              | Security group          | Stateful rules controlling allowed inbound and outbound network traffic                         |
| Storage room                  | EBS volume              | Persistent block storage independent of the instance’s active compute lifecycle                 |
| Closing overnight             | Stop instance           | Release compute while preserving attached EBS storage                                           |
| Reopening                     | Start instance          | Provision compute again and boot from the persistent root volume                                |
| Demolishing the store         | Terminate instance      | Permanently delete the EC2 instance                                                             |
| Moving into a larger store    | Vertical scaling        | Change to an instance type with more capacity                                                   |
| Shop maintenance              | Customer responsibility | Patch, configure, secure, monitor, and operate the guest system and application                 |

---

## A Note From the Author

The boba shop is described as staying open because EC2 represents provisioned compute.

That does not mean an EC2 instance must run every hour of every day.

An EBS-backed instance can be stopped and started. While stopped, instance-usage billing stops, but attached EBS storage and some other retained resources may still generate charges. ([AWS Documentation](https://docs.aws.amazon.com/us_en/AWSEC2/latest/UserGuide/ec2-instance-lifecycle.html?utm_source=chatgpt.com "Amazon EC2 instance state changes - Amazon Elastic Compute Cloud"))

A real shop retains the same physical building after closing. A stopped EC2 instance usually starts on underlying host capacity chosen by AWS. The durable abstraction is the virtual instance and its EBS-backed state, not a promise that the same physical machine waits underneath it.

The storage analogy also has limits.

EBS is durable block storage, but durability does not eliminate the need for backups, snapshots, replication, access control, and recovery planning.

Security groups are stateful virtual firewalls controlling network traffic. They do not inspect application intent, patch vulnerable software, or replace the rest of a security architecture. ([AWS Documentation](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-security-groups.html?utm_source=chatgpt.com "Amazon EC2 security groups for your EC2 instances - Amazon Elastic Compute Cloud"))

Finally, AWS operates the infrastructure below the virtualization boundary.

You still own the operating system and application above it.

The shop is rented.

The shift is yours.

---

## The Last Bite

EC2 gives your application a shop of its own.

You choose its size.

You decide what runs inside it.

You open the network doors.

You maintain the equipment.

AWS provides the underlying building, but the shop remains ready because you chose provisioned compute.

> ****EC2 does not wait for an event to create the worker. It keeps the workplace available for whatever arrives next.****

---

****Next chapter:**** __AMI: Bottle the Blueprint, Not the Business__

One boba shop gives you compute.

An AMI gives you repeatability.

Auto Scaling will turn that repeatability into a fleet.
