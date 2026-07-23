# Auto Scaling: The Fleet Manager Who Opens Boba Shops Before the Line Gets Too Long

> Amazon EC2 Auto Scaling automatically adjusts the number of EC2 instances so your application has the capacity it needs without keeping unnecessary servers running.

---

## The Pain

Yesterday, one boba shop was enough.

Today, a local influencer posted a video.

A line wrapped around the block.

Customers left because the wait was too long.

A week later, the excitement faded.

Now the giant shop sat mostly empty.

Building one enormous shop for the busiest day wastes money.

Keeping one tiny shop for every day loses customers.

The business needs capacity that grows and shrinks with demand.

---

## The Story

The owner already solved two important problems.

Every new shop uses the same approved operating package.
Every production shop launches from the same production launch template.

Opening another shop is easy.

Knowing **when** to open another shop is not.

So the owner hires a fleet manager.

The fleet manager follows simple rules.

```text
Always keep at least 2 shops open.

If the lines stay long,
open another shop.

If the shops stay quiet for a while,
close one.

Never go below two shops.
```

The fleet manager never installs software.
The fleet manager never chooses shop size.
The fleet manager never decides the menu.

Those decisions belong to the AMI and Launch Template.

The fleet manager answers only one question:

> **How many identical shops should be operating right now?**

That is Amazon EC2 Auto Scaling.

---

## Meet Auto Scaling

An Auto Scaling Group (ASG) manages a fleet of EC2 instances.

It launches new instances when more capacity is needed.

It removes instances when demand falls.

It also replaces unhealthy instances automatically.

> **Core idea:** Auto Scaling maintains the desired number of healthy EC2 instances.

---

## The Boba Mapping

| Boba Company | AWS | Meaning |
|---|---|---|
| Fleet manager | Auto Scaling Group | Maintains the fleet |
| Boba shop | EC2 instance | Compute capacity |
| Approved operating package | AMI | Standard software |
| Opening instructions | Launch Template | Launch configuration |
| Customer line | CloudWatch metrics | Measured demand |
| Open another shop | Scale out | Add capacity |
| Close a shop | Scale in | Remove capacity |

---

## How It Works

### Step 1: Watch Demand

The fleet manager watches the lines.

AWS watches metrics such as CPU utilization, request count, or custom CloudWatch metrics.

### Step 2: Decide

The fleet manager is not counting exactly when to send two more shops.

The goal is simpler: keep each shop about equally busy.

If every shop becomes too busy, another joins the fleet.

If the shops become underused, one quietly closes.

AWS calls this **target tracking scaling**. Instead of manually defining "add two instances when CPU reaches 70 percent," a target tracking policy asks Auto Scaling to keep a metric—such as average CPU utilization—near a chosen target. Auto Scaling adjusts capacity within the group's minimum and maximum limits to move the metric toward that target.

### Step 3: Launch

```text
CloudWatch Metrics
        |
        v
 Auto Scaling Group
        |
        v
 Launch Template
        |
        v
       AMI
        |
        v
   New EC2 Instance
```

The new shop opens from the same operating package and launch configuration as every other production shop.

---

## Scale Out vs. Scale In

When demand increases:

```text
2 Shops
   |
Rush begins
   |
   v
4 Shops
```

When demand falls:

```text
4 Shops
   |
Quiet evening
   |
   v
2 Shops
```

Elasticity means changing capacity to match demand instead of guessing months in advance.

---

## Four Ways to Plan the Fleet

Target tracking is the simplest default mental model, but Auto Scaling can respond to demand in several ways.

| Strategy | Boba story |
|---|---|
| Step scaling | "If the line gets this long, send two shops." |
| Target tracking | "Keep each shop about equally busy." |
| Scheduled scaling | "Open more shops before the football game." |
| Predictive scaling | "The forecast already knows the Friday concert crowd is coming." |

### The Concert Calendar

The fleet manager notices a pattern.

Every Friday at 7 PM, the concert ends.

Every Saturday morning, the farmers market opens.

Instead of waiting until customers are already standing in line, the manager prepares more shops before the rush begins.

AWS calls this **predictive scaling**. It uses historical load data to forecast future demand and can increase capacity ahead of expected traffic instead of reacting only after utilization rises.

---

## High Availability

Scaling changes capacity. High availability keeps the fleet useful when individual shops or locations fail.

### Health Checks

Suppose one shop loses power.

Customers should not be sent there.

If a managed instance becomes unhealthy, Auto Scaling can terminate it and launch a replacement.

The goal is not simply having four instances.

The goal is having four **healthy** instances.

### Do Not Put Every Shop on One Street

The company owns six shops.

Putting all six on the same street seems convenient.

Until that street loses power.

Instead, the fleet manager spreads shops across several neighborhoods.

