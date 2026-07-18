# Shared Responsibility: The Restaurant Agreement


## The Pain

You decide to open a restaurant.

You have two choices.

### Option 1: Build Everything Yourself

Buy land.
Construct the building.
Install plumbing.
Purchase ovens.
Hire electricians.
Maintain refrigerators.
Repair broken stoves.
Clean grease traps.
Replace broken equipment.

Then...

finally...

cook food.

Most restaurants don't actually want to become construction companies.

---

### Option 2: Rent a Professional Kitchen

Instead, imagine a company says:

> "We'll own the building.
> We'll maintain every oven.
> We'll repair equipment.
> We'll clean the ventilation.
> We'll make sure the lights stay on."

Now your job becomes...

Cooking amazing food.

---

## AWS Is That Professional Kitchen

AWS says:
>"We'll manage the infrastructure."

You say:
>"I'll build my application."

Both sides have responsibilities.

Neither can do the other's job.

---

## What AWS Owns

AWS worries about:

- The building
- Electricity
- Plumbing
- Fire protection
- Kitchen equipment
- Repairs
- Kitchen maintenance
- Security guards
- Physical servers
- Networking equipment
- Cooling
- Power
- Disk failures
- Hypervisors
- Availability Zones
- Regions

---

## What You Own

You decide:

- The recipes
- The chefs
- Ingredients
- Menu
- Who can enter the kitchen
- Food safety
- Customer information
- Restaurant policies
- Application code
- IAM permissions
- Encryption choices
- Database contents
- Secrets
- Compliance

---

## The Twist

Imagine blaming the kitchen owner because your chef put salt instead of sugar into dessert.

That isn't the kitchen's fault.

Likewise...

Imagine blaming AWS because an IAM policy granted AdministratorAccess to everyone.

AWS gave you secure locks.

You handed out every key.

---

## The Different Restaurants

Now explain why responsibilities change.

### EC2

AWS gives you the kitchen.

- You cook.
- You clean.
- You maintain your appliances.

Lots of responsibility.

---

### RDS

AWS also maintains the ovens.

- You still cook.

Less responsibility.

---

### Lambda

AWS maintains almost everything.

- You simply submit recipes.

Very little infrastructure responsibility remains.

---

### SaaS

Now imagine Uber Eats.

- You don't even own the restaurant anymore.

You simply order food.

---

### Architectural Mapping

```text
Traditional Data Center

You own everything.

────────────────────────────

EC2

AWS
  Building
  Hardware
  Networking

You
  OS
  Runtime
  Application
  Data

────────────────────────────

RDS

AWS
  Building
  Hardware
  Networking
  Database Engine
  Patching
  Backups

You
  Schema
  Queries
  Data

────────────────────────────

Lambda

AWS
  Almost Everything

You
  Function
  IAM
  Data
```

---

## Knife Cut

People think AWS means:

> "AWS handles security."

Wrong.

AWS handles **security OF the cloud.**

You handle **security IN the cloud.**

That one preposition changes everything.

---

## Real World

When a company gets breached because:

- IAM was too permissive
- Secrets were committed to GitHub
- S3 bucket was public
- Database password was "Password123"

These are typically failures in the customer-controlled portion of the shared responsibility model.

AWS supplies security capabilities and assurance for its layer. Customers must configure and operate their workloads securely, and some controls are shared by both parties.

---

## The Last Bite

AWS doesn't remove responsibility.

It redistributes it.

The more managed the service...
The more AWS cooks.

The more time you have to build your application.

---

**Next chapter:** _Event-Driven Thinking_

"Now that you understand who runs the kitchen, let's explore a different question: **Why keep chefs standing around waiting for orders?**"
