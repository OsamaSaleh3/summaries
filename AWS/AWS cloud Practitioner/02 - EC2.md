# EC2 (Elastic Compute Cloud)

## Executive Overview

**Amazon EC2 (Elastic Compute Cloud)** is one of the single most important and most frequently used services in all of AWS, and it is the flagship example of **Infrastructure as a Service (IaaS)** on the platform. EC2 allows you to rent **virtual servers** ("instances") on demand, in a matter of seconds, with full control over the operating system, compute power, memory, storage, and networking — without ever owning or managing physical hardware.

Understanding EC2 deeply is essential for the AWS Certified Cloud Practitioner (CLF-C02) exam because:

- It is the **conceptual foundation of cloud computing** itself: the ability to rent compute capacity elastically, pay only for what you use, and scale up or down instantly.
- EC2 is not a single service but an **umbrella of related capabilities** — instances, storage (EBS), networking/firewalling (security groups), load balancing, and auto-scaling — many of which appear as standalone exam topics later in the course.
- It introduces critical, testable concepts such as **AMIs, instance types, purchasing options, security groups, key pairs, and user data**.
- It reinforces the **Shared Responsibility Model** in a very concrete, hands-on way (AWS manages the physical hardware; you manage the OS, security groups, patches, and data).
- Billing nuances around EC2 (free tier, IPv4 charges, purchasing options) are common sources of exam questions and real-world cost-management pitfalls.

This guide walks through every concept covered across the full set of lessons: account budgeting and billing tools, connecting to instances (SSH, PuTTY, EC2 Instance Connect), IAM roles for EC2, purchasing options, IPv4 charging changes, the Shared Responsibility Model for EC2, and a deep hands-on walkthrough of launching, configuring, and securing an EC2 instance.

---

## 1. Billing, Budgets, and Cost Management (Account Setup)

Before diving into EC2 itself, the course establishes essential account-level billing tools to avoid unexpected charges — a practice every AWS user (and every exam candidate) should understand.

### 1.1 Accessing Billing Information

- Billing and Cost Management is accessed from the account name menu in the top-right corner of the console.
- **Important nuance**: Even an IAM user with full **AdministratorAccess** cannot view billing data by default. This is because access to billing/cost data is governed by a **separate root-account-level setting**, not by ordinary IAM policies.
- To allow IAM users (even administrators) to view billing information, the **root user** must go to **Account → IAM User and Role Access to Billing Information** and explicitly **activate** IAM access to billing. Once enabled, IAM users with appropriate permissions can see cost data.

> **Exam Tip:** Remember that billing/cost data access is controlled by a special root-account setting ("IAM User and Role Access to Billing Information"), separate from standard IAM policies. This is a subtle but testable nuance about how billing permissions work differently from other AWS permissions.

### 1.2 The Billing Dashboard

The Billing console dashboard provides:

- **Month-to-date cost** and **total forecasted cost** for the current month.
- **Last month's total cost**.
- A **cost breakdown by month** over time.

### 1.3 Bills and "Charges by Service"

- Under **Bills**, you can select a specific billing month and scroll to **Charges by Service**, which breaks down exactly how much each individual AWS service cost you (e.g., EC2, NAT Gateway, EBS storage, Elastic IP charges, etc.).
- This is the primary tool for **troubleshooting unexpected charges** — if your bill is higher than expected, "Charges by Service" lets you drill down service-by-service (and sub-item by sub-item) to see precisely what generated the cost.

### 1.4 AWS Free Tier Dashboard

- The **Free Tier** page (left-hand navigation) shows your **current usage** against free tier limits, as well as a **forecasted usage** projection.
- If your forecast is expected to exceed the free tier allowance, the dashboard highlights this in red, warning you that charges are imminent — a signal to turn off unused/running resources.

### 1.5 AWS Budgets

**AWS Budgets** lets you proactively set spending thresholds and receive email alerts when thresholds are reached or forecasted to be reached.

- **Zero Spend Budget** (template): Alerts you the moment you incur **any** cost (as little as $0.01) — ideal for a learning/sandbox account where you expect $0 in charges.
- **Monthly Cost Budget** (template): Lets you define a maximum acceptable monthly spend (e.g., $10/month) and configure multiple alert thresholds — for example:
    - Alert when **actual spend reaches 85%** of budget.
    - Alert when **actual spend reaches 100%** of budget.
    - Alert when **forecasted spend is expected to reach 100%** of budget.
- Alerts are sent via **email** to the address(es) you configure.

> **Use Case:** Budgets and Free Tier monitoring are essential safety nets for developers, students, and businesses alike — they prevent "bill shock" by catching runaway costs early, whether from a forgotten running instance, an oversized resource, or a misconfigured service.

> **Exam Tip:** Know that **AWS Budgets** is the correct service for setting proactive cost/usage alert thresholds (as opposed to Cost Explorer, which is for analyzing historical/current cost trends, or the Free Tier dashboard, which specifically tracks free-tier-eligible usage).

---

## 2. What is Amazon EC2?

### 2.1 Definition

**EC2 stands for Elastic Compute Cloud.** It is AWS's core **Infrastructure as a Service (IaaS)** offering, allowing you to rent resizable virtual computing capacity ("instances") in the cloud.

> **IaaS (Infrastructure as a Service)** is a cloud computing model where the provider (AWS) manages the underlying physical infrastructure (servers, storage hardware, networking hardware, data centers), while the customer manages everything above that layer — operating system, middleware, runtime, applications, and data. This is contrasted with PaaS (Platform as a Service) and SaaS (Software as a Service), which abstract away progressively more of the stack.

### 2.2 EC2 Is Not "Just One Service"

EC2 is best understood as an **umbrella of interconnected capabilities**, including:

- **EC2 Instances** — rentable virtual machines.
- **EBS (Elastic Block Store) Volumes** — virtual hard drives attached to instances.
- **Elastic Load Balancer (ELB)** — distributes incoming traffic across multiple instances.
- **Auto Scaling Groups (ASG)** — automatically scales the number of instances up or down based on demand.

Each of these components is explored in far greater depth elsewhere in a typical AWS course, but understanding that they are all part of the broader "EC2 family" is foundational to understanding how compute works on AWS.

### 2.3 Why EC2 Matters for Understanding "The Cloud"

The ability to **rent compute capacity on demand** — spinning up one instance or a hundred instances in seconds, paying only for what you use, and shutting them down when done — is the essence of cloud computing. EC2 is the clearest, most direct expression of this idea in AWS, which is why it's considered fundamental to understanding how the cloud works at all.

