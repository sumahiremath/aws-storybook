# Elastic Load Balancing: The Host Who Seats Every Customer

> Elastic Load Balancing gives customers one front door while intelligently routing each request to a healthy backend instance.

## The Pain

Yesterday, the business had one boba shop.

Everyone knew exactly where to go.

Then business exploded.

Auto Scaling opened five identical shops.

Customers still walked to the original location.
One shop had a line out the door.
Four shops waited quietly with empty counters.
Opening more shops solved capacity.

It did not solve traffic.

The business needed someone to greet every customer and send them to the right shop.

## The Story

Every customer walks to one front desk.

A smiling host looks across the city.

```text
Shop A   Busy
Shop B   Busy
Shop C   Almost Empty
Shop D   Closed
Shop E   Normal
```

The host says:

> "Welcome! Shop C can serve you immediately."

The next customer arrives.

> "Please head to Shop E."

Another customer arrives.

Shop E suddenly loses power.

Without announcing anything, the host simply stops sending customers there.

Customers keep arriving at the same front desk.

They never need to know which shop made the drink.

That front desk is Elastic Load Balancing.

## Meet Elastic Load Balancing

Elastic Load Balancing (ELB) distributes incoming requests across multiple healthy EC2 instances or other supported targets.

Its job is not to build servers.
Its job is not to install software.
Its job is not to decide how many servers should exist.

Its only responsibility is:

> **Which healthy shop should serve this customer?**

## Boba Mapping

| Boba | AWS |
|---|---|
| Front desk | Elastic Load Balancer |
| Host | Routing decision |
| Customer | Request |
| Shop | EC2 instance |
| Fleet | Auto Scaling Group |
| Closed shop | Unhealthy target |

## Health Checks

The host periodically asks every shop:

> Can you still make tea?

Healthy shops continue receiving customers.

Unhealthy shops are quietly removed until they recover.

## Auto Scaling + ELB

Auto Scaling asks:

> **How many shops should exist?**

ELB asks:

> **Which healthy shop should serve this customer?**

When Auto Scaling opens a new shop, ELB begins routing traffic after it passes health checks.

## Wrong Way

Publish five different shop addresses and let customers choose.

Some shops become overloaded.

Others stay empty.

Infrastructure should make routing decisions.

Customers should not.

## Architectural Mapping

```text
                Customers
                     |
                     v
        Elastic Load Balancer
             Front Desk Host
          /        |        \
         v         v         v
      EC2 A     EC2 B     EC2 C
             ^
             |
    Auto Scaling maintains the fleet
```

## Painkiller

> **Problem:** Multiple application servers exist but customers should not choose one manually.
> **Pain:** Traffic becomes uneven and failed servers continue receiving requests.
> **AWS solution:** Elastic Load Balancing presents one stable front door and routes requests only to healthy backend instances.

## Knife Cut

> **Auto Scaling decides how many shops exist. Elastic Load Balancing decides which healthy shop serves this customer.**

## The Last Bite

Customers always visit one front door.

The host quietly decides where they should go.

Elastic Load Balancing does the same thing for applications.

It gives every customer one destination while ensuring each request reaches a healthy backend.

---

**Next chapter:** *Containers: The Food Truck That Carries the Kitchen*

Elastic Load Balancing directs customers across a fleet of healthy shops.

Containers ask what happens when the kitchen itself needs to become portable.
