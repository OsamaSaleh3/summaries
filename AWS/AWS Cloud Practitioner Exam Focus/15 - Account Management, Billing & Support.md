
## Executive Summary

This topic covers how AWS lets you **manage multiple accounts** (Organizations, Control Tower, SCPs), **understand and control what you spend** (Pricing Calculator, Cost Explorer, Budgets, Cost & Usage Reports), **get help when things break** (Trusted Advisor, Support Plans), and **understand AWS's pricing philosophy** (pay-as-you-go, reserved, volume discounts, Savings Plans). For the exam, focus on _what each tool/service is for_ and _when to pick one over another_ — not console click-paths.

---

## Core Concepts Explained

### 1. AWS Organizations

A **global, free service** that lets you centrally manage multiple AWS accounts. The account that creates the organization is the **Management (Master) Account**; every other account is a **Child Account**. Think of it like a parent holding one credit card for the whole family — one bill, shared perks.

**Key benefits:**

- **Consolidated Billing** – one bill for all accounts; no need to set up payment per account.
- **Pricing benefits from aggregated usage** – combined usage across accounts hits volume discount tiers faster (e.g., S3).
- **Shared Reserved Instances / Savings Plans discounts** – unused RI/Savings Plan discounts in one account can benefit EC2 usage in another account.
- **API for automated account creation** (e.g., spinning up sandbox accounts programmatically).
- **Service Control Policies (SCPs)** to restrict what accounts can do.

**Multi-Account Strategy:** organizations often split accounts by department, cost center, or environment (dev/test/prod) to isolate resources, apply different service limits, and separate logging. Accounts are grouped into **Organizational Units (OUs)** in a tree, with a Root OU at the top — OUs can be nested (e.g., Prod OU containing Finance OU and HR OU).

### 2. Service Control Policies (SCP)

An SCP is like an IAM policy but applied at the **OU or Account level** to whitelist or blacklist actions for everyone in that account — including the root user. Key rules:

- SCPs **do NOT apply to the Management (Master) Account**.
- SCPs apply to all IAM Users and Roles in an account, **but not to service-linked roles**.
- SCPs need an **explicit Allow** — by default, nothing is allowed.
- Common use cases: restrict access to certain services (e.g., "no EMR in production"), or enforce compliance (e.g., PCI) by blocking non-compliant services.

### 3. AWS Control Tower

Sits **on top of Organizations** and gives you an easy, few-click way to set up a secure, compliant multi-account environment following best practices. It automatically creates OUs, sets up guardrails (via SCPs) to prevent or detect policy violations, and gives you a compliance dashboard. Think of it as "Organizations + guardrails + automation" for teams that don't want to build multi-account governance by hand.

### 4. AWS Resource Access Manager (RAM)

Lets you **share AWS resources you own** (like a VPC, subnets, Transit Gateway, or Aurora databases) with other AWS accounts — including accounts outside your organization. This avoids duplicating resources: e.g., share one VPC so multiple accounts can launch resources into it and talk to each other over the private network.

### 5. AWS Service Catalog

A **self-service portal** for end users who shouldn't have unrestricted AWS access. Admins define **Products** (CloudFormation templates) and group them into a **Portfolio**, then control who can launch what. Users just pick from an approved catalog (e.g., "spin up an RDS database") without needing to know how to configure it correctly — ensuring consistency, tagging, and compliance.

### 6. AWS Compute Optimizer

Uses **machine learning** to analyze your EC2 instances, Auto Scaling Groups, EBS volumes, and Lambda functions (CPU/memory usage via CloudWatch metrics) and recommends better-sized (cheaper or higher-performing) resources. Can lower costs by up to 25%, and recommendations can be exported to S3.

### 7. Billing & Cost Tools

AWS groups these into three purposes: **estimate**, **track**, and **monitor** cost.

