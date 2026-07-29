---
description: "Deployment strategies trade speed, cost, capacity, mixed-version complexity, and blast radius. No strategy removes the need for compatibility and health evidence."
tags:
  - "aws"
  - "deployment"
  - "devops"
  - "comparison"
---

# AWS Deployment Strategies: How Many Stores Change Tonight?

> Deployment strategies trade speed, cost, capacity, mixed-version complexity, and blast radius. No strategy removes the need for compatibility and health evidence.

## The Business Goal

Leena wanted release 4.7.2 everywhere before the television campaign began.

Nia wanted one pilot store.

Dev warned that the EC2 fleet could not take half its instances out of service during dinner. Marco warned that old and new application versions would process the same queue during a gradual rollout. Finance warned that a complete parallel environment doubled capacity temporarily.

Everyone wanted “the safest strategy.”

They meant different things by safe.

---

## The Story

Nia placed two hundred store tokens on a map.

For **all-at-once**, she flipped every token together. Fast, cheap in elapsed time, and one defect away from total impact.

For **rolling**, she changed one batch of store tokens while the rest stayed open. Capacity fell unless extra stores were added. Old and new versions coexisted.

For **canary**, she changed a tiny pilot group and waited for evidence before continuing.

For **linear**, she moved a fixed share at regular intervals: ten percent now, another ten percent later.

For **blue/green**, she built a complete green fleet beside the blue fleet, tested it, and moved traffic between them.

The strategy meeting stopped asking which name sounded safest.

It asked which failure the system could observe and survive.

---

## The Wrong Way

Canary is not “deploy to one instance and hope.” It requires:

- Representative traffic
- Version-specific telemetry
- A long enough observation window
- A threshold that catches harm
- An automatic or practiced return path

Blue/green is not a time machine. Both environments may share databases, queues, caches, and external APIs. A new version can still perform irreversible work.

Rolling is not the same as linear traffic shifting. Rolling commonly replaces compute capacity in batches. Linear commonly describes traffic moving by a fixed percentage over intervals, such as Lambda alias traffic.

---

## Meet the Strategies

> **Core idea:** Choose according to acceptable blast radius, available capacity, compatibility during coexistence, and the speed and reliability of detection.

### All-at-Once

The new version replaces the old version everywhere in one operation or traffic shift.

**Strengths:** Fast; no long mixed-version period; simple.

**Costs:** Largest blast radius; platform may experience reduced availability or downtime depending on target; rollback begins only after broad exposure.

Good for low-risk, non-production, or easily reversible changes when interruption is acceptable.

### Rolling

Instances or tasks update in batches while the rest serve traffic.

**Strengths:** Lower infrastructure cost than a full parallel fleet; reduced blast radius by batch.

**Costs:** Temporary capacity reduction unless extra capacity is launched; old and new versions coexist; deployment takes longer.

The application, schema, cache, and message contracts must support coexistence.

### Rolling with Additional Capacity

New capacity is added before old capacity is taken out, preserving service capacity through batches.

**Strengths:** Avoids the capacity dip.

**Costs:** Temporary additional cost and quota needs.

Elastic Beanstalk calls this **rolling with an additional batch**.

### Canary

A small percentage or small target group receives the new version first. If health remains acceptable, the rest follows.

**Strengths:** Small initial blast radius; real production evidence.

**Costs:** Requires strong observability; low traffic can make percentages noisy; rare failures may not appear during the canary window.

Lambda weighted routing is probabilistic, so a low-volume canary may not receive exactly the configured percentage.

### Linear

Traffic moves in equal increments over a schedule.

**Strengths:** Gradually increases exposure with repeated observation windows.

**Costs:** Longer mixed-version operation; more opportunities for incompatible state interactions.

AWS CodeDeploy and AppConfig provide predefined and custom gradual strategies appropriate to their targets.

### Blue/Green

The current environment, blue, remains while a new green environment is deployed and validated. Routing moves to green.

**Strengths:** Strong environment isolation; fast traffic return if blue remains healthy; green can be tested before production exposure.

**Costs:** Temporary duplicate capacity; state compatibility remains; environment creation and warm-up take time.

For ECS through CodeDeploy, blue and green are task sets. For EC2 they are instance groups. For an Elastic Beanstalk environment swap, DNS and environment behavior introduce their own considerations. The exact mechanism matters.

### Immutable

An immutable deployment creates fresh compute with the new version rather than updating existing hosts in place.

It resembles blue/green in resource isolation, but the product's routing and cleanup semantics define whether it is a full blue/green release.

Elastic Beanstalk immutable deployments launch a new group of instances and keep the original group untouched until health succeeds.

---

