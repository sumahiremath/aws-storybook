---
description: "AWS WAF inspects HTTP(S) requests before they reach Byte Burger's public ordering systems and applies deliberate web-traffic rules."
tags:
  - "aws"
  - "identity-security"
  - "security"
  - "waf"
---

# AWS WAF: The Front-Door Screen

> AWS WAF inspects HTTP(S) requests before they reach Byte Burger's public ordering systems and applies deliberate web-traffic rules.

## The Business Goal

Byte Burger's online ordering counter is finally popular. Most requests are hungry customers. Some are malformed probes, scripted credential attacks, and one relentless client refreshing the menu faster than the kitchen can answer.

The team cannot solve that by making the API private: real customers still need it. Nor can the application treat every hostile request as ordinary business traffic without wasting capacity before it decides to reject it.

---

## The Story

Byte Burger adds a screening desk before the public entrance.

The screeners inspect each visitor's request: where it came from, which entrance it chose, whether the order form contains an obviously hostile pattern, and whether one visitor is arriving at an unreasonable rate. Some visitors are admitted, some are blocked, and some are counted quietly while the team tests a new rule.

The screeners do not know whether a customer has loyalty status or permission to refund an order. Those are still identity and application decisions inside the restaurant.

---

## Meet the AWS Service

AWS WAF is a web application firewall for HTTP(S) requests. A **web ACL** contains ordered rules and a default action, then associates with supported resources such as CloudFront, API Gateway, Application Load Balancer, AppSync, Cognito, and Amplify.

> **Core idea:** WAF filters web requests at the public edge. It is not user authentication, application authorization, or a cure for insecure application code.

---

## How It Works

### The screening policy

#### Web ACL

The web ACL is Byte Burger's screening policy for an associated public resource. It evaluates rules in priority order and applies the web ACL's default action if no terminating rule decides the request.

### The screening instructions

#### Rules and rule groups

Rules inspect request properties such as IP address, country, headers, URI path, query string, body size, SQL-injection-like input, and cross-site-scripting-like input. A rule group packages reusable rules. Byte Burger can use AWS Managed Rules, third-party managed rules, or carefully maintained custom rules.

Start unfamiliar managed rules in **Count** mode where appropriate. Count is non-terminating: it records matching traffic and lets later rules continue. Allow and Block are terminating actions, so a false positive can deny a real customer.

### The crowd rope

#### Rate-based rules

A rate-based rule aggregates incoming requests by configured criteria and rate limits groups that exceed its threshold over the evaluation window. Scope it narrowly—for example, the login or order-submission path—rather than treating every burst of legitimate browsing as abuse.

### The incident notebook

#### Logging and metrics

Enable logging, sampled requests, and CloudWatch metrics so the team can understand which rules matched, tune false positives, and investigate abuse. Rules consume capacity and managed protection can add cost; security controls need the same operational attention as application features.

---

## Architectural Mapping

| In the story | In AWS | What it actually means |
| --- | --- | --- |
| Screening desk | web ACL | Ordered request-inspection policy |
| Screening instruction | rule | Match criteria and action |
| Standard security playbook | managed rule group | Vendor-maintained reusable protections |
| Crowd rope at one entrance | rate-based rule | Request-rate control for matching aggregates |
| Quiet observation | Count action | Measure matches without blocking |

---

## When to Use It

Use AWS WAF when:

- A public HTTP(S) resource needs application-layer request filtering.
- Managed protections, custom request matches, or rate-based controls fit a known web-traffic risk.
- The team can monitor, test, and tune rules over time.

## When Not to Use It

Consider another or additional control when:

- The question is who the caller is or what business action they may perform; use Cognito, IAM, API authorization, and application logic.
- The workload is not a supported HTTP(S) resource.
- The risk is volumetric DDoS protection rather than application-layer request inspection; understand the distinct AWS Shield and network-protection boundary.

---

## Painkiller

> **Problem:** Public web endpoints receive both legitimate traffic and requests that should not reach the application.  
> **Pain:** Making the application inspect every request wastes capacity and leaves common web threats unfiltered.  
> **AWS solution:** Attach a monitored web ACL with appropriate managed, custom, and rate-based rules to the public resource.

---

## Knife Cut

> **WAF decides whether an HTTP request reaches the counter. Cognito, IAM, and application authorization decide who the caller is and what they may do once there.**

---

## The Masthead

### What Actually Just Happened

| In the story | In AWS | Precise technical meaning |
| --- | --- | --- |
| Visitor's order form | HTTP(S) request | Request inspected before the protected resource handles it |
| Admission or refusal | Allow/Block | Terminating web ACL action |
| Test clipboard | Count | Non-terminating observation action |
| One visitor refreshing nonstop | rate-based aggregation | Controlled requests exceeding configured rate criteria |
| Staff badge check inside | authentication/authorization | A separate identity and permission decision |

---

## A Note From the Author

The screening desk is a useful boundary, not a promise that every bad request is identified or every good request is admitted. WAF rules need testing, tuning, capacity awareness, monitoring, and careful handling of forwarded client IP information. A web ACL is associated with a supported resource; a rule group alone cannot be associated directly. See [AWS WAF protection packs (web ACLs)](https://docs.aws.amazon.com/waf/latest/developerguide/web-acl.html), [AWS WAF rules](https://docs.aws.amazon.com/waf/latest/developerguide/waf-rules.html), and [rate-based rules](https://docs.aws.amazon.com/waf/latest/developerguide/waf-rule-statement-type-rate-based.html).

---

## The Last Bite

The front-door screen reduces avoidable web traffic before it burdens the restaurant. It does not replace the locks, badges, or judgment inside.

---

**Next chapter:** *[AWS Certificate Manager and Amazon EC2: The Protected Staff Entrance](08-aws-certificate-manager-and-amazon-ec2-the-protected-staff-entrance.md)*

With public requests screened, Byte Burger must decide where the allowed work should run and what shape that work takes.