|Purpose|Tool|What it does|
|---|---|---|
|Estimate|**Pricing Calculator**|Estimate cost of a planned architecture before building it|
|Track|**Billing Dashboard**|High-level view: month-to-date cost, forecast, cost by service|
|Track|**Free Tier Dashboard**|Shows your usage against free tier limits|
|Track|**Cost Allocation Tags**|Tag resources (AWS-generated or user-defined) to break down costs by department/project/owner|
|Track|**Cost & Usage Reports (CUR)**|Most **comprehensive** and granular dataset — every cost, why it was incurred; analyzable via Athena/Redshift/QuickSight|
|Track|**Cost Explorer**|Visual tool; view cost/usage over time, get Savings Plan recommendations, **forecast up to 12 months ahead**|
|Monitor|**Billing Alarms**|Simple CloudWatch alarm on the billing metric (only stored in **us-east-1**), triggers email at a $ threshold|
|Monitor|**Budgets**|More powerful — alerts on actual or **forecasted** cost, usage, RI, or Savings Plan; up to 5 SNS notifications each|
|Monitor|**Cost Anomaly Detection**|Uses ML to learn your spending pattern and flag unusual spikes automatically — no manual thresholds needed|

**Budgets** support four types: **Cost, Usage, Reservation (RI), and Savings Plan** budgets. First two budgets are free; after that it's $0.02/day per budget.

### 8. AWS Service Quotas

Every AWS account has limits ("quotas") — e.g., max concurrent Lambda executions. Service Quotas lets you monitor these, set **CloudWatch Alarms** when you approach a limit, and **request a quota increase** directly from the console.

### 9. AWS Trusted Advisor

An automated, no-install "account health check" across six categories: **Cost Optimization, Performance, Security, Fault Tolerance, Service Limits, Operational Excellence**. With the free **Basic/Developer** plan you only get 7 **core security checks**. To unlock the **full set of checks** (including cost optimization) plus **programmatic access via the Support API**, you need a **Business or Enterprise support plan**.

### 10. AWS Support Plans

Four paid tiers plus a free Basic tier, differentiated mainly by **response time** and **who** you can talk to.

|Plan|Cost|Access|Trusted Advisor|Fastest response (prod system down)|
|---|---|---|---|---|
|**Basic**|Free|Docs, forums, 24/7 self-service|7 core checks only|N/A (no case support)|
|**Developer**|Paid|Business-hours email to Cloud Support Associates|7 core checks|~12 business hrs (system impaired)|
|**Business**|Paid|24/7 phone/email/chat, Cloud Support Engineers|**Full checks** + API access|< 1 hour|
|**Enterprise On-Ramp**|Paid|Pool of Technical Account Managers (TAMs), concierge team|Full checks|< 1 hour (business-critical down: < 30 min)|
|**Enterprise**|Paid|**Dedicated** TAM, concierge team, incident detection & response (extra fee)|Full checks|< 1 hour (business-critical down: **< 15 min**)|

### 11. Pricing Models & Fundamentals

AWS has four core pricing philosophies:

1. **Pay-as-you-go** – pay for what you use, stay agile.
2. **Save when you reserve** – commit for 1–3 years for predictable workloads (EC2, RDS, DynamoDB, ElastiCache, Redshift).
3. **Pay less by using more** – volume discounts (e.g., S3) as usage grows.
4. **Free services / Free Tier** – some services (IAM, VPC, Consolidated Billing, CloudFormation, Elastic Beanstalk, Auto Scaling) are free themselves, but resources they create (EC2, ALB) still cost money. Free Tier has 4 flavors: **Always Free, 12-Months Free, Trials, and Featured offers**.

**EC2 purchasing options:**

|Option|Discount vs On-Demand|Best for|
|---|---|---|
|On-Demand|— (baseline)|Short-term, unpredictable workloads; billed per second (Linux/Windows)|
|Reserved Instances|Up to 75%|Steady-state, long-term (1 or 3 yr commit)|
|Spot Instances|Up to 90%|Flexible, interruption-tolerant workloads (can be reclaimed)|
|Dedicated Host|Varies|Compliance/licensing needs — physical server dedicated to you|
|Savings Plans|Up to 72% (EC2)|Flexible $-per-hour commitment (see below)|