## How It Works

### How to Choose

Ask in this order:

1. **Can old and new versions coexist?** If not, gradual strategies may be dangerous without a compatibility release first.
2. **Can the system detect harm quickly?** Weak alarms make a canary ceremonial.
3. **Can the platform afford extra capacity?** Blue/green and additional batches need headroom and quotas.
4. **Can rollback restore the service contract?** Shared state and migrations may prevent it.
5. **How much impact is acceptable before detection?** That determines initial exposure.
6. **How representative is early traffic?** A pilot Region or customer cohort may be biased.

### Alarms and Bake Time

Error rate, latency, throttles, health checks, and business metrics should be segmented by version where possible.

A **bake time** waits after exposure so delayed failures can appear before the deployment is declared complete. Too short misses slow failures; too long slows recovery and delivery.

### Backward-Compatible Change

Gradual releases often require:

- Readers that understand old and new data
- Writers that do not break old readers
- Idempotent handlers
- Stable event contracts
- Expand-and-contract schema migrations
- Cache-key or session compatibility

Deployment strategy cannot compensate for an application that only works when every component changes simultaneously.

---

## Architectural Mapping

|Strategy|Initial blast radius|Extra capacity|Mixed versions|Typical strength|
|---|---:|---:|---:|---|
|All-at-once|High|Low|Short or none|Speed|
|Rolling|Batch-sized|Low to medium|Yes|Fleet replacement without full duplicate|
|Canary|Small|Low to medium|Yes|Early production evidence|
|Linear|Gradually increasing|Low to medium|Yes|Controlled traffic growth|
|Blue/green|Routing-sized|High|Yes during shift|Isolation and fast traffic return|
|Immutable|New fleet|High|During validation|Avoid changing hosts in place|

---

## When to Use It

Use canary or linear shifts when version-level health is strong and early traffic is representative. Use rolling when fleet capacity and coexistence are understood. Use blue/green when isolation and fast routing reversal justify extra capacity.

Use all-at-once for environments or changes where broad immediate exposure is acceptable.

## When Not to Use It

Do not choose a strategy by service diagram alone. Do not use slow traffic shifting for versions that cannot coexist. Do not keep blue alive as a supposed rollback after it has become incompatible with shared state.

---

## Painkiller

> **Problem:** The team wants a universally safest deployment strategy.  
> **Pain:** Each strategy hides different costs in blast radius, capacity, coexistence, and detection.  
> **AWS solution:** Select all-at-once, rolling, canary, linear, blue/green, or immutable behavior according to target mechanics, compatibility, health evidence, and return path.

---

## Knife Cut

> **Canary limits first exposure. Blue/green isolates environments. Rolling limits each replacement batch. None reverses external side effects.**

---

## The Masthead

### What Actually Just Happened

|In the story|In deployment|What it actually means|
|---|---|---|
|Every store tonight|All-at-once|Immediate broad deployment or traffic shift|
|Region by region|Rolling|Replace compute in batches|
|One pilot store|Canary|Small initial production exposure|
|Ten percent every interval|Linear|Fixed incremental traffic shift|
|Twin restaurant fleets|Blue/green|Old and new environments with routed traffic|
|Brand-new equipment set|Immutable|Fresh compute rather than in-place change|
|Observation period|Bake time|Wait for alarms before declaring success|

---

## A Note From the Author

AWS product names for similar strategies do not guarantee identical mechanics. Always identify what is being shifted—instances, tasks, Lambda invocations, API mappings, configuration targets, or load-balancer traffic.

Connect alarms to deployment controls before the release begins. An alarm that only pages a human after a five-minute all-at-once outage is observability, but not automatic deployment protection.

- [Lambda weighted aliases](https://docs.aws.amazon.com/lambda/latest/dg/configuring-alias-routing.html)
- [AWS AppConfig deployment strategies](https://docs.aws.amazon.com/appconfig/latest/userguide/appconfig-creating-deployment-strategy.html)
- [Elastic Beanstalk deployment policies](https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/using-features.rolling-version-deploy.html)

---

## The Last Bite

Nia chose a canary followed by linear traffic growth for the Lambda checkout path and a capacity-preserving rolling deployment for the store servers.

One smaller franchise team looked at the infrastructure, load balancers, Auto Scaling groups, health reporting, and platform updates and asked:

> “Can AWS run more of this platform for us?”

> **Sometimes the best deployment decision is to choose a more managed operating boundary.**

---

**Next chapter:** *[AWS Elastic Beanstalk: The Turnkey Franchise](13-the-turnkey-franchise.md)*

Elastic Beanstalk offers the team a turnkey franchise platform—without taking ownership of the application away.
