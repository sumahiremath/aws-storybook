# AWS Fargate: The RV Resort for Your Food Trucks

> Amazon ECS orchestrates your containerized application. AWS Fargate supplies compute capacity without requiring you to manage EC2 instances.

---

## The Pain

In the last chapter, we created a **task definition**.

It became the operating manual for every food truck.

It answered questions such as:

- Which container image should we use?
- How much CPU does it need?
- How much memory does it need?
- Which ports should it expose?
- Where should its logs go?

Everything was ready.

One question remained:

**Where are we going to park all these trucks?**

---

## The Story

Amazon ECS reads the task definition.

It knows how your application should run.

But every task still needs compute capacity.

One option is a fleet of Amazon EC2 instances. With that option, your team manages the instances that form the ECS cluster, including:

- provisioning enough capacity
- patching operating systems
- replacing failed instances
- monitoring the infrastructure
- balancing task demand against available cluster capacity

Your developers wanted to build applications.

Instead, they became campground managers.

---

## Meet AWS Fargate

One day, a fully serviced RV resort opens nearby.

The owner says:

> Bring your operating manual.  
> Bring your food truck.  
> We will assign the campsite and maintain the campground.  
> You focus on serving customers.

That is AWS Fargate.

You declare the task's requirements.

AWS manages the underlying compute infrastructure.

> **Core idea:** Fargate provides serverless compute capacity for containers, so you do not provision or manage the EC2 instances underneath your ECS tasks.

---

## What Fargate Actually Does

AWS Fargate is a **serverless compute engine for containers**.

Serverless does not mean there are no servers.

It means your team does not manage those servers.

With Amazon ECS, you select Fargate capacity and provide a compatible task definition. ECS orchestrates the task, while Fargate provisions the isolated compute environment needed to run it.

---

## How It Works

```text
Container Image
       |
       v
Task Definition
Operating requirements
       |
       v
Amazon ECS
Orchestrate and place the task
       |
       v
AWS Fargate
Provide managed compute capacity
       |
       v
Running ECS Task
```

Each layer answers a different question:

- The container image packages the application.
- The task definition describes how it should run.
- Amazon ECS orchestrates the desired tasks.
- AWS Fargate provides compute capacity for those tasks.

---

## ECS and Fargate Are Partners

Amazon ECS can orchestrate tasks on different capacity options.

With ECS on EC2, your team manages the EC2 cluster capacity.

With ECS on Fargate, AWS manages the underlying compute capacity.

They are not replacements for one another. ECS is the operations manager; Fargate is the managed RV resort.

---

## What Happens When a Truck Breaks Down?

Imagine one payment task stops unexpectedly.

For an ECS service configured with a load balancer, the recovery flow can look like this:

1. The load balancer stops routing new requests to an unhealthy target.
2. The ECS service detects that its running count no longer matches its desired count.
3. ECS requests a replacement task.
4. Fargate provisions capacity for the replacement.
5. The replacement task starts and passes its configured health checks.
6. The load balancer begins routing requests after the new target becomes healthy.

Each service focuses on one job.

Together, when configured correctly, they create a self-healing application platform.

---

## Architectural Mapping

| In the story | In AWS | What it means |
|---|---|---|
| Food truck | Container | Packaged application process |
| Operating manual | Task definition | Declared runtime requirements |
| Operations manager | Amazon ECS | Orchestrates tasks and services |
| RV resort | AWS Fargate | AWS-managed container compute |
| Campsite | Fargate task capacity | Isolated compute allocated to a task |
| Utilities | Requested CPU, memory, networking, and storage | Resources available to the task |

---

## When to Use It

Choose Fargate when:

- the application is packaged in containers
- the team does not want to manage ECS container instances
- operational simplicity matters more than host-level control
- workloads benefit from task-level resource allocation
- scaling should not require provisioning EC2 capacity first

---

## When Not to Use It

Consider ECS on EC2 or another compute option when:

- the workload requires host-level access or customization
- specialized instance capabilities are required
- existing EC2 capacity should be shared across many tasks
- a large, steady workload makes EC2 capacity economically preferable
- the task requires a configuration unsupported by Fargate

---

## Painkiller

> **Problem:** Containerized applications need compute capacity, but managing container hosts creates additional operational work.
> **Pain:** Teams must provision, patch, monitor, scale, and replace the EC2 instances underneath their tasks.
> **AWS solution:** Use AWS Fargate to run ECS tasks on AWS-managed compute capacity.

---

## Knife Cut

> **Amazon ECS orchestrates the food trucks. AWS Fargate manages the campground underneath them.**

---

## The Masthead

### What Actually Just Happened

| In the story | In AWS | What it actually means |
|---|---|---|
| Food truck | Container image | Packages the application |
| Operating manual | Task definition | Declares task runtime requirements |
| Operations manager | Amazon ECS | Maintains and places tasks |
| Managed RV resort | AWS Fargate | Supplies compute without customer-managed EC2 instances |
| Serviced campsite | Running Fargate task | Allocated task-level compute environment |

---

## A Note From the Author

The RV resort makes capacity sound like a visible campsite selected after ECS makes every decision. In practice, ECS and Fargate integrate to place and run tasks, and AWS does not expose the underlying server as a customer-managed host.

Fargate removes EC2 host management, not application operations. Your team still chooses task resources, networking, IAM roles, logging, secrets, health checks, deployment settings, scaling policies, and durable storage. Fargate also does not scale an ECS service by itself; the service's desired count or Application Auto Scaling policy controls how many tasks should run.

Fargate pricing is based on requested task resources and runtime rather than direct management of an EC2 fleet. Workload shape, supported configurations, and pricing should be evaluated before choosing it.

- [AWS Fargate for Amazon ECS](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/AWS_Fargate.html)
- [Fargate task definitions](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/task_definition_parameters.html)

---

## The Last Bite

Container images package your application.

Task definitions describe how it should run.

Amazon ECS orchestrates the tasks.

AWS Fargate supplies the compute capacity without asking your team to manage the EC2 campground.

You bring the food truck.

AWS maintains the resort.

---

**Next chapter:** *Amazon ECR: The Warehouse of Approved Food Truck Blueprints*

Fargate gives each truck a place to run.

Amazon ECR gives the fleet an approved place to store and retrieve its container images.