**Savings Plans** (simpler alternative to Reserved Instances — commit to a $/hour spend, not a specific instance):

|Type|Discount|Flexibility|
|---|---|---|
|**EC2 Instance Savings Plan**|Up to 72%|Locked to one instance family + region; flexible on size, OS, tenancy, AZ|
|**Compute Savings Plan**|Up to 66%|Most flexible — applies across instance family, region, size, OS, tenancy, **and even EC2/Fargate/Lambda**|
|**Machine Learning Savings Plan**|Varies|For SageMaker|

**Storage & database pricing** (high-level, no need to memorize exact numbers):

- **S3**: pay for storage volume/size (tiered discounts), requests, data transfer OUT (transfer in is free), Transfer Acceleration, and lifecycle transitions.
- **EFS**: pay-per-use, has an infrequent access tier + lifecycle rules.
- **EBS**: pay for **provisioned** size regardless of use, IOPS type (Provisioned IOPS costs extra), snapshots (per GB/month), and data transfer out.
- **RDS**: hourly billing based on engine/size; On-Demand or Reserved; backup storage free up to 100% of DB storage; pay for underlying storage, I/O requests, Multi-AZ (pay for 2 DBs), and data transfer out.
- **CloudFront**: priced by edge-location/continent, data transfer out (in is free), and number of requests.

**Networking costs (exam favorite):**

- Traffic **into** an EC2 instance: free.
- Same AZ, private IP: free.
- Different AZ, same region: **private IP cheaper** than public IP (public IP traffic routes over the internet).
- Different region: pay inter-region transfer fee.
- **Rule of thumb:** always prefer private IP for EC2-to-EC2 communication to save money and improve performance; using a single AZ maximizes savings but sacrifices high availability.

### 12. Consolidated Billing (deep dive)

Enabling this under Organizations gives you two things:

1. **Combined usage** → shared volume pricing (e.g., combined S3 usage crosses discount tiers faster) and **shared RI/Savings Plan discounts** across accounts.
2. **One bill** for the whole organization.

**Reserved Instance sharing example:** Account B owns 5 Reserved EC2 Instances but only runs 3 EC2 instances; Account A runs 6 EC2 instances with zero RIs. Because RI sharing is enabled, the **2 leftover reservations from Account B automatically apply to Account A's instances** — so 5 of the combined 9 instances get RI pricing even though B only "used" 3. RI sharing can be **turned off per account**, including the management account.

---

## The Big Picture

Think of this whole domain as three layers stacked on each other:

1. **Governance layer** – AWS Organizations is the foundation for running multiple accounts. Control Tower automates best-practice setup on top of Organizations. SCPs enforce guardrails within that structure. RAM lets accounts share resources instead of duplicating them, and Service Catalog gives end users safe, pre-approved ways to self-serve.
2. **Financial visibility layer** – Once accounts exist, you need to estimate (Pricing Calculator), track (Billing Dashboard, Cost Allocation Tags, CUR, Cost Explorer), and monitor (Billing Alarms, Budgets, Cost Anomaly Detection) spend — all made more powerful by Consolidated Billing pooling usage and discounts across accounts.
3. **Operational support layer** – Trusted Advisor gives automated best-practice checks, Service Quotas keeps you from silently hitting limits, and Support Plans determine how fast and how deeply AWS will help you when something breaks.

Underneath all of this sits AWS's core pricing philosophy (pay-as-you-go, reserve-to-save, volume discounts, free tier) which explains _why_ the cost tools and purchasing options exist in the first place.

---

## Exam Focus (keywords, cost traps, scenario triggers)