---

## 3. EC2 Instance Configuration Options

When configuring an EC2 instance, you make decisions across several dimensions:

### 3.1 Operating System (via the AMI)

You choose from three broad OS families:

- **Linux** (the most popular choice, especially for web servers and infrastructure).
- **Windows**.
- **macOS**.

This OS choice is embodied in something called an **AMI (Amazon Machine Image)** — a template that includes the operating system and, optionally, pre-installed software, configuration, and data. AWS provides several **"Quick Start" AMIs** (like **Amazon Linux 2**, a distribution maintained and optimized by AWS), and you can also create and reuse your own **custom AMIs**.

### 3.2 Compute Power (CPU)

You choose how many virtual CPU cores (**vCPUs**) the instance has — this determines its processing power.

### 3.3 Memory (RAM)

You choose how much **Random Access Memory (RAM)** the instance has, which affects how much data it can process/hold in memory at once.

### 3.4 Storage

You choose the type and amount of storage attached to your instance:

- **Network-attached storage**, such as:
    - **EBS (Elastic Block Store)** — persistent, network-attached block storage (covered in depth in a separate storage section).
    - **EFS (Elastic File System)** — a fully managed, shared network file system.
- **Hardware-attached (local) storage**, referred to as **EC2 Instance Store** — physically attached to the underlying host, offering very high performance but **non-persistent** storage (data is lost if the instance stops or the underlying hardware fails).

### 3.5 Networking

- The **type/speed of the network card** attached to the instance (network performance varies significantly by instance type/size).
- Whether the instance gets a **public IP address** (for internet-facing access) and/or a **private IP address** (for internal AWS network communication).

### 3.6 Firewall Rules (Security Groups)

Every EC2 instance has a **security group** attached, acting as a virtual firewall that controls inbound and outbound traffic (covered in full depth in Section 8).

### 3.7 Bootstrap Script (EC2 User Data)

An optional script that runs automatically **the very first time** the instance boots (see Section 4).

---

## 4. EC2 User Data (Bootstrapping)

### 4.1 What is Bootstrapping?

**Bootstrapping** means launching automated commands when a machine first starts up. In EC2, this is achieved via the **EC2 User Data** field.

### 4.2 Key Characteristics of EC2 User Data

- The User Data script runs **only once** — specifically, the very first time the instance is launched. It will **never run again** during subsequent stop/start cycles (this is distinct from a reboot script that would run every time).
- Its purpose is to **automate boot-time tasks**, such as:
    - Installing software updates.
    - Installing application software or web servers.
    - Downloading files or common resources from the internet.
    - Performing essentially any command-line configuration you want the instance to have ready "out of the box."
- **The EC2 User Data script runs with root user privileges.** This means every command in the script executes with elevated (root/sudo) permissions automatically — you do not need to explicitly prefix commands with `sudo`.
- **A practical trade-off:** the more logic and setup you cram into your User Data script, the **longer the instance takes to fully boot and become ready**, since all of that work must complete during the initial startup sequence.

### 4.3 Use Case Demonstrated

In the hands-on walkthrough, the User Data script was used to:

1. Update the instance's software packages.
2. Install the **HTTPD** web server (Apache HTTP Server).
3. Write a simple HTML file to disk (displaying a "Hello World" message along with the instance's private IP address).

This turned a freshly launched, blank EC2 instance into a **fully functioning web server**, entirely automatically, with zero manual post-launch configuration — a powerful illustration of Infrastructure as Code principles even at a very basic level.

> **Exam Tip:** Know that **User Data runs once at first boot only**, executes as root, and is the standard mechanism for automating initial server configuration ("bootstrapping") in EC2. This is distinct from concepts like **instance metadata** (queried via a special internal endpoint) which is a different, related concept covered elsewhere in more advanced courses.

---

## 5. EC2 Instance Types

### 5.1 Naming Convention

EC2 instance type names follow a consistent pattern, illustrated with the example **m5.2xlarge**:

|Component|Meaning|Example|
|---|---|---|
|**Instance Class (Family)**|The broad category of instance, indicating its optimization target|`M` = General Purpose|
|**Generation**|The hardware generation number — increases as AWS releases improved underlying hardware|`5` = 5th generation|
|**Size**|How large the instance is within its class — determines vCPU count, RAM, and relative cost|`2xlarge` (larger sizes = more vCPU/RAM/cost)|

Instance sizes typically scale in a progression such as: `micro → small → medium → large → xlarge → 2xlarge → 4xlarge → 8xlarge → 16xlarge`, and so on — the larger the size designation, the more CPU, memory, and (usually) network performance the instance provides.

### 5.2 Instance Type Families (Categories)

There are hundreds of specific EC2 instance types, but they fall into a handful of major families, each optimized for different workload characteristics:

#### 5.2.1 General Purpose

- **Balanced** mix of compute, memory, and networking resources.
- **Use cases**: web servers, code repositories, and diverse workloads that don't have one dominant resource requirement.
- **Example family name**: `T` series (e.g., `t2.micro`, `t3.micro`). The `T2/T3 micro` instance types are specifically noted as being **free-tier eligible**, which is why they are used extensively in introductory hands-on labs.

#### 5.2.2 Compute Optimized

- Optimized for **compute-intensive tasks** requiring a high-performance processor.
- **Use cases**: batch processing, media transcoding, high-performance web servers, **High Performance Computing (HPC)**, machine learning inference/training tasks, and dedicated gaming servers.
- **Naming convention**: `C` series (e.g., `C5`, `C6`).

#### 5.2.3 Memory Optimized

- Optimized for workloads that process **large datasets in memory** (RAM).
- **Use cases**: high-performance relational or non-relational (NoSQL) databases, in-memory databases, distributed web-scale caching stores (such as **Amazon ElastiCache**), Business Intelligence (BI) workloads, and real-time processing of large unstructured datasets.
- **Naming convention**: `R` series (R = "RAM"), as well as `X1` (high memory) and `Z1` series.

#### 5.2.4 Storage Optimized

- Optimized for workloads requiring **high, fast access to large local datasets**.
- **Use cases**: high-frequency **Online Transaction Processing (OLTP)** systems, relational and NoSQL databases, in-memory caching databases (e.g., Redis-backed caches), data warehousing applications, and distributed file systems.
- **Naming convention**: instance families starting with `I`, `G`, or `H1` (e.g., I3, G-series for some storage-optimized use cases, H1).

### 5.3 Comparing Instance Types

The transcript provides direct comparisons to illustrate how resources scale:

|Instance Type|vCPUs|Memory|Notes|
|---|---|---|---|
|`t2.micro`|1|1 GB|Free-tier eligible; low-to-moderate network performance|
|`t2.xlarge`|4|16 GB|Same family, larger size; moderate network performance|
|`c5d.4xlarge`|16|32 GB|Compute-optimized; includes 400 GB local NVMe SSD storage; up to 10 Gbps network|
|`r5.16xlarge`|16|512 GB|Memory-optimized; heavy emphasis on RAM relative to CPU|
|`m5.8xlarge`|(varies)|(varies)|General purpose, large size|

> **Practical takeaway:** choose the instance family/type that best matches your **application's actual bottleneck** — CPU-bound workloads go to compute-optimized types, memory-bound workloads (like databases/caches) go to memory-optimized types, and general-purpose workloads (web servers, small apps) fit comfortably on general-purpose types like the `T` series.

### 5.4 Reference Tools

- The **official AWS EC2 Instance Types page** is the canonical, always-up-to-date reference for browsing all available instance types, their specifications, and family groupings.
- A popular third-party community resource, **instances.info** (referred to in the transcript as "instituteinstances.info"), aggregates all EC2 instance types into a single sortable/searchable table, showing vCPU count, memory, on-demand pricing, and reserved pricing — a very convenient tool widely used by AWS practitioners for quick comparisons.

> **Exam Tip:** For the CLF-C02 exam, you do not need to memorize exact specs of every instance type. You **do** need to recognize the naming convention pattern (class + generation + size) and know, at a conceptual level, which family (General Purpose, Compute Optimized, Memory Optimized, Storage Optimized) fits which type of workload description in a scenario-based question.

---

## 6. Launching an EC2 Instance (Hands-On Walkthrough)

This section captures the detailed step-by-step process demonstrated for launching a first EC2 instance via the AWS Management Console.

### 6.1 Step 1 — Name and Tags

- Provide a **Name** tag for the instance (e.g., "My First Instance") — this is technically just a tag with the key `Name`.
- Additional custom tags can optionally be added for further metadata/organization.

### 6.2 Step 2 — Choose an AMI (Application and OS Image)

- AWS provides **"Quick Start" AMIs** for convenience, including **Amazon Linux 2** (built and maintained by AWS, and free-tier eligible).
- You choose the **architecture** as well (e.g., **64-bit x86**).
- Beyond quick-start images, there is a **full AMI catalog** you can search, including AMIs shared by other AWS users or purchased from the **AWS Marketplace**, and you can also create and save your **own custom AMIs** from a configured instance for reuse later.

### 6.3 Step 3 — Choose an Instance Type

- Instance types differ in **CPU count, memory, and cost**.
- `t2.micro` is **free-tier eligible** — free to run continuously for an entire month within the free tier limits.
- Older-generation types like `t1.micro` may also be free-tier eligible but represent **older hardware generations**.
- Depending on your AWS Region, if `t2.micro` isn't available, AWS may default to a comparable type like `t3.micro`.

### 6.4 Step 4 — Key Pair for Login

- A **key pair** is required if you intend to use SSH (Secure Shell) to log into the instance.
- Key pair configuration includes:
    - **Key pair type**: **RSA** encryption is the standard/recommended choice.
    - **Key pair format**:
        - **.pem** — used on **macOS, Linux, and Windows 10+** (Windows 10 introduced native OpenSSH client support).
        - **.ppk** — used specifically with **PuTTY**, the SSH client typically required on **Windows 7 and Windows 8** (versions of Windows that lack a native SSH client).
- Upon creation, AWS **automatically downloads the private key file to your computer** — this is the **only time** you can retrieve this file, so it must be stored securely (this mirrors the same one-time-download principle as IAM access keys).
- It is technically possible to launch an instance **without** a key pair, but this removes your ability to SSH into it directly (relevant only if you plan to manage the instance exclusively through other means).

### 6.5 Step 5 — Network Settings and Security Group

- By default, the instance is configured to receive a **public IP address**.
- A **security group** is created and attached to control inbound/outbound traffic. The first auto-created security group is typically named something like `launch-wizard-1`.
- Common initial rules configured through the wizard:
    - **Allow SSH traffic from anywhere** (port 22) — needed to remotely administer the instance.
    - **Allow HTTP traffic from the internet** (port 80) — needed if you intend to serve a website.
    - (HTTPS/port 443 can be added separately if needed, but wasn't required for this introductory example.)

### 6.6 Step 6 — Configure Storage

- By default, an **8 GB gp2 (General Purpose SSD)** root volume is attached.
- The AWS Free Tier includes up to **30 GB of EBS General Purpose SSD storage** per month, so an 8 GB root volume comfortably fits within free-tier limits.
- Under **Advanced** storage settings, one particularly important setting is **"Delete on Termination"**:
    - By default, this is set so that the **root EBS volume is deleted automatically when the instance is terminated**.
    - This behavior can be changed if you want the volume to persist independently of the instance's lifecycle.

### 6.7 Step 7 — Advanced Details (Including User Data)

- Advanced configuration options include things like **Spot instance requests** and **IAM Instance Profile** assignment (both explored in more depth elsewhere).
- At the very bottom of this section is the **User Data** field, where you paste in your bootstrap script (see Section 4).

### 6.8 Step 8 — Review and Launch

- A final summary confirms your selections, including free-tier eligibility notes (e.g., "750 hours of t2.micro per month," "30 GB of EBS storage").
- Clicking **Launch Instance** provisions the instance, which enters a **"pending"** state for roughly 10–15 seconds before transitioning to **"running."**

> **Use Case / Benefit:** The ability to provision one instance — or a hundred instances — in under 10 seconds, without owning or managing a single physical server, is a direct, tangible demonstration of the elasticity and speed benefits of cloud computing. This on-demand provisioning capability underlies virtually every other AWS service and architecture pattern.

### 6.9 Instance Details After Launch

Once running, the EC2 console shows a wealth of information about the instance, including:

- **Instance Name** and **Instance ID** (a unique identifier).
- **Public IPv4 address** — used to access the instance from the internet.
- **Private IPv4 address** — used for internal communication within the AWS network (not internet-routable).
- **Instance state** (e.g., running, stopped, pending, terminated).
- **Hostname**, **Private DNS**, **Instance Type**, **AMI used**, and **Key Pair name**.
- **Security** tab — shows attached security group(s) and their rules.
- **Storage** tab — shows attached EBS volumes.

### 6.10 Accessing the Web Server

- To view the running "Hello World" web server, copy the **Public IPv4 address** and access it via a browser using the **`http://`** prefix (not `https://` — since no SSL/TLS certificate was configured, HTTPS traffic would hang indefinitely rather than connecting, since nothing is listening on port 443).
- The webpage displays a "Hello World" message that includes the **private IP address** of the instance (as programmed into the User Data script) — illustrating the distinction between the public IP (used to reach the instance from outside) and the private IP (the instance's internal identity on the AWS network).

---

## 7. Instance Lifecycle: Start, Stop, and Terminate

### 7.1 Stopping an Instance

- Navigate to **Instance State → Stop Instance**.
- While stopped, **AWS does not bill you for compute (instance) time**, though you may still incur charges for **attached storage (EBS volumes)**, since the volume and its data persist.
- A stopped instance can be **started again later**.

### 7.2 Starting an Instance (Important IP Behavior)

- Starting a previously stopped instance transitions it through a **"pending"** state before becoming **"running"** again.
- **Critical behavior**: The instance's **Public IPv4 address will very likely change** after a stop/start cycle (unless an Elastic IP is explicitly attached — a static public IP address feature covered in more advanced networking material).
- The **Private IPv4 address remains constant** across stop/start cycles, for the life of the instance.

> **Exam Tip:** Public IPv4 addresses are **not guaranteed to persist** across a stop/start cycle by default; only an **Elastic IP** would provide a persistent public IP. Private IPv4 addresses, in contrast, remain stable throughout an instance's lifetime (until termination).

### 7.3 Terminating an Instance

- **Terminate Instance** permanently destroys the instance (and, by default, its root EBS volume, per the "Delete on Termination" setting discussed above).
- This action is **irreversible** — unlike stopping, a terminated instance cannot be restarted; you would need to launch an entirely new instance.

> **Use Case / Benefit:** The ability to freely start, stop, and terminate instances on demand — with billing that reflects only actual running time — is central to the cost-efficiency benefits of cloud computing. It is very common practice in real-world AWS usage to spin up instances for temporary workloads and destroy them immediately afterward, minimizing cost.

---

## 8. Security Groups (EC2 Firewalls)

### 8.1 What is a Security Group?

A **Security Group** acts as a **virtual firewall** controlling network traffic to and from an EC2 instance. Security groups are fundamental to network security in AWS.

### 8.2 Core Characteristics

- Security groups **only contain ALLOW rules** — you cannot create an explicit "deny" rule within a security group (this is a key distinguishing feature versus **Network ACLs**, a separate VPC-level concept that supports both allow and deny rules, covered in networking sections elsewhere).
- Rules can reference:
    - **IP address ranges** (e.g., where your computer connects from), or
    - **Other security groups** (enabling security groups to reference each other — see Section 8.6).
- Security groups regulate:
    - **Inbound traffic** — from outside sources into the instance.
    - **Outbound traffic** — from the instance out to external destinations.
- Rules are defined over both **IPv4** and **IPv6** address ranges.

### 8.3 Anatomy of a Security Group Rule

Each rule specifies:

- **Type** — a shortcut/preset for common protocols (e.g., SSH, HTTP, HTTPS), which auto-fills the correct port.
- **Protocol** — e.g., TCP.
- **Port Range** — which port(s) the rule applies to.
- **Source (for inbound) / Destination (for outbound)** — an IP address range in **CIDR notation**.
    - `0.0.0.0/0` means **"anywhere" (all IPv4 addresses)**.
    - A single specific IP (e.g., `x.x.x.x/32`) restricts access to just that one address — useful for a "my IP only" rule.

### 8.4 Default Behavior

> **By default, all inbound traffic is BLOCKED, and all outbound traffic is ALLOWED.**

This means a freshly created security group (before you add any custom rules) will refuse **all** connection attempts from the outside world, while still letting the instance freely initiate outbound connections (e.g., to download software updates or call external APIs).

### 8.5 Security Group Rules and EC2 Behavior

- Security groups are **"stateful"** in nature and **live entirely outside the EC2 instance itself** — the instance's own operating system/application is completely unaware of blocked traffic. If a security group blocks incoming traffic, the **EC2 instance never even sees the connection attempt**.
- A security group can be attached to **multiple EC2 instances**, and **a single EC2 instance can have multiple security groups attached simultaneously** — there is no strict one-to-one relationship. When multiple security groups are attached, their rules are combined (additive).
- Security groups are **scoped to a specific Region/VPC combination** — if you switch regions, or use a different VPC, you must create a new security group; security groups cannot span across regions or VPCs.
- **Best practice tip** (from the instructor): maintain a **separate, dedicated security group specifically for SSH access**, since SSH misconfigurations are a very common source of connectivity problems, and isolating SSH rules makes troubleshooting easier.

### 8.6 Security Groups Referencing Other Security Groups

A more advanced (but common and testable) pattern: instead of authorizing traffic by IP address, a security group's inbound rule can authorize traffic **from another security group** as its source.

**Example scenario:**

- EC2 Instance A has **Security Group 1** attached.
- Security Group 1's inbound rules **allow traffic from Security Group 1 itself and from Security Group 2**.
- EC2 Instance B has **Security Group 2** attached — Instance B can now communicate with Instance A on the allowed port, **regardless of Instance B's actual IP address**, because the rule is based on group membership, not a specific IP.
- If a third instance (Instance C) has **Security Group 3** attached, and Security Group 3 was never referenced in Security Group 1's inbound rules, then Instance C's traffic to Instance A would be **denied**.

> **Use Case / Benefit:** This pattern is extremely common when working with **Elastic Load Balancers** — instead of hardcoding the load balancer's IP addresses (which can change) into your instances' security groups, you simply authorize traffic from the **load balancer's security group**. This decouples your security rules from ever-changing IP addresses and is considered a networking best practice.

### 8.7 Troubleshooting: Timeout vs. Connection Refused

This is one of the most practically useful (and testable) diagnostic distinctions in networking on AWS:

|Symptom|Likely Cause|
|---|---|
|**Timeout** (connection hangs indefinitely, then eventually fails)|A **security group** is blocking the traffic — the packet never reaches the instance, so there is no response at all.|
|**Connection Refused** (an explicit rejection response is received)|The security group **allowed** the traffic through, but the **application/service on the instance** isn't running, isn't listening on that port, or actively rejected the connection. This is an **application-level** issue, not a security-group issue.|

> **Exam Tip:** This distinction — **timeout = security group problem; connection refused = application problem** — is one of the most classic, practically useful troubleshooting heuristics for EC2 networking questions on the exam.

### 8.8 Key Ports to Memorize

|Port|Protocol|Purpose|
|---|---|---|
|**22**|**SSH** (Secure Shell)|Used to log into a **Linux** EC2 instance remotely via a secure, encrypted terminal session.|
|**21**|**FTP** (File Transfer Protocol)|Used to upload files to a file share (unencrypted).|
|**22**|**SFTP** (SSH File Transfer Protocol)|Used to securely upload files, leveraging the SSH protocol for encryption (note: shares port 22 with SSH).|
|**80**|**HTTP**|Used to access **unsecured** websites.|
|**443**|**HTTPS**|Used to access **secured** (encrypted, TLS/SSL) websites — the modern web standard.|
|**3389**|**RDP** (Remote Desktop Protocol)|Used to log into a **Windows** EC2 instance remotely via a graphical desktop session.|

> **Exam Tip:** A very common exam pattern is a scenario asking "which port would you open to allow X" — memorize this table precisely: **22 = SSH (Linux)**, **3389 = RDP (Windows)**, **80 = HTTP**, **443 = HTTPS**, **21 = FTP**.

---

## 9. Connecting to an EC2 Instance

There are multiple methods for remotely connecting to and controlling an EC2 instance, and the right method depends on your operating system and preferences.

### 9.1 SSH (Secure Shell) — Overview

**SSH** is a protocol/utility that lets you **remotely control a machine using your terminal or command line**, as if you were sitting directly in front of it. It operates over **port 22** and is one of the most important tools for working with cloud servers.

- **macOS, Linux, and Windows 10+** have a **native, built-in SSH command-line utility** (no extra installation required).
- **Windows 7 and Windows 8** (or any Windows version lacking native SSH support) require a **third-party SSH client**, most commonly **PuTTY**, a free tool available online. PuTTY accomplishes the exact same underlying task as the native `ssh` command.
- **EC2 Instance Connect** (see 9.3) is a **browser-based alternative** that works identically across all operating systems.

### 9.2 SSH on macOS/Linux — Step-by-Step

1. Locate your downloaded private key file (e.g., `EC2Tutorial.pem`) and ensure the filename contains **no spaces**.
2. Place the file in a directory of your choice.
3. Retrieve the instance's **Public IPv4 address** from the EC2 console.
4. Confirm the security group allows **SSH (port 22) from your IP or from anywhere (`0.0.0.0/0`)**.
5. Navigate your terminal to the directory containing your `.pem` file (use `pwd` to check your current directory and `cd` to change into the correct one; `ls`/`ll` to confirm the file is present).
6. Run the connection command:
    
    ```
    ssh -i EC2Tutorial.pem ec2-user@<public-ip-address>
    ```
    
    - The `-i` flag specifies the identity (private key) file to use for authentication.
    - `ec2-user` is the **default username pre-configured on the Amazon Linux 2 AMI** specifically (other AMIs may use different default usernames, such as `ubuntu` for Ubuntu-based AMIs).
7. **Common errors and fixes**:
    - **"Too many authentication failures"** — typically means the key file wasn't correctly referenced in the command (or you're relying on the wrong default key).
    - **"UNPROTECTED PRIVATE KEY FILE" warning** — the `.pem` file's permissions are too open. Fix with:
        
        ```
        chmod 0400 EC2Tutorial.pem
        ```
        
        This restricts the file so that only the file's owner can read it (and no one can write or execute it), satisfying SSH's security requirements for private key files.
    - You may be prompted to confirm trusting the host the first time you connect — type **"yes"** to proceed.
8. Once connected, the terminal prompt changes to reflect the remote session (e.g., `ec2-user@<ip>`), confirming you are now issuing commands directly on the remote EC2 instance.
9. To disconnect: type `exit`.

### 9.3 SSH on Windows 10 (Native)

- Windows 10 includes a built-in SSH client accessible via **PowerShell** or **Command Prompt**.
- The process mirrors the macOS/Linux steps exactly:
    1. Navigate to the directory containing the `.pem` file (`cd`, `ls`).
    2. Run: `ssh -i EC2Tutorial.pem ec2-user@<public-ip>`.
    3. If you encounter **permission errors** on Windows (a common issue, since Windows file permissions work differently from Linux/macOS), you must adjust the file's **NTFS permissions**:
        - Right-click the `.pem` file → **Properties → Security tab → Advanced**.
        - Ensure **you (the current Windows user) are the file's Owner**.
        - **Remove inherited permissions** (disable inheritance) so that broad groups like "System" and "Administrators" no longer have automatic access.
        - Grant **only your own user account** full control over the file.
    4. Once permissions are corrected, the SSH command will work without further prompting.

### 9.4 SSH on Windows 7/8 (via PuTTY)

**PuTTY** is a free SSH client for Windows that provides equivalent functionality to the native `ssh` command, and it also works on any version of Windows (including 10), not just older ones.

- **PuTTYgen** (bundled with PuTTY) is used to **convert a `.pem` file into the `.ppk` format** that PuTTY requires, if you don't already have a `.ppk` key:
    1. Open PuTTYgen → **Load** → select your `.pem` file.
    2. Click **Save private key** → save as a `.ppk` file.
- **Configuring PuTTY to connect**:
    1. Open PuTTY → enter the **Host Name** field as `ec2-user@<public-ip-address>` (combining the default username and the instance's public IP directly into the hostname field).
    2. Ensure **connection type is SSH**.
    3. Save this configuration under a memorable session name (e.g., "EC2 Instance") for reuse.
    4. Navigate to **Connection → SSH → Auth** and browse to select your `.ppk` private key file for authentication.
    5. Return to the main **Session** screen and click **Save** again to persist the full configuration (host + key).
    6. Click **Open** to establish the connection — accept the host's authenticity prompt if shown.
- Once connected, you receive a full terminal session into the instance, identical in capability to native SSH.

### 9.5 EC2 Instance Connect (Browser-Based SSH)

**EC2 Instance Connect** is a simpler, modern alternative that establishes an **SSH session directly through your web browser**, with no local terminal, SSH client, or key file management required on your end.

- Accessed via: select the instance → click **Connect** → choose the **EC2 Instance Connect** tab.
- The username field is **pre-filled** (typically `ec2-user` for Amazon Linux 2) and generally should not be changed.
- **No manual SSH key management is required** — AWS automatically generates and uploads a **temporary SSH key** behind the scenes to establish the session, then discards it afterward.
- Clicking **Connect** opens a new browser tab with a fully functional terminal session directly into the instance.
- **Important limitations**:
    - EC2 Instance Connect currently works **only with Amazon Linux 2** (and certain other supported AMIs) — this is why Amazon Linux 2 is used consistently throughout introductory hands-on labs.
    - It works across **any operating system you're using locally** (Windows, macOS, Linux) since it runs entirely in the browser.
    - It still relies on the **SSH protocol and port 22 under the hood** — meaning your security group **must still allow inbound traffic on port 22** for EC2 Instance Connect to function, exactly as with traditional SSH. If port 22 is closed, EC2 Instance Connect will fail with a connectivity error, just like normal SSH would.
    - Depending on your setup, you may also need to allow **port 22 for IPv6** in addition to IPv4, if IPv6 connectivity is being attempted.

> **Use Case / Benefit:** EC2 Instance Connect removes the friction of key file management and local SSH client setup, making it ideal for quick administrative tasks, troubleshooting, or for users unfamiliar with the command line. It is especially useful in training/demo environments.

### 9.6 Troubleshooting Summary for Connectivity

- If **none of your connection methods** (SSH, PuTTY, EC2 Instance Connect) work, always double-check:
    - The security group allows inbound traffic on the relevant port (22 for SSH).
    - You are using the correct **current** public IP address (remember: it can change after a stop/start cycle).
    - Your key file has the correct permissions (`chmod 0400` on macOS/Linux, or corrected NTFS ownership/permissions on Windows).
    - You are running commands from the correct local directory containing your key file.
- You do **not** need every connection method to work — succeeding with **just one** method is sufficient to proceed with hands-on work.

---

## 10. IAM Roles for EC2 Instances

### 10.1 The Problem: Never Hardcode Credentials on an Instance

A critical, heavily emphasized security principle: **you should never run `aws configure` and enter your personal IAM Access Key ID and Secret Access Key directly on an EC2 instance.**

**Why this is dangerous:**

- If credentials are configured locally on an instance, **anyone else with access to that instance** (e.g., via EC2 Instance Connect or SSH, if they have permission) could retrieve those stored credentials and use them to impersonate you elsewhere in the AWS account.
- This creates a serious security exposure, especially in shared or larger organizational environments.

> **Golden Rule:** _Never, ever, ever enter your personal IAM access keys onto an EC2 instance._ If you see this being done, it should be corrected immediately.

### 10.2 The Solution: IAM Roles

Instead of embedding personal credentials, you attach an **IAM Role** to the EC2 instance (as introduced conceptually in the IAM section of the course). The role grants the instance itself a set of permissions, managed centrally and revocable at any time, **without ever exposing a static, long-lived secret key on the instance**.

### 10.3 Attaching a Role — Step-by-Step

1. Select the instance in the EC2 console.
2. Go to **Actions → Security → Modify IAM Role**.
3. Choose the desired role from the dropdown (e.g., a previously created role like `DemoRoleForEC2`, which had the **`IAMReadOnlyAccess`** policy attached).
4. Click **Save**.

### 10.4 Verifying the Role Works

- Before attaching the role: running `aws iam list-users` from within the instance's terminal fails with an error like _"Unable to locate credentials."_
- After attaching the role: the exact same command **succeeds**, returning IAM user data — demonstrating that the instance now has valid, temporary credentials derived from the attached role, **without any manual `aws configure` step**.
- If the policy is later **detached** from the role, the same command will begin returning an **Access Denied** error — proving the permissions are dynamically tied to the role's current policy attachments, evaluated live on every API call.
- **Propagation delay**: When re-attaching a policy to a role, there can be a **short delay** before the change takes effect across AWS's systems — if a command fails immediately after re-attaching a policy, retrying a moment later often succeeds once the change has propagated.

> **Exam Tip:** This is one of the most testable practical scenarios in the entire EC2/IAM curriculum: _"How should an EC2 instance be granted permission to call other AWS services?"_ — the correct answer is always **attach an IAM Role to the instance**, never store access keys on it.

---

## 11. EC2 Purchasing Options

Choosing the right EC2 **purchasing option** is a major cost-optimization decision and a heavily tested exam topic. Each option makes different trade-offs between **cost, flexibility, and commitment**.

### 11.1 On-Demand Instances

- **Billing**: Pay for what you use.
    - **Linux and Windows**: billed **per second**, after the first minute.
    - **All other operating systems**: billed **per hour**.
- **Cost**: Highest per-hour/per-second cost among all purchasing options.
- **Commitment**: **No upfront payment, no long-term commitment.**
- **Best for**: **Short-term, unpredictable workloads** where you cannot forecast how the application will behave, or where flexibility to start/stop instantly is more valuable than cost savings.

### 11.2 Reserved Instances

- **Discount**: Up to **72% discount** compared to On-Demand pricing (approximate figure — subject to change over time).
- **Mechanics**: You reserve **specific instance attributes** — instance type, region, tenancy, and operating system — for a fixed term.
- **Term options**: **1 year or 3 years** (longer terms yield greater discounts).
- **Payment options**: **No upfront, partial upfront, or all upfront** (all-upfront yields the maximum discount).
- **Scope**: Can be scoped to a **specific Region** or to a **specific Availability Zone (AZ)** (the latter also reserving capacity in that AZ).
- **Best for**: **Steady-state, predictable, long-running workloads** — the classic example being a **database server** that runs continuously for years.
- **Flexibility**: Reserved Instances can be **bought or sold on the AWS Reserved Instance Marketplace** if your needs change and you no longer require them.
- **Convertible Reserved Instances**: A special variant that allows you to **change the instance type, instance family, operating system, scope, and tenancy** over the term. Because of this added flexibility, the discount is slightly lower — **up to 66%** — compared to standard Reserved Instances.

### 11.3 EC2 Savings Plans

- **Discount**: Similar magnitude to Reserved Instances (roughly **~72%**, aligned with the source material).
- **Mechanics**: Instead of committing to specific instance attributes, you commit to a **specific dollar amount of usage per hour** (e.g., "$10/hour") over a **1-year or 3-year term**.
- **Flexibility**: You remain **locked into a specific instance family and region** (e.g., "M5 family in us-east-1"), but you retain flexibility across:
    - **Instance size** (e.g., freely switch between `m5.xlarge`, `m5.2xlarge`, etc.)
    - **Operating System** (e.g., switch between Linux and Windows)
    - **Tenancy** (e.g., switch between default, dedicated, and dedicated host)
- **Overage handling**: Any usage **beyond** your committed savings plan amount is simply billed at standard **On-Demand rates**.
- **Best for**: Long-term, predictable workloads where you want cost savings but slightly more flexibility than a traditional Reserved Instance offers (in exchange for locking in a specific instance family/region rather than exact instance attributes).

### 11.4 Spot Instances

- **Discount**: The most aggressive discount available — up to **90% off** On-Demand pricing.
- **Mechanics**: You specify a **maximum price** you are willing to pay. As long as the current "spot price" for that instance type stays below your maximum, your instance keeps running. **If the spot price rises above your maximum, AWS can reclaim (terminate) your instance at any time**, with little to no warning.
- **Reliability**: **The least reliable purchasing option** — instances can be interrupted/terminated unpredictably.
- **Best for**: Workloads that are **resilient to interruption**, such as:
    - Batch processing jobs
    - Data analysis workloads
    - Image/video processing
    - Distributed computing workloads
    - Any workload with a **flexible start and end time**
- **Not suited for**: **Critical jobs or databases** — since sudden termination could cause data loss or service disruption. This distinction (spot = not for critical/stateful workloads) is explicitly called out as testable exam material.

### 11.5 Dedicated Hosts

- **Mechanics**: You reserve an **entire physical server**, with full visibility into and control over instance placement on that specific hardware.
- **Billing**: Available **On-Demand** (billed per second) or **reserved for 1 or 3 years** (for additional discount).
- **Cost**: The **most expensive** EC2 purchasing option overall, since you are paying for exclusive access to physical hardware.
- **Best for**:
    - **Regulatory or compliance requirements** mandating physical isolation of your workloads from other customers.
    - **"Bring Your Own License" (BYOL)** software licensing models that are billed **per socket, per core, or per VM**, which often require visibility into the underlying physical hardware to correctly track and comply with licensing terms.

### 11.6 Dedicated Instances

- **Mechanics**: Instances run on hardware that is **dedicated to your AWS account**, but — critically — this is **different from a Dedicated Host**:
    - You **may still share the underlying physical hardware** with other instances **within your own account**.
    - You have **no visibility or control over the specific instance placement** on that hardware (unlike a Dedicated Host, where you control placement directly).
- **Exam nuance**: The exam is not expected to trick you between Dedicated Instances and Dedicated Hosts, but you should remember the core distinction: **Dedicated Instance = your own instance(s) on hardware not shared with other AWS accounts (but you don't manage the physical server); Dedicated Host = you get the entire physical server itself, with full visibility into its hardware-level details.**

### 11.7 Capacity Reservations

- **Mechanics**: Reserve **On-Demand capacity** in a **specific Availability Zone**, for **any duration** you choose.
- **Commitment**: **No long-term time commitment** — you can create or cancel a capacity reservation at any time.
- **Billing**: **No billing discount** is provided by a Capacity Reservation on its own — you are billed at full **On-Demand rates** whether or not you actually launch instances into the reserved capacity.
- **Combining with discounts**: To get an actual billing discount alongside guaranteed capacity, you must **combine a Capacity Reservation with a Regional Reserved Instance or a Savings Plan**.
- **Best for**: **Short-term, uninterrupted workloads that must run in a very specific Availability Zone**, where guaranteed capacity availability (not cost savings) is the primary concern.

### 11.8 A Helpful Analogy: The Resort/Hotel

To make these purchasing options intuitive, the instructor offers a hotel/resort analogy:

- **On-Demand** = Walking into a resort whenever you like and paying full price for your stay, with no advance planning.
- **Reserved Instances** = Committing in advance to stay at the resort for a long period (1–3 years) in exchange for a much better rate.
- **Savings Plans** = Committing to spend a fixed amount per month at the resort (e.g., "$300/month"), while retaining flexibility to choose different room types (king, suite, sea view) over time.
- **Spot Instances** = Grabbing a last-minute deeply discounted room the resort is trying to fill — but you can be **kicked out at any moment** if someone else is willing to pay more for that room.
- **Dedicated Host** = Booking the **entire building** of the resort for yourself.
- **Capacity Reservation** = Booking a specific room in advance "just in case" you need it — you pay full price for it regardless of whether you actually check in.

### 11.9 Purchasing Option Comparison Summary

|Option|Discount vs. On-Demand|Commitment|Interruption Risk|Best For|
|---|---|---|---|---|
|**On-Demand**|0% (baseline, highest cost)|None|None|Short-term, unpredictable workloads|
|**Reserved Instances**|Up to ~72%|1 or 3 years|None|Steady-state, long-running workloads (e.g., databases)|
|**Convertible Reserved Instances**|Up to ~66%|1 or 3 years|None|Long-running workloads needing future flexibility|
|**Savings Plans**|~72% (similar to RIs)|1 or 3 years ($/hour commitment)|None|Long-term usage with instance-size/OS flexibility|
|**Spot Instances**|Up to ~90%|None|High — can be reclaimed anytime|Fault-tolerant, flexible/batch workloads|
|**Dedicated Hosts**|N/A (most expensive)|On-Demand or 1-3yr reserved|None|Compliance needs, BYOL licensing|
|**Dedicated Instances**|N/A|Varies|None|Hardware isolation from other AWS accounts|
|**Capacity Reservations**|None (On-Demand rate)|None (cancel anytime)|None|Guaranteed capacity in a specific AZ, short-term|

> **Exam Tip:** Expect scenario-based questions describing a workload (e.g., "a workload that can be interrupted at any time and is used for big data analysis" → **Spot Instances**; "a database that must run continuously for the next 3 years" → **Reserved Instances**; "a company with strict compliance requirements needing dedicated physical hardware" → **Dedicated Hosts**). Practice matching workload descriptions to the correct purchasing option.

---

## 12. IPv4 Address Charges (Important Cost Consideration)

### 12.1 The Policy Change

Since **February 1, 2024**, AWS charges for **all Public IPv4 addresses**, whether or not they are actively being used, at a rate of approximately **$0.005 per hour** (roughly **$3.60 per month** per address).

### 12.2 Free Tier Allowance

- For **new AWS accounts**, within the **first 12 months** (the standard AWS Free Tier period), you receive **750 hours per month** of Public IPv4 usage **specifically tied to EC2**.
- This 750-hour allowance is **cumulative across all your EC2 instances' public IPs combined** — meaning you could run 2, 3, or 4 instances simultaneously with public IPs, as long as the **combined total usage** stays under 750 hours per month. Exceeding that combined threshold results in charges for the excess.
- **Beyond EC2**, there is generally **no free tier for Public IPv4 addresses** used by other services.

### 12.3 Services That Also Incur IPv4 Charges

- **Elastic Load Balancers (ELBs)**: typically allocate **one Public IPv4 address per Availability Zone** they operate in. For example, a load balancer spanning **3 AZs** would use **3 Public IPv4 addresses**, all subject to charges with **no free tier** applicable.
- **Amazon RDS (Relational Database Service)**: if you configure a database to be **publicly accessible** (assigning it a Public IPv4 address for direct internet connectivity), this also incurs charges, again with **no free tier**.

### 12.4 The Strategic Reason Behind the Charge

AWS's stated motivation for this charging model is to **encourage broader adoption of IPv6**, which offers a vastly larger address space and is easier to manage at scale than the increasingly scarce pool of IPv4 addresses.

- **IPv6 traffic is not subject to this charge.**
- However, practical adoption of IPv6 is limited by the fact that **many Internet Service Providers (ISPs) worldwide do not yet fully support IPv6**, making a full switch impractical for many learners and organizations.
- You can check your own IPv6 connectivity status using tools like **test-ipv6.com**.
- If choosing to use IPv6 in your own environment, be aware that you are responsible for configuring your **own security group rules and networking decisions** appropriately for IPv6 traffic.

### 12.5 Troubleshooting IPv4 Charges

If you notice unexpected IPv4-related charges on your bill:

1. Go to **Billing and Cost Management → Bills**, and drill into **Charges by Service** to identify where IPv4 charges originate.
2. Use the **AWS Public IP Insights** feature, accessible via the **Amazon VPC IP Address Manager (IPAM)** console (search "IPAM" in the console search bar) — this tool helps you monitor and audit all public IP addresses currently in use across your account/organization.
    - You must first **create an IPAM instance** (this itself is included within the AWS Free Tier) and select which regions to monitor.
    - Once set up, **Public IP Insights** provides visibility into exactly which public IPs are allocated and how they're being used, helping you identify and eliminate unnecessary or forgotten public IP allocations.

> **Exam Tip:** Know that **Public IPv4 addresses now incur an hourly charge regardless of usage** (since February 2024), that EC2 provides a limited **750 hours/month free tier allowance for the first 12 months**, and that this free tier **does not extend** to other services like Elastic Load Balancers or publicly accessible RDS databases. This is a relatively recent, practically important billing change that reflects current AWS policy.

---

## 13. Shared Responsibility Model for EC2

The **Shared Responsibility Model** divides security and operational obligations between AWS and the customer. Applied specifically to EC2:

### AWS is responsible for ("Security _of_ the Cloud"):

- **All data centers and their physical infrastructure and security.**
- **Isolation of the physical host** — for example, ensuring proper isolation when you use Dedicated Hosts.
- **Replacing faulty hardware** if underlying physical servers fail.
- **Maintaining compliance** with the regulatory frameworks and certifications AWS has committed to.

### You (the customer) are responsible for ("Security _in_ the Cloud"):

- **Defining your own security group rules**, controlling exactly who and what can access your EC2 instances.
- **The entire guest operating system** running inside your instance — whether Windows or Linux — including **all patches and updates**. AWS provides the virtual machine; you are responsible for keeping its OS current and secure.
- **All software and utilities installed on the instance** are entirely your responsibility to manage, configure, and secure.
- **Correctly configuring and assigning IAM Roles**, ensuring permissions granted to the instance are appropriate and follow the principle of least privilege.
- **Ensuring data security** on your instance — encryption, backups, access control for any data stored or processed on the instance.

> **Exam Tip:** A simple mental model for EC2 specifically: **AWS secures the physical data center, hardware, and host-level isolation; you secure everything from the guest operating system upward** — patches, software, firewall rules (security groups), IAM role assignment, and data protection. This division is one of the most consistently tested concepts across the entire CLF-C02 exam, and EC2 is one of the clearest services to illustrate it with concretely, since you have such deep, hands-on control over the guest OS layer.

---

## 14. Comprehensive EC2 Summary

Bringing every concept together into one cohesive picture:

- **EC2 (Elastic Compute Cloud)** is AWS's core **IaaS** offering, letting you rent virtual servers ("instances") with configurable OS, CPU, memory, storage, and networking.
- An EC2 instance is defined by an **AMI** (its OS/image), an **instance type** (its compute/memory/network profile), **storage** (network-attached EBS/EFS or hardware-attached instance store), a **security group** (its firewall), and optionally an **EC2 User Data script** (a one-time bootstrap script executed at first launch, run as root).
- **Security Groups** are stateful, allow-only virtual firewalls that live outside the instance itself, can reference IP ranges or other security groups, default to blocking all inbound and allowing all outbound traffic, and are scoped per Region/VPC.
- **Connecting to an instance** can be done via native **SSH** (macOS/Linux/Windows 10+), **PuTTY** (older Windows versions), or the browser-based **EC2 Instance Connect** (currently limited to Amazon Linux 2-type AMIs) — all fundamentally relying on the SSH protocol over port 22.
- **IAM Roles**, not hardcoded access keys, are the correct and secure way to grant an EC2 instance permission to call other AWS services.
- **Purchasing options** — On-Demand, Reserved Instances (standard and convertible), Savings Plans, Spot Instances, Dedicated Hosts, Dedicated Instances, and Capacity Reservations — offer different trade-offs between cost savings, flexibility, and commitment, matched to different workload profiles (short-term/unpredictable vs. steady-state/long-term vs. interruption-tolerant vs. compliance-driven).
- **Public IPv4 addresses now carry an hourly charge** (since February 2024), with a limited free-tier allowance specifically for EC2 in a new account's first 12 months; other services (like ELB and public RDS) have no such free tier.
- The **Shared Responsibility Model** for EC2 places physical infrastructure, hardware isolation, and hardware failure recovery on AWS, while the guest OS, patching, software, security group configuration, IAM role assignment, and data security remain the customer's responsibility.
- Practical troubleshooting heuristics — such as **timeout = security group issue** vs. **connection refused = application issue**, and the fact that **public IPs can change on stop/start while private IPs remain stable** — are frequently tested, real-world-relevant details.

Mastering EC2 at this level of detail provides a strong foundation not only for direct EC2 exam questions, but for understanding related services built on top of it — Elastic Load Balancing, Auto Scaling Groups, EBS/EFS storage, and VPC networking — all of which are natural extensions of the concepts introduced here.