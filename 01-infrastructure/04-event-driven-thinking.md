	## Stop Hiring People to Wait

### The Pain

Imagine opening a pizza restaurant.

You hire one employee.
Their job?

Stand next to the oven.
All day.
Waiting.

Maybe one order comes at noon.
Maybe another comes at six.
For the other seven hours...

they simply wait.

You still pay them.

---

## Traditional Computing

Servers often work exactly like that employee.

```
while(true){

    wait();

    if(work){

        doWork();

    }

}
```

The machine exists...

waiting.

CPU.
Memory.
Electricity.
Monitoring.
Patching.
Cost.

Even when nobody needs it.

---

## The Better Idea

What if nobody waited?

Instead...

The customer places an order.

🔔

The oven turns on.
Pizza gets cooked.
The oven turns off.

Nobody waits.

Nothing burns money while idle.

---

## AWS Calls This Event Driven

Something happens.

That "something" is called an event.

Examples:

- Someone uploads a photo.
- Payment succeeds.
- Order arrives.
- User logs in.
- File lands in S3.
- Timer reaches noon.
- Message enters a queue.

Those aren't requests.

They're events.

---

## Events Are Doorbells

Nobody stands at your front door all day.

Visitors ring the bell.
Then you answer.

Lambda works this way.

EventBridge works this way.

SNS works this way.

Step Functions work this way.

Even CloudWatch alarms work this way.

AWS loves events.

---

## Architectural Mapping

```
Old World

Server

↓

Wait

↓

Wait

↓

Wait

↓

Work

↓

Wait


────────────────────────

AWS

Event

↓

Service wakes up

↓

Work

↓

Done
```

---

## Why AWS Loves Events

Events solve several problems.

No idle infrastructure.
Better scaling.
Lower cost.
Faster reactions.
Loose coupling.
Independent systems.
Everything becomes smaller.

Simpler.

More resilient.

---

## Real Examples

Upload photo

↓

Lambda resizes image

────────────────────

Order placed

↓

SNS notifies systems

────────────────────

Payment completed

↓

EventBridge routes events

↓

Accounting updates

↓

Email sent

↓

Fraud analyzed

────────────────────

Message arrives

↓

SQS

↓

Lambda processes

---

## Knife Cut

Traditional software asks

> "Is there work yet?"

every second.

Event-driven software says

> "Tell me when work arrives."

One wastes energy.

One reacts only when needed.

---

## The Last Bite

AWS isn't trying to keep computers busy.

AWS is trying to keep **engineers** busy building products.

Computers can wait.

People shouldn't.

---

**Next chapter:** _How to Think Like an Architect_

Now that you understand how AWS reacts when work arrives, the next question is bigger:

**How do you decide which service should handle that work in the first place?**