- **SCPs never apply to the Management/Master account.** SCPs need an explicit Allow (default deny) and don't apply to service-linked roles. Frequently tested with "why can't the root user in a child account use X service?" → SCP applied at OU/account level.
- **Organizations benefits are commonly tested as a set**: Consolidated Billing, aggregated volume pricing, shared RI/Savings Plans, API for account automation, and SCPs.
- **Reserved Instance / Savings Plan sharing** across a consolidated bill is a classic scenario question — know that unused reservations in one account flow to another account's usage automatically (and can be disabled).
- **Cost Explorer forecasts usage up to 12 months** — if a question asks "which tool predicts future AWS spend," it's Cost Explorer, not Budgets.
- **Billing metric/alarm data lives only in us-east-1** in CloudWatch, even though it aggregates cost from all regions.
- **Trusted Advisor**: only 7 core checks (mostly security) on Basic/Developer plans; full checks + Support API access require **Business or Enterprise**.
- **Support plan response times** for a "production system down" scenario: Business < 1 hr, Enterprise On-Ramp < 1 hr (business-critical < 30 min), Enterprise < 1 hr (business-critical < **15 min**). **AWS Incident Detection and Response** is an Enterprise-only add-on (extra fee) — flagged explicitly as exam-worthy.
- **Networking cost trap**: private IP is cheaper than public IP for cross-AZ EC2 traffic; same-AZ private traffic is free; cross-region always costs more.
- **Cost and Usage Report (CUR)** = most comprehensive/granular billing data (use with Athena/Redshift/QuickSight). **Cost Explorer** = visual, high-level, with forecasting. Don't confuse the two.
- **Compute Savings Plan vs EC2 Instance Savings Plan**: Compute Savings Plan is the more flexible one (covers EC2 + Fargate + Lambda); EC2 Instance Savings Plan is locked to a family/region but gives a higher max discount (72% vs 66%).

---

## Quick Reference Table

|Concept|What it is|Key thing to remember|
|---|---|---|
|AWS Organizations|Manage multiple accounts centrally|Global, free; Master + Child accounts; consolidated billing|
|SCP|Whitelist/blacklist IAM actions at OU/account|Never applies to Master account; default deny|
|Control Tower|Automated multi-account setup|Runs on top of Organizations; adds guardrails|
|AWS RAM|Share resources across accounts|Avoids duplicating VPCs, subnets, etc.|
|Service Catalog|Self-service portal of approved products|Products = CloudFormation templates; grouped in portfolios|
|Compute Optimizer|ML resource sizing recommendations|Up to 25% cost savings; covers EC2, ASG, EBS, Lambda|
|Pricing Calculator|Estimate cost before deploying|Use case: "how much will X cost me?"|
|Billing Dashboard|High-level cost overview|Includes Free Tier usage view|
|Cost Allocation Tags|Tag-based cost breakdown|AWS-generated (`aws:`) vs user-defined (`user:`) tags|
|Cost & Usage Report (CUR)|Most detailed billing dataset|Analyze via Athena/Redshift/QuickSight|
|Cost Explorer|Visual cost analysis + forecasting|Forecasts up to 12 months ahead|
|Billing Alarms|Simple CloudWatch $ threshold alert|Data only in us-east-1|
|Budgets|Alerts on cost/usage/RI/Savings Plan|First 2 free, then $0.02/day each|
|Cost Anomaly Detection|ML-based unusual spend alerts|No manual thresholds needed|
|Service Quotas|Track & raise account limits|CloudWatch alarms + request increase in console|
|Trusted Advisor|Automated best-practice checks|Full checks need Business/Enterprise support|
|Support Plans|Basic/Developer/Business/Ent. On-Ramp/Enterprise|Prod-down response: Business <1hr, Enterprise <15min (biz-critical)|
|Consolidated Billing|One bill + shared discounts for org|Shares volume pricing & RI/Savings Plan discounts|
|Savings Plans|$/hour commitment, alt. to RIs|Compute SP = most flexible; EC2 Instance SP = higher discount|
|Reserved Instances|1–3 yr commit for discount|Up to 75% off; AZ-specific|
|Spot Instances|Bid on unused EC2 capacity|Up to 90% off; can be reclaimed|
|Networking cost|Pricing for EC2-to-EC2 traffic|Private IP cheaper than public IP cross-AZ|