If one neighborhood has a localized failure, shops in the others can continue serving customers.

AWS calls those neighborhoods **Availability Zones**. An Auto Scaling group can distribute instances across multiple Availability Zones so one localized infrastructure failure does not remove the entire fleet.

### Do Not Close a Shop Mid-Order

A shop should not close while someone is still waiting for a drink.

The fleet manager first stops sending new customers there.

The remaining orders finish.

Only then does the shop close.

In AWS, graceful scale-in requires coordination. A load balancer can stop routing new requests and allow in-flight requests time to finish, while an Auto Scaling lifecycle hook can pause termination so the application performs cleanup before the instance is removed.

---

## Desired, Minimum, and Maximum Capacity

The fleet manager follows guardrails.

```text
Minimum: 2 shops
Desired: 4 shops
Maximum: 10 shops
```

The fleet manager never drops below the minimum.

It tries to maintain the desired capacity.

It never exceeds the configured maximum.

These limits prevent both underprovisioning and runaway scaling.

---

## Wrong Way

Every Friday afternoon the owner manually opens another shop.

Every Sunday night the owner manually closes it.

Sometimes the owner forgets.

Sometimes the rush comes early.

Sometimes the rush never comes.

Human schedules rarely match customer demand.

Well-designed scaling policies can react continuously, follow a known schedule, or prepare for forecast demand.

---

## Architectural Mapping

```text
              CloudWatch
                 Metrics
                    |
                    v
          Auto Scaling Group
                    |
      +-------------+-------------+
      |                           |
 Launch Instance            Terminate Instance
      |                           |
      +-------------+-------------+
                    |
             Launch Template
                    |
                   AMI
                    |
               EC2 Instance
```

---

## When to Use It

Use Auto Scaling when:

- demand changes throughout the day
- availability is important
- failed instances should be replaced automatically
- applications are stateless or can externalize state
- predictable manual provisioning wastes money

---

## When Not to Use It

Auto Scaling is a poor fit when:

- the application depends on local machine state
- every server is manually customized
- scaling requires manual data migration
- the workload cannot safely add or remove instances

Applications should be designed so any healthy shop can serve the next customer.

---

## Painkiller

> **Problem:** Customer demand changes over time, making a fixed number of servers either too expensive or too small.
> **Pain:** Manual scaling is slow, error-prone, and often happens after customers are already waiting.
> **AWS solution:** Auto Scaling automatically launches, removes, and replaces EC2 instances to maintain the desired healthy capacity.

---

## Knife Cut

> **AMI defines what every shop knows. Launch Templates define how AWS builds each shop. Auto Scaling decides how many shops should exist.**

---

## The Masthead

### What Actually Just Happened

| In the story | AWS | Meaning |
|---|---|---|
| Fleet manager | Auto Scaling Group | Maintains desired fleet size |
| Shop | EC2 instance | Compute resource |
| Approved operating package | AMI | Standard software image |
| Opening order | Launch Template | Launch configuration |
| Customer line | CloudWatch metrics | Demand signal |
| Open another shop | Scale out | Add instances |
| Close a shop | Scale in | Remove instances |
| Replace broken shop | Health replacement | Self-healing fleet |
| Equally busy shops | Target tracking policy | Maintains a metric near a target |
| Several neighborhoods | Multiple Availability Zones | Distributes capacity across failure boundaries |
| Concert forecast | Predictive scaling | Adds capacity ahead of forecast demand |

---

## A Note From the Author

The fleet manager makes scaling sound immediate, but real instances need time to launch, pass health checks, warm up, serve traffic, and terminate safely. Target tracking moves a metric toward a target; it does not guarantee that the metric will remain at that exact value.

High availability also requires more than choosing multiple Availability Zones. The application, load balancer, data layer, health checks, and minimum capacity must all be designed so the remaining fleet can continue serving traffic when one location fails.

- [Target tracking scaling policies](https://docs.aws.amazon.com/autoscaling/ec2/userguide/as-scaling-target-tracking.html)
- [Predictive scaling policies](https://docs.aws.amazon.com/autoscaling/ec2/userguide/ec2-auto-scaling-predictive-scaling.html)
- [Lifecycle hooks](https://docs.aws.amazon.com/autoscaling/ec2/userguide/lifecycle-hooks.html)

---

## The Last Bite

The owner no longer guesses how many shops to keep open.

The fleet manager watches demand continuously.

When the crowd grows, another identical shop opens.

When the crowd disappears, extra shops quietly close.

Every shop is built the same way.

Only the number of shops changes.

That is the promise of Auto Scaling.

---

**Next chapter:** *Elastic Load Balancing: The Host Who Seats Every Customer*

Auto Scaling decides **how many** shops should exist.

Elastic Load Balancing decides **which healthy shop** serves each customer.
