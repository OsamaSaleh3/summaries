
## Executive Overview

Amazon Virtual Private Cloud (VPC) is the networking foundation of AWS. It is the private, isolated section of the AWS Cloud where you deploy resources such as EC2 instances, RDS databases, and Lambda functions (when placed inside a VPC). Understanding VPC is critical because **every single resource you launch on AWS lives inside some kind of network** — and how that network is designed determines whether your resources are reachable from the internet, how they talk to each other, how secure they are, and how they connect back to your own on-premises data center.

For the **AWS Certified Cloud Practitioner (CLF-C02)** exam, VPC & Networking is a **light but broad** topic. According to the instructor, it typically represents only **one or two exam questions**, but those questions can touch on any of the many sub-services under the VPC umbrella. The exam does not expect deep architectural mastery (that's reserved for the Solutions Architect Associate and SysOps Administrator Associate certifications) — instead, it expects you to:

- Recognize each networking service/component by name.
- Understand **what problem it solves** at a high level.
- Be able to **match a described scenario to the correct service** (e.g., "connect thousands of VPCs together" → Transit Gateway).

This guide walks through every concept covered in the video series — IP addressing, VPCs and subnets, Internet Gateways, NAT Gateways, Security Groups, Network ACLs, VPC Flow Logs, VPC Peering, VPC Endpoints, PrivateLink, Site-to-Site VPN, Client VPN, Direct Connect, and Transit Gateway — and expands on each with the technical depth needed to fully understand it, even where the transcript only mentions it briefly.

---

## 1. IP Addressing Fundamentals in AWS

Before diving into VPCs, it's essential to understand the types of IP addresses used in AWS networking.

### 1.1 IPv4 (Internet Protocol version 4)

- The traditional and most familiar IP protocol.
- Offers a total address space of about **4.3 billion addresses** (2³²).
- IPv4 addresses in AWS come in two flavors:

#### Public IPv4

- **Publicly reachable from the internet** — anyone on the internet can attempt to reach a resource that has one.
- When you launch an EC2 instance in a public subnet, AWS can automatically assign it a public IPv4 address.
- **Key behavior:** A standard (auto-assigned) public IPv4 address is **not fixed**. If you **stop** the EC2 instance, the public IP is **released back to AWS**. When you **start** the instance again, it receives a **brand-new public IPv4 address**.
- This matters operationally: any DNS records, whitelist entries, or scripts hardcoded to that IP will break after a stop/start cycle.

#### Private IPv4

- Used only for communication **inside a private network**, such as your AWS VPC (e.g., `192.168.1.1` or the `172.31.x.x` range used by AWS default VPCs).
- **Not reachable from the public internet** — you cannot access these addresses from your home browser.
- **Key behavior:** The private IPv4 address assigned to an EC2 instance is **fixed for the lifetime of that instance** — it does **not** change even if you stop and restart the instance (unlike the public IP).

### 1.2 Elastic IP (EIP)

- An Elastic IP is a way to attach a **fixed/static public IPv4 address** to an EC2 instance.
- Unlike the auto-assigned public IP, an Elastic IP **persists** even if you stop and start the instance — the instance keeps the same public IP.
- **Use case:** Useful when you need a stable, unchanging public IP address (e.g., for DNS records, firewall whitelisting, or failover scenarios where you re-map the EIP to a different instance).
- **Cost caveat:** If you allocate an Elastic IP but leave it **unattached to a running instance**, or attached to a **stopped** instance, AWS considers this a **waste of a public IP address** — you are charged for it and get no benefit, since the instance isn't reachable while stopped.

> **Exam Tip:** Elastic IPs are one of the classic "hidden cost" traps on the exam. Remember: an idle/unassociated Elastic IP still incurs an hourly charge.

### 1.3 Public IPv4 Pricing

- As of the pricing model discussed, **every public IPv4 address in AWS is charged at $0.005 per hour** — this includes both:
    - Standard auto-assigned public IPv4 addresses, **and**
    - Elastic IP addresses.
- **AWS Free Tier** includes **750 hours per month** of public IPv4 usage at no cost, giving new users room to experiment.
- This pricing change (introduced by AWS in 2024) is explicitly designed to **nudge customers toward adopting IPv6**, which is free.

> **Exam Tip:** Know that public IPv4 addresses are _not_ free at scale, while private IPv4 addresses have no direct charge. This cost differentiation itself can be an exam distinction.

### 1.4 IPv6 (Internet Protocol version 6)

- The newer generation of the Internet Protocol, designed to solve IPv4 address exhaustion.
- Massive address space: approximately **3.4 × 10³⁸ addresses** (i.e., a number with 38 zeros) — practically unlimited compared to IPv4.
- **Critical distinction:** In AWS, **every IPv6 address is public** — there is **no concept of a "private" IPv6 range** like there is with IPv4's `192.168.x.x` or `172.31.x.x`.
- **IPv6 addresses are free in AWS** (no hourly charge like public IPv4).
- **Use case:** If you want to expose services to the internet at scale without paying the public IPv4 hourly fee, IPv6 is the strategic choice AWS is pushing customers toward.

|Feature|IPv4|IPv6|
|---|---|---|
|Address space|~4.3 billion|~3.4 × 10³⁸|
|Public/Private split|Yes (public vs private ranges)|No — all addresses are public|
|Cost in AWS|$0.005/hour per public IPv4|Free|
|Legacy support|Universal|Growing but not universal|

---

## 2. VPC (Virtual Private Cloud) Fundamentals

### 2.1 What is a VPC?

- **VPC = Virtual Private Cloud.**
- It is your own **private, isolated network** within AWS where you deploy resources like EC2 instances, RDS databases, Lambda functions (with VPC configuration), etc.
- **A VPC is tied to a specific AWS Region.** If you operate across multiple regions, you need a separate VPC in each region (they are regionally scoped resources).
- Every AWS account comes with a **Default VPC** already created in each region, pre-configured with public subnets, an Internet Gateway, and route tables — ready to use immediately without any manual setup.

### 2.2 CIDR Blocks

- When you create a VPC, you define a **CIDR (Classless Inter-Domain Routing) block** — this specifies the **range of IP addresses** available within that VPC.
- Example seen in the default VPC: `172.31.0.0/16`
    - This CIDR notation means: the first 16 bits of the address are fixed (`172.31`), and the remaining bits can vary, producing a large pool of usable addresses (~65,000 IPs in a `/16` block).
- Tools like **cidr.xyz** can be used to visualize a CIDR range and calculate the first/last usable IP addresses in that block.
- A VPC can be assigned more than one CIDR block if needed (secondary CIDR blocks) to expand its available address space.

> **Deep Dive:** CIDR notation `X.X.X.X/N` — the `/N` indicates how many bits are "fixed" as the network prefix. A smaller `N` (e.g., `/16`) means a larger address range; a larger `N` (e.g., `/24` or `/28`) means a smaller, more restricted range. This is foundational to how subnets are carved out of a VPC's total IP space.

### 2.3 Subnets

- A **subnet** is a **partition of your VPC's network** — essentially, a smaller slice of the overall VPC CIDR range.
- **Critical rule: A subnet is tied to exactly one Availability Zone (AZ).** It cannot span multiple AZs.
- Each subnet gets its **own CIDR block**, which is a subset of the VPC's overall CIDR range (e.g., a VPC with `/16` might have subnets carved out as `/20` blocks).
- In the default VPC example shown, there were **three subnets**, one per AZ (e.g., `eu-west-1a`, `eu-west-1b`, `eu-west-1c`), each with **4,091 available IPv4 addresses** in a `/20` block.
- When you launch an EC2 instance, you place it into a specific subnet, and it consumes one IP address from that subnet's available pool.

#### Public Subnet vs. Private Subnet

|Type|Definition|Typical Resources|
|---|---|---|
|**Public Subnet**|A subnet that **has a route to an Internet Gateway**, making it directly reachable from (and able to reach) the internet|Web servers, load balancers, bastion hosts|
|**Private Subnet**|A subnet that has **no direct route to an Internet Gateway** — not reachable from the internet|Databases, internal application servers, backend services|

- A subnet becomes "public" **not by any inherent flag**, but purely because its associated **Route Table** contains a route directing internet-bound traffic (`0.0.0.0/0`) to an Internet Gateway.
- **Why use private subnets?** Placing sensitive resources like databases in a private subnet means they cannot be directly attacked or accessed from the public internet — a core security best practice (defense in depth).

### 2.4 Route Tables

- A **Route Table** defines how network traffic is directed — both **between subnets inside the VPC** and **out to the internet or other networks**.
- Every subnet is associated with a route table.
- Example of the default route table's logic (as seen in the console):
    - Traffic destined for the VPC's own CIDR range → stays **local** within the VPC.
    - Traffic destined for anywhere else (`0.0.0.0/0`) → routed to the **Internet Gateway**.
- This is the actual mechanism that determines whether a subnet behaves as "public" or "private."

### 2.5 A Complete VPC Architecture (Conceptual Diagram)

```
AWS Region
 └── VPC (CIDR: e.g. 10.0.0.0/16)
      ├── Availability Zone 1
      │     ├── Public Subnet 1  → Route to Internet Gateway
      │     └── Private Subnet 1 → Route to NAT Gateway
      └── Availability Zone 2
            ├── Public Subnet 2  → Route to Internet Gateway
            └── Private Subnet 2 → Route to NAT Gateway
```

- A well-architected VPC typically spans **2 or more Availability Zones** for high availability, with each AZ containing at least one public and one private subnet.
- This design pattern (public + private subnet per AZ) is the standard, exam-relevant reference architecture.

### Use Cases & Benefits of VPC

- **Isolation:** Your VPC is logically isolated from every other customer's VPC on AWS.
- **Control:** You fully control IP ranges, subnetting, routing, and connectivity.
- **Security layering:** Combine public/private subnet design with Security Groups and NACLs for defense in depth.
- **Hybrid connectivity:** VPCs can be connected back to on-premises data centers (via VPN or Direct Connect) to extend your existing network into the cloud.

---

## 3. Internet Gateway (IGW)

- An **Internet Gateway** is a horizontally scaled, redundant, and highly available VPC component that allows communication between instances in your VPC and the internet.
- **A VPC can only have one Internet Gateway attached to it at a time.**
- Simply attaching an IGW to a VPC is not enough — a **route** must also exist in the subnet's route table pointing internet-bound traffic (`0.0.0.0/0`) to that Internet Gateway.
- **This is precisely what defines a "public subnet":** IGW attached to the VPC **+** a route from the subnet's route table to that IGW.
- If the Internet Gateway is **detached** from the VPC, instances in previously "public" subnets **immediately lose internet access** (both inbound and outbound).

> **Exam Tip:** Internet Gateway = internet access **for public subnets**. It requires both (1) the gateway attached to the VPC, and (2) the route table entry. Missing either one breaks internet connectivity.

---

## 4. NAT Gateway & NAT Instances

### 4.1 The Problem

Instances in a **private subnet** have no direct internet access (by design, for security). But sometimes those instances still need **outbound-only** internet access — for example, to:

- Download operating system security patches/updates.
- Download software packages or files.
- Call out to external APIs.

...while still remaining **unreachable from the internet** (no inbound connections allowed).

### 4.2 The Solution: NAT (Network Address Translation)

There are two ways to solve this:

|Option|Description|Management|
|---|---|---|
|**NAT Gateway**|An **AWS-managed** service that provides NAT functionality|Fully managed by AWS (scaling, patching, availability)|
|**NAT Instance**|An EC2 instance configured to perform NAT|**Self-managed** by the customer (you patch, scale, and maintain it)|

### 4.3 How It Works

- The NAT Gateway (or NAT instance) is deployed **inside a public subnet** (since it needs its own route to the Internet Gateway).
- A route is then created in the **private subnet's** route table, sending internet-bound traffic to the NAT Gateway.
- The NAT Gateway then forwards that traffic out through the **Internet Gateway**.
- Return traffic follows the reverse path — but the internet cannot **initiate** a connection into the private subnet; only responses to requests originating from inside are allowed.

```
Private Subnet Instance → NAT Gateway (in Public Subnet) → Internet Gateway → Internet
```

### Use Cases & Benefits

- Lets private, security-sensitive resources (e.g., application servers, databases needing patch downloads) reach the internet for **outbound** purposes only, without exposing them to inbound internet traffic.
- **NAT Gateway is preferred over NAT Instance in most real-world/production scenarios** because it's managed, highly available within an AZ, and automatically scales — though this comparison goes deeper in the Solutions Architect-level content. At the Cloud Practitioner level, just know: NAT Gateway = managed, NAT Instance = self-managed EC2-based alternative.

> **Exam Tip:** If a scenario describes "private subnet needs outbound internet access without being exposed publicly," the answer is a NAT Gateway (or NAT instance).

---

## 5. Network Security: Security Groups vs. Network ACLs (NACL)

Network security within a VPC operates at **two different layers**, forming a defense-in-depth model.

### 5.1 Network ACL (NACL) — First Line of Defense

- A **Network ACL** is a **firewall operating at the subnet level**.
- It controls traffic **entering and leaving an entire subnet**, before that traffic ever reaches individual instances.
- NACL rules:
    - Can be **Allow** rules **and Deny** rules.
    - Can only reference **IP addresses/CIDR ranges** (not security groups).
    - Are evaluated **in order by rule number** (lowest number evaluated first).
- **NACLs are stateless:** this means that if you allow inbound traffic on a port, you must **explicitly** also allow the corresponding outbound **return traffic** — the NACL doesn't automatically remember/permit the reply.
- The **Default NACL** (created automatically with every VPC) allows **all inbound and all outbound traffic** by default — it's wide open until you customize it.
- A NACL is associated with **one or more subnets** (in the demo, one default NACL was associated with all three subnets in the default VPC).

### 5.2 Security Groups — Second Line of Defense

- A **Security Group** is a **firewall operating at the EC2 instance level** (more precisely, at the **Elastic Network Interface / ENI level**).
- Security Group rules:
    - Can **only contain Allow rules** — there is **no explicit "deny" rule** in a security group (anything not explicitly allowed is implicitly denied).
    - Can reference **IP addresses/CIDR ranges** _and_ **other Security Groups** (allowing traffic from any instance that belongs to a referenced security group — very useful for tiered architectures, e.g., "allow traffic from the web-tier security group only").
- **Security Groups are stateful:** if you allow inbound traffic on a given port, the **response/return traffic is automatically allowed** out, regardless of outbound rules. You do not need to create a matching outbound rule for replies.
- Example from the demo (`launch-wizard-1` security group):
    - **Inbound:** Allow HTTP (port 80) and SSH (port 22) from anywhere (`0.0.0.0/0`).
    - **Outbound:** Allow all traffic, all ports, all protocols, to anywhere — giving the instance unrestricted ability to reach out to the internet.

### 5.3 Key Comparison Table

|Feature|Security Group|Network ACL (NACL)|
|---|---|---|
|**Operates at**|EC2 instance / ENI level|Subnet level|
|**Rule types**|Allow rules **only**|Allow **and** Deny rules|
|**State**|**Stateful** (return traffic automatically allowed)|**Stateless** (return traffic must be explicitly allowed)|
|**Rule targets**|IP addresses **and** other Security Groups|IP addresses only|
|**Rule evaluation**|All rules evaluated together|Evaluated in numbered order|
|**Scope**|Attached directly to instances/ENIs|Applies to **all** instances within the associated subnet(s)|

> **Exam Tip:** This is one of the **most commonly tested VPC concepts** even at the Cloud Practitioner level. Memorize at minimum:
> 
> 1. Security Group = instance level, Stateful, Allow rules only.
> 2. NACL = subnet level, Stateless, Allow AND Deny rules.
> 3. Traffic to an EC2 instance passes through the NACL **first** (subnet boundary), then the Security Group (instance boundary).

### Use Cases & Benefits

- **NACLs** are useful for **broad, subnet-wide restrictions** — e.g., explicitly blocking a known malicious IP range from an entire subnet, or as a coarse-grained additional security layer.
- **Security Groups** are the primary, day-to-day mechanism for controlling exactly which traffic can reach specific instances (e.g., "only allow HTTPS from the load balancer's security group").

---

## 6. VPC Flow Logs

- **VPC Flow Logs** capture **metadata about IP traffic** flowing through the network interfaces in your VPC.
- Flow logs can be created at **three different scopes**:
    1. **VPC Flow Log** — captures traffic for the entire VPC.
    2. **Subnet Flow Log** — captures traffic for a specific subnet.
    3. **Elastic Network Interface (ENI) Flow Log** — captures traffic for a single network interface (e.g., attached to one EC2 instance).
- **Use cases:**
    - **Monitoring** network traffic patterns.
    - **Troubleshooting connectivity issues** — e.g., diagnosing why a subnet cannot reach the internet, why one subnet cannot reach another subnet, or why external traffic cannot reach a subnet.
- Flow logs can capture traffic information not just for EC2, but also for other VPC-attached services such as **Elastic Load Balancers, ElastiCache, RDS, and Aurora**.
- **Flow log destinations** — where the captured logs are sent:
    - **Amazon S3**
    - **CloudWatch Logs**
    - **Amazon Data Firehose** (for streaming/real-time processing pipelines)
- **Configurable options when creating a flow log:**
    - **Filter:** capture _all_ traffic, only _accepted_ traffic, or only _rejected_ traffic.
    - **Maximum aggregation interval:** e.g., 1-minute or 10-minute intervals.
    - **Destination-specific settings:** e.g., for CloudWatch Logs, you must specify a **Log Group** and an **IAM Role** granting permission to write logs.
- **Log record fields** captured include: version, account ID, interface ID, source address, destination address, source port, destination port, protocol, number of packets, number of bytes, start time, end time, action (accept/reject), and log status.

> **Exam Tip:** VPC Flow Logs = network traffic visibility/troubleshooting tool. If a scenario asks "how can I diagnose why traffic isn't reaching my instance," think VPC Flow Logs.

---

## 7. VPC Peering

- **VPC Peering** creates a **private network connection between two VPCs**, allowing them to communicate **as if they were part of the same network**, using AWS's own internal network infrastructure (traffic never traverses the public internet).
- **Requirement:** The two VPCs being peered **must have non-overlapping CIDR ranges**. If their IP ranges overlap, a peering connection **cannot** be established (routing would be ambiguous).
- VPC Peering can connect:
    - VPCs within the **same AWS account** or **different accounts**.
    - VPCs in the **same region** or **different regions**.
- **Critical limitation — VPC Peering is NOT transitive:**
    - If VPC A is peered with VPC B, and VPC B is peered with VPC C, this does **not** automatically allow **VPC A to communicate with VPC C**.
    - To enable that, you would need to create a **separate, direct peering connection** explicitly between VPC A and VPC C.

```
VPC A ⟷ VPC B    (A and B can talk)
VPC B ⟷ VPC C    (B and C can talk)
VPC A  ✕  VPC C  (A and C CANNOT talk — peering is non-transitive)
```

- **Setup process (as demonstrated):** name the peering connection, select a "requester" (local) VPC, then select the "accepter" VPC (which can be in the same or a different account/region, referenced by VPC ID). The connection must then be **accepted** by the owner of the second VPC before traffic can flow.

> **Exam Tip:** Two defining facts to remember: (1) non-overlapping CIDRs required, (2) **not transitive**. Both are classic exam traps.

### Use Cases & Benefits

- Connecting a small, limited number of VPCs together privately (e.g., a shared-services VPC to an application VPC).
- **Note:** As the number of VPCs needing interconnection grows, peering becomes unmanageable (since you'd need a peering connection for every pair of VPCs) — this is precisely the problem that **Transit Gateway** (covered later) solves.

---

## 8. VPC Endpoints

### 8.1 The Problem

By default, when your EC2 instances talk to AWS services like S3 or DynamoDB, that traffic travels over the **public internet** (or at least AWS's public-facing endpoints), even though it never technically "leaves" AWS's backbone in a literal sense from the user's perspective — the point is it uses **public API endpoints**.

### 8.2 The Solution

- A **VPC Endpoint** allows you to connect to supported AWS services using AWS's **private internal network** instead of the public internet path.
- **Benefits:**
    - **Better security** — traffic doesn't traverse the public internet.
    - **Lower latency** — traffic doesn't have to pass through additional public network hops/hubs.

### 8.3 Two Types of VPC Endpoints

|Type|Supported Services|How it Works|
|---|---|---|
|**Gateway Endpoint**|**Only Amazon S3 and DynamoDB**|Acts as a target in your route table; traffic is routed privately to the service|
|**Interface Endpoint**|**Virtually every other AWS service** (e.g., CloudWatch, EC2 API, SNS, SQS, etc.)|Creates an **Elastic Network Interface (ENI)** with a private IP inside your subnet, which your resources connect to|

> **Exam Tip (explicitly emphasized by the instructor):** Remember that **almost every AWS service supports an Interface Endpoint**, but **only Amazon S3 and DynamoDB support a Gateway Endpoint** (and in S3's case, it also offers an Interface Endpoint option as an alternative). This S3/DynamoDB-as-gateway-endpoint fact is a very common exam question.

### 8.4 Demo Walkthrough Recap

- To create an endpoint: VPC Console → **PrivateLink and Lattice** → **Endpoints** → **Create Endpoint**.
- Choose "AWS services" as the type, then search/select the specific service (e.g., CloudWatch, EC2, S3, DynamoDB).
- For an Interface Endpoint, you must specify **which VPC** and **which subnets** it will be created in.
- Example use case shown: an EC2 instance in a private subnet pushing a **custom metric to CloudWatch** — this requires a CloudWatch **Interface Endpoint**, since CloudWatch is not one of the two gateway-endpoint-eligible services.

### Use Cases & Benefits

- Keep all traffic to AWS services **private** — critical for regulated industries, compliance requirements, or simply as a security best practice.
- Reduce data transfer costs and latency associated with routing through NAT Gateways/Internet Gateways just to reach AWS's own services.

---

## 9. AWS PrivateLink

- **AWS PrivateLink** is part of the broader **VPC Endpoint Services** family, but it solves a different problem: **privately and securely exposing a service that runs inside one VPC to consumers in other, different VPCs** (potentially belonging to different AWS accounts entirely).

### 9.1 The Scenario

- Imagine a **third-party vendor** (e.g., found on the **AWS Marketplace**) runs a SaaS-style application/service inside **their own VPC**, in **their own AWS account**.
- Thousands of **customers** (each with their own VPC) want **private, secure access** to that vendor's service — without exposing traffic over the public internet.

### 9.2 Why Not Just Use VPC Peering?

- VPC Peering technically _could_ connect the two VPCs, but:
    - **It doesn't scale** — imagine the vendor needing a separate peering connection for every one of their thousands of customers.
    - **It's less secure** — peering grants broader network-level access between the two VPCs (full IP range reachability, subject to route tables/security groups), rather than exposing just a single specific service endpoint.

### 9.3 How PrivateLink Works

1. The **service provider** (vendor) creates a **Network Load Balancer (NLB)** in front of their application to expose the service.
2. The **service consumer** (customer) creates an **Elastic Network Interface (ENI)** inside their own VPC/subnet.
3. A **PrivateLink connection** (VPC Endpoint Service on the provider side, VPC Endpoint on the consumer side) is established between the two.
4. All traffic flows over AWS's **private network** — **never** over the public internet, and **without requiring**:
    - VPC Peering
    - An Internet Gateway
    - NAT Gateways
    - Complex route table configuration

```
Consumer VPC (ENI)  ⟷  PrivateLink  ⟷  Vendor VPC (Network Load Balancer → Service)
```

5. **Scalability:** For each new customer, the vendor simply provisions a new PrivateLink connection endpoint — a lightweight, repeatable, easily manageable process, unlike scaling out individual peering connections.

> **Exam Tip:** If the scenario describes a **SaaS vendor exposing a service privately to many customer VPCs at scale**, the answer is **AWS PrivateLink** — not VPC Peering.

### Use Cases & Benefits

- SaaS providers exposing services to customers securely and at scale.
- Enterprises exposing internal shared services (e.g., a central logging or authentication service) to multiple internal VPCs privately.
- Avoids the security exposure and non-scalability of full VPC peering when only **one specific service** needs to be shared, not entire network reachability.

---

## 10. Hybrid Connectivity: Connecting On-Premises to AWS

This section covers connecting your **own physical/on-premises data center** to your AWS VPC — a foundational concept for "hybrid cloud" architectures.

### 10.1 Site-to-Site VPN

- A **Site-to-Site VPN** establishes an **encrypted connection** between your on-premises data center and your AWS VPC, **over the public internet**.
- **Key characteristics:**
    - **Fast to set up** — can be established in as little as **~5 minutes**.
    - Travels over the **public internet**, so:
        - **Limited/variable bandwidth** (dependent on your internet connection).
        - Some inherent security concerns (though the traffic itself **is encrypted**, so it remains protected in transit).
- **Required components:**
    - **Customer Gateway (CGW):** A resource representing **your side** — your on-premises VPN device/router. This is configured **on-premises**.
    - **Virtual Private Gateway (VGW):** The AWS-side VPN concentrator, attached to your VPC.
    - Once both the CGW and VGW are provisioned, you connect them together to establish the Site-to-Site VPN tunnel.

```
On-Premises Data Center ⟷ [Customer Gateway (CGW)] ⟷ Public Internet (encrypted) ⟷ [Virtual Private Gateway (VGW)] ⟷ AWS VPC
```

> **Exam Tip:** Memorize the two named components: **Customer Gateway (CGW)** = on-premises side, **Virtual Private Gateway (VGW)** = AWS side. This pairing is a frequently tested exam fact.

### 10.2 AWS Direct Connect (DX)

- **Direct Connect** establishes a **dedicated, private, physical network connection** between your on-premises data center and AWS — bypassing the public internet entirely.
- **Key characteristics:**
    - **Private, secure, and fast** — traffic travels over a dedicated private line, not shared public infrastructure.
    - **More expensive** than Site-to-Site VPN, since it requires physical cabling/circuits established through a **Direct Connect partner location**.
    - **Slow to establish** — typically takes **at least one month** to provision, due to the physical infrastructure work involved.
    - More **reliable and consistent** performance (predictable bandwidth/latency) compared to a VPN over the public internet.

### 10.3 Choosing Between Site-to-Site VPN and Direct Connect

The instructor frames the decision around **two key questions**:

1. **Does the connection need to be private (i.e., not traverse the public internet)?**
2. **Does the connection need to be established quickly, or is time not a constraint?**

|Factor|Site-to-Site VPN|Direct Connect|
|---|---|---|
|**Path**|Over the public internet (encrypted)|Dedicated private physical line|
|**Setup time**|~5 minutes|~1 month or more|
|**Cost**|Lower|Higher|
|**Bandwidth/Reliability**|Variable, internet-dependent|Consistent, high-bandwidth, reliable|
|**Best for**|Quick setup, backup/failover connectivity, lower-budget needs|Mission-critical, high-bandwidth, low-latency, sustained hybrid workloads|

> **Exam Tip:** A scenario emphasizing **speed of setup** or **temporary/backup connectivity** → **Site-to-Site VPN**. A scenario emphasizing **maximum privacy, consistent low latency, and high sustained bandwidth**, where setup time is not a constraint → **Direct Connect**. Often, VPN is also used as a **backup connection** for Direct Connect in real architectures.

### 10.4 AWS Client VPN

- **Client VPN** allows an **individual person's computer** to establish a **private connection directly into a VPC** (or even into an on-premises network reachable through that VPC), using the **OpenVPN** protocol.
- **Use case:** You have EC2 instances deployed in a **private subnet**, and you (as an individual user/developer) need to access them using their **private IP address** — normally impossible without being physically inside the VPC's network. Client VPN solves this by making your laptop behave as if it's a device sitting inside the VPC network.
- **How it works:**
    1. You install/configure a Client VPN connection on your personal computer.
    2. This connection is established **over the public internet**, but the tunnel itself is encrypted.
    3. Once connected, your computer behaves as though it is part of the VPC's private network — able to reach private IP addresses directly.
- **Extended reach:** If your VPC **also** has a Site-to-Site VPN connection established to an on-premises data center, then your Client VPN-connected computer can **also** privately reach servers in that **on-premises data center** — effectively chaining the two private connections together.

```
Your Computer → [Client VPN / OpenVPN, over public internet] → VPC (private access)
                                                                      ↓ (if Site-to-Site VPN also exists)
                                                              On-Premises Data Center
```

> **Exam Tip:** Distinguish Client VPN (connects an **individual user's device** to a VPC) from Site-to-Site VPN (connects an **entire on-premises network/data center** to a VPC).

---

## 11. Transit Gateway

### 11.1 The Problem It Solves

As an organization's AWS footprint grows, they may end up with:

- Dozens, hundreds, or even **thousands of VPCs**.
- Multiple **Site-to-Site VPN** connections.
- Multiple **Direct Connect** connections.

If you tried to connect all of these together using **VPC Peering** alone, you would need a **separate peering connection for every single pair** of VPCs that need to communicate — and remember, peering is **non-transitive**, so there's no shortcut. This creates an unmanageable **mesh of point-to-point connections** — often visualized as a tangled "spaghetti" network topology.

### 11.2 The Solution: Transit Gateway

- **Transit Gateway** is a **central hub** that allows you to connect **thousands of VPCs, VPN connections, and Direct Connect gateways together**, using a **"hub-and-spoke" (star) topology**.
- Instead of each VPC/connection needing individual point-to-point links to every other VPC/connection, **everything connects to the single Transit Gateway** in the middle, and the Transit Gateway handles routing between all of them.

```
                 ┌─────────────┐
   VPC A  ────── │             │ ────── VPC D
                 │             │
   VPC B  ────── │  Transit    │ ────── Site-to-Site VPN
                 │  Gateway    │
   VPC C  ────── │             │ ────── Direct Connect Gateway
                 └─────────────┘
```

- **What it connects:**
    - **Amazon VPCs**
    - **VPN connections** (Site-to-Site VPN)
    - **Direct Connect Gateways**

### 11.3 Key Benefits

- Eliminates the need for individual VPC peering connections between every pair of VPCs.
- Eliminates the need for separate, redundant route configurations between every VPC and every VPN/Direct Connect connection.
- Massively simplifies network topology and management at scale.
- Functions as **one single gateway** through which all this connectivity is managed.

> **Exam Tip:** Whenever an exam scenario describes needing to connect **"hundreds or thousands of VPCs"** together, along with on-premises infrastructure, **the answer is Transit Gateway** — this is explicitly called out by the instructor as the trigger phrase to watch for.

---

## 12. Full Summary — Putting It All Together

The table below consolidates every service/concept from this section, matching the instructor's own summary lecture:

|Concept|What It Is|Key Point to Remember|
|---|---|---|
|**VPC**|Virtual Private Cloud — your private network in AWS|Regional resource; contains subnets|
|**Subnet**|A network partition within a VPC|Tied to **one** Availability Zone|
|**Internet Gateway**|Enables internet access for a VPC|Only **one** per VPC; needed for public subnets|
|**NAT Gateway / NAT Instance**|Gives private subnets outbound-only internet access|Gateway = managed; Instance = self-managed|
|**Network ACL (NACL)**|Stateless firewall at the **subnet** level|Allow **and** Deny rules|
|**Security Group**|Stateful firewall at the **instance/ENI** level|**Allow rules only**|
|**VPC Peering**|Private connection between two VPCs|Non-overlapping CIDRs required; **non-transitive**|
|**Elastic IP**|Fixed/static public IPv4|Ongoing cost if unused/idle|
|**VPC Endpoint**|Private access to AWS services from within a VPC|Gateway type = **S3 & DynamoDB only**; Interface type = almost everything else|
|**PrivateLink**|Private connection to a service in a _third-party_ VPC|Solves scaling problem of exposing a service to many consumer VPCs|
|**VPC Flow Logs**|Logs of network traffic|Sent to S3, CloudWatch Logs, or Data Firehose|
|**Site-to-Site VPN**|Encrypted connection, on-premises ⟷ AWS, over public internet|Needs Customer Gateway (CGW) + Virtual Private Gateway (VGW); fast setup|
|**Client VPN**|Connects an individual computer privately into a VPC|Uses OpenVPN, over the public internet|
|**Direct Connect (DX)**|Dedicated private physical connection, on-premises ⟷ AWS|Private, fast, reliable, but expensive and slow to set up (~1 month)|
|**Transit Gateway**|Central hub-and-spoke connector for many VPCs + VPN + Direct Connect|Use when connecting **thousands** of networks together|

---

## 13. Hands-On: Exploring the Default VPC

The instructor walked through the AWS Console to inspect the **default VPC** that AWS automatically provisions in every account/region. Key observations from the walkthrough:

1. **VPC Console overview** showed: **1 VPC, 3 subnets, 1 Route Table, 1 Internet Gateway** — the standard default VPC setup.
2. **VPC details:** The default VPC had a CIDR of `172.31.0.0/16`, giving roughly 65,000 usable IP addresses across the whole VPC.
3. **Subnets:** Three subnets existed, one per Availability Zone in the region (e.g., `eu-west-1a`, `eu-west-1b`, `eu-west-1c`), each with its own smaller CIDR (e.g., `/20`, giving 4,091 usable IPs per subnet).
4. **Launching an EC2 instance** into one of these subnets (e.g., `eu-west-1a`) resulted in the instance receiving a private IP from within that subnet's CIDR range — confirming the relationship between subnet CIDR and instance IP allocation.
5. **Internet Gateway:** Confirmed attached to the VPC, and confirmed that the subnet's Route Table had a route pointing non-local traffic to that Internet Gateway — this combination is what made the subnets "public," and explains why the EC2 instances launched earlier in the course were internet-accessible (used as web servers, able to install packages, etc.).
6. **Security Groups** (viewed from the VPC console, same underlying resource as seen from the EC2 console): the `launch-wizard-1` group allowed inbound HTTP (80) and SSH (22) from anywhere, and all outbound traffic.
7. **Network ACL:** One default NACL existed, associated with all three subnets, allowing all inbound and all outbound traffic by default (the "wide open" default state).
8. **No private subnets or NAT Gateways** existed in the default VPC — since AWS designs the default VPC to be simple and immediately internet-accessible for ease of onboarding; a private subnet + NAT Gateway setup requires manual configuration.
9. **Cleanup reminder:** Always terminate any EC2 instances created for practice/demo purposes to avoid ongoing charges.

> **Exam Tip:** Know that the **default VPC** in each region is pre-configured with an Internet Gateway and public subnets in every AZ of that region — making it the easiest way to get started, but not necessarily a security best practice for production workloads (which typically use custom VPCs with proper public/private subnet segmentation).

---

## 14. Consolidated Exam Tips

- **VPC & Networking is roughly 1–2 questions on the CLF-C02 exam** — don't over-invest study time relative to core services like EC2, S3, and IAM, but do know every term in this guide by name and function.
- **Subnets are AZ-specific**, VPCs are region-specific.
- **Public subnet = Internet Gateway attached to VPC + route table entry pointing to it.**
- **NAT Gateway/Instance** = outbound-only internet access for private subnets.
- **Security Group vs. NACL:** instance-level/stateful/allow-only vs. subnet-level/stateless/allow-and-deny. This is the single most testable comparison in this domain.
- **VPC Peering is non-transitive and requires non-overlapping CIDRs.**
- **Gateway Endpoints are only for S3 and DynamoDB; everything else uses Interface Endpoints.**
- **PrivateLink** = privately exposing a specific service to many external VPCs at scale (think SaaS/Marketplace vendors).
- **Site-to-Site VPN** (fast, public internet, encrypted, CGW+VGW) vs. **Direct Connect** (slow to provision, private physical line, expensive, reliable) — know the trade-offs.
- **Client VPN** connects an individual's computer to a VPC privately, distinct from Site-to-Site VPN which connects an entire data center.
- **Transit Gateway** = the answer whenever the exam scenario mentions connecting a **large number of VPCs** (and/or VPNs and Direct Connect) together via a hub-and-spoke model.
- **Elastic IPs cost money when idle/unattached** — a classic cost-optimization exam trap.
- **Public IPv4 addresses cost $0.005/hour** (750 free hours/month in Free Tier); **IPv6 addresses are free** and have no public/private distinction — all IPv6 addresses are public.
- **Shared Responsibility angle:** AWS is responsible for the security **of** the underlying network infrastructure (physical cabling, hypervisor-level isolation between VPCs); the customer is responsible for configuring their **own** VPC correctly — subnetting, route tables, Security Groups, NACLs, and choosing appropriate connectivity options (VPN vs. Direct Connect) are all customer-managed configuration decisions "in the cloud."