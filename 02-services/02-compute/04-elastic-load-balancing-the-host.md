# Elastic Load Balancing: The Host Who Seats Every Customer

> Elastic Load Balancing gives customers one stable front door while routing each request to a healthy backend target — and different kinds of hosts make routing decisions in different ways.

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

Elastic Load Balancing (ELB) distributes incoming requests across multiple healthy targets.

Its job is not to build servers.
Its job is not to install software.
Its job is not to decide how many servers should exist.

Its only responsibility is:

> **Which healthy target should serve this request?**

## Three Kinds of Host

Not every front desk makes the same kind of decision.

One host reads the order ticket before sending the customer anywhere.

Another sees only that a customer has arrived and moves them immediately.

A third makes every customer pass through an inspection point first.

AWS calls these three routing jobs different kinds of load balancers.

### The Host Who Reads the Order

A customer walks up and asks for:

> `/api/orders`

The host reads the request.

API traffic goes to one group of shops.

Image traffic goes somewhere else.

Traffic for `api.example.com` may take a different path from traffic for `app.example.com`.

That is an **Application Load Balancer**, or **ALB**.

ALB routes at Layer 7, where the host can inspect HTTP details such as the path, hostname, headers, method, or query string before choosing a destination.

One front door can serve several applications because the host understands what the customer asked for.

### The Host Who Moves the Connection

Some traffic cannot wait for the host to inspect the order.

The connection arrives.

The host sends it onward.

No menu reading.

No path inspection.

That is a **Network Load Balancer**, or **NLB**.

NLB routes at Layer 4 using TCP, TLS, or UDP connection information. It fits workloads that need connection-level routing, non-HTTP protocols, or stable network addresses.

The host knows that a connection arrived.

Not what the customer plans to order.

### The Host With a Security Checkpoint

Sometimes the customer should not reach any shop directly.

Every arrival must first pass through a firewall, inspection system, or another network appliance.

That is the job of a **Gateway Load Balancer**, or **GWLB**.

GWLB inserts third-party virtual appliances into the traffic path.

The host does not choose the final shop first.

The host chooses the checkpoint.

## The Entrance, the Sections, and the Seating Chart

The host does not stand in the middle of the city.

She has a post.

That post watches a particular port and protocol, such as HTTPS on port 443.

AWS calls that a **listener**.

Behind the entrance, the city is divided into named sections.

One section serves API traffic.

Another serves static content.

Another may contain a fleet of containers.

AWS calls each section a **target group**.

The host follows a seating chart:

> "If the customer asks for `/api`, send them to the API section."
> "If the customer asks for `/images`, send them to the static-content section."
> "Otherwise, use the default section."

Those instructions are **routing rules**.

The sections stay the same.

The shops inside them can change.

Auto Scaling may open a new shop.

A deployment may replace an old one.

A container task may appear or disappear.

The host keeps following the same seating chart while the membership of each target group changes underneath it.

## Health Checks

The host does not assume every shop is ready.

She checks.

> Can you still serve customers?

Healthy targets remain in rotation.

Unhealthy targets stop receiving new traffic until they recover.

### Let the Customer Finish

A shop may be closing because Auto Scaling is scaling in or a deployment is replacing it.

The host stops sending new customers there.

But customers already seated do not move.

They finish their order.

Then the shop leaves the section.

AWS calls this **deregistration delay**, often described as connection draining.

The point is simple:

> Stop new work. Let in-flight work finish.

## Auto Scaling + ELB

Auto Scaling asks:

> **How many shops should exist?**

ELB asks:

> **Which healthy shop should serve this request?**

When Auto Scaling opens a new shop, ELB begins routing traffic after it passes health checks.

When Auto Scaling closes a shop, ELB drains in-flight requests through the deregistration window before the shop stops serving entirely.

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
          (Listener: HTTPS 443)
                     |
           Routing Rule Evaluates
          /                       \
         v                         v
   /api/* → API               /static/* → Static
   Target Group               Target Group
   (EC2 instances)            (EC2 instances)
         |                         |
    Auto Scaling             Auto Scaling
   maintains fleet          maintains fleet
```

## Painkiller

> **Problem:** Multiple application servers exist but customers should not choose one manually.
> **Pain:** Traffic becomes uneven, failed servers continue receiving requests, and routing logic is hard to update.
> **AWS solution:** Elastic Load Balancing presents one stable front door and routes requests to healthy targets based on configured rules — with different load balancer families offering connection-level, content-aware, or appliance-routed behavior.

## Knife Cut

> **Auto Scaling decides how many shops exist. Elastic Load Balancing decides where each healthy request goes.**

## The Masthead

### What Actually Just Happened

| In the story | In AWS | What it actually means |
|---|---|---|
| Front desk | Elastic Load Balancer | Single stable entry point for requests |
| Routing decision | Listener + routing rules | Configuration that maps conditions to target groups |
| Customer | Request | Incoming HTTP, TCP, or UDP traffic |
| What the customer ordered | HTTP path, hostname, headers | Layer 7 attributes used by ALB routing rules |
| Door number | TCP/UDP connection | Layer 4 attribute used by NLB routing |
| Security checkpoint | Gateway Load Balancer | Routes traffic through third-party inspection appliances |
| Section of the shop | Target group | Named set of backend targets |
| Shop | EC2 instance, container, IP, Lambda | Individual backend target |
| Closed shop | Unhealthy target | Target removed from rotation after failed health checks |
| Letting the customer finish | Deregistration delay | Window for in-flight requests to complete before target removal |
| Fleet | Auto Scaling Group | Group of instances scaled up or down based on demand |

## A Note From the Author

The boba host makes routing feel like one person looking across a city. Real load balancers make decisions through listeners, rules, target groups, health checks, and network protocols.

ALB, NLB, and GWLB are not faster and slower versions of the same host. They solve different routing problems. ALB understands HTTP content. NLB routes connections. GWLB inserts network appliances into the path.

Target groups can contain different kinds of targets, including EC2 instances, IP addresses, Lambda functions, and container tasks. Health checks, stickiness, cross-zone behavior, logging, and connection handling vary by load balancer family and configuration.

- [Elastic Load Balancing documentation](https://docs.aws.amazon.com/elasticloadbalancing/)
- [ALB routing conditions](https://docs.aws.amazon.com/elasticloadbalancing/latest/application/load-balancer-listeners.html)

---

## The Last Bite

Customers always visit one front door.

The host looks at what was ordered — or simply which door is free — and decides where they go.

Elastic Load Balancing does the same thing for applications: one stable entry point, routing decisions based on health and rules, and a clean handoff when servers come and go.

---

**Next chapter:** *Containers: The Food Truck That Carries the Kitchen*

Elastic Load Balancing directs customers across a fleet of healthy shops.

Containers ask what happens when the kitchen itself needs to become portable.
