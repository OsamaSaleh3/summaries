
## Executive Overview

EC2 Instance Storage is one of the most heavily tested domains in the **AWS Certified Cloud Practitioner (CLF-C02)** exam, because storage decisions sit at the heart of almost every AWS architecture. Amazon EC2 (Elastic Compute Cloud) instances are virtual machines, and like any computer, they need a place to store their operating system, application files, logs, and data. AWS offers **several fundamentally different storage options**, each with different durability guarantees, performance characteristics, cost models, and availability scope.

Understanding these options — **EBS (Elastic Block Store)**, **Instance Store**, **EFS (Elastic File System)**, and **FSx** — along with supporting concepts like **AMIs (Amazon Machine Images)**, **EBS Snapshots**, and the **EC2 Image Builder**, is critical because:

- The exam frequently asks you to match a **use case** to the correct storage type (e.g., "which storage should be used for a shared file system accessed by hundreds of Linux servers?").
- Cost optimization, durability, and the **Shared Responsibility Model** are recurring themes across nearly every AWS service, and storage is a prime example of where AWS and the customer split responsibilities.
- Real-world architectures (web servers, databases, HPC clusters, disaster recovery strategies) are built by combining these storage types correctly.

This guide breaks down every storage type mentioned in the source material, deepens the technical explanations where needed, and highlights the exact details the CLF-C02 exam is known to test.

---

## 1. Amazon EBS (Elastic Block Store)

### 1.1 What is EBS?

**EBS (Elastic Block Store)** is a **network drive** that you can attach to a running EC2 instance. Think of it as a **virtual, network-attached USB stick**:

- It behaves like a hard drive plugged into your computer, but instead of a physical cable, the connection happens **over the network** inside AWS's data center.
- Because the connection is network-based rather than physical, there can be a small amount of **latency** between the EC2 instance and its EBS volume — this is a trade-off for the flexibility EBS provides.

### 1.2 Key Characteristics

- **Persistence**: EBS Volumes allow data to **persist even after the EC2 instance is terminated** (unless configured otherwise — see Delete on Termination below). This means you can terminate an instance, later create a new one, re-attach the same EBS volume, and get all your data back.
- **One-to-one attachment (CCP level)**: At the Cloud Practitioner exam level, an EBS Volume **can only be attached to one EC2 instance at a time**. (Note: in more advanced AWS knowledge, **Multi-Attach** exists for io1/io2 volumes, but this is beyond CCP scope and not something the exam will test.)
- **Multiple volumes per instance**: While one EBS Volume can only attach to one instance, a **single EC2 instance can have multiple EBS Volumes attached to it** — like having two USB sticks plugged into the same machine.
- **Availability Zone (AZ) locked**: An EBS Volume is **created in, and bound to, a specific Availability Zone** (e.g., `us-east-1a`). It **cannot** be attached to an instance in a different AZ (e.g., `us-east-1b`) directly. To move data across AZs, you must use an **EBS Snapshot** (explained in Section 2).
- **Detachable and re-attachable quickly**: Because it's a network drive (not physically wired), an EBS Volume can be **detached from one instance and reattached to another very quickly**. This is extremely useful for **failover scenarios** — if an instance fails, you can quickly attach its EBS volume to a replacement instance and resume operations with the data intact.
- **Provisioned capacity**: EBS is a **capacity-provisioned** service. When creating a volume, you must specify:
    - The **size** in GB.
    - The **IOPS** (Input/Output Operations Per Second) you want, which determines performance.
    - You are **billed for the provisioned capacity**, regardless of how much you actually use.
    - You **can increase the capacity over time** (size and/or performance) as your needs grow.
- **Standalone existence**: An EBS Volume does **not need to be attached to an instance** to exist. You can create volumes and leave them unattached, then attach them on demand later. This makes EBS a flexible, on-demand storage building block.

> **Analogy to remember:** EBS = Network USB stick. Bound to one AZ. Attach/detach at will. Persists independently of the instance's lifecycle.

### 1.3 Free Tier

AWS Free Tier includes **30 GB of free EBS storage per month**, which can be of type **General Purpose SSD (gp2/gp3)** or **Magnetic**. In most modern courses and real-world deployments, **GP2 and GP3** (General Purpose SSD volume types) are the most commonly used, cost-effective, general-purpose choices.

### 1.4 Delete on Termination Attribute

This is a specific configuration setting that **frequently appears on the exam**:

- When you launch an EC2 instance (or view its attached volumes in the console), there is a column/attribute called **"Delete on Termination."**
- **Default behavior**:
    - The **Root Volume** (the volume containing the OS) has **Delete on Termination = enabled (Yes)** by default. This means when you terminate the instance, its root EBS volume is **automatically deleted** along with it.
    - Any **additional (non-root) EBS Volume** attached to the instance has **Delete on Termination = disabled (No)** by default, meaning that volume **survives** instance termination.
- **You can control this setting** — enabling or disabling it for any volume, including the root volume.

**Use Case / Exam Scenario:** If you want to **preserve your root volume's data** even after the instance is terminated (e.g., to retrieve boot configuration or logs later), you would **disable Delete on Termination** for the root volume before terminating the instance.

### 1.5 EBS Volume Diagram Logic (Conceptual)

- In `us-east-1a`: EC2 Instance #1 ↔ attached to ↔ EBS Volume #1.
- A second EC2 Instance (#2) in the same AZ needs its **own separate EBS Volume** — it cannot share Volume #1 (at CCP level).
- An EC2 instance can have **two (or more) EBS volumes** attached simultaneously.
- To have EBS Volumes available in another AZ (e.g., `us-east-1b`), you must **create new volumes directly in that AZ** — you cannot simply reach across from `us-east-1a`.

---

## 2. EBS Snapshots

### 2.1 What is a Snapshot?

An **EBS Snapshot** is essentially a **backup** of an EBS Volume, taken at a specific point in time. Key facts:

- You can create a snapshot **at any time**.
- It is **not required** to detach the volume before snapshotting, but it is **recommended** to ensure data consistency (a "clean" snapshot), especially if the volume is actively being written to.
- Snapshots let you **restore** the state of a volume even after the original volume has been deleted or the instance has been terminated.

### 2.2 Why Use Snapshots?

1. **Backup and Restore** — protect against data loss.
2. **Move data across Availability Zones or Regions** — since EBS Volumes are locked to a single AZ, the only way to "move" a volume is to:
    1. Take a **snapshot** of the volume (snapshots exist at the **Region level**, not AZ level).
    2. **Restore** a new EBS Volume from that snapshot in a **different Availability Zone** (or even copy the snapshot to a different Region).
    3. Attach the new volume to an EC2 instance in the target AZ/Region.

> **Important distinction:** This process **creates a copy**, not a live/in-sync replica. The original and the new volume are two separate, independent volumes going forward.

### 2.3 EBS Snapshot Archive

- A **cost-saving storage tier** for snapshots you don't need to access quickly.
- Moving a snapshot to the **Archive Tier** gives you approximately **75% cost savings** compared to standard snapshot storage.
- **Trade-off**: Restoring a snapshot from the archive tier takes **between 24 to 72 hours**.
- **Best for**: Long-term, rarely-accessed backups where cost matters more than restore speed.

### 2.4 Recycle Bin for Snapshots (and AMIs)

- By default, **deleting a snapshot is permanent** — it cannot be recovered.
- AWS offers a **Recycle Bin** feature that protects against **accidental deletion**.
- When enabled via a **Retention Rule**, deleted snapshots (and AMIs) go into the Recycle Bin instead of being immediately destroyed.
- You define a **retention period** — configurable from **1 day up to 1 year** — during which the deleted resource can be **recovered**.
- After the retention period expires, the snapshot is permanently removed from the Recycle Bin.
- Retention Rules can be **locked or unlocked**:
    - **Unlocked** rules can be deleted/modified freely.
    - **Locked** rules provide stronger protection (cannot be easily removed), useful for compliance requirements.

### 2.5 Practical Cross-AZ Transfer Example

Scenario: You have an EBS Volume attached to an EC2 instance in `us-east-1a`, and you want the data available in `us-east-1b`.

1. Optionally stop the EC2 instance to ensure a clean, consistent snapshot (not mandatory).
2. Create an **EBS Snapshot** of the volume (this snapshot lives at the region level).
3. Use the snapshot to **create ("restore") a new EBS Volume** — this time specifying `us-east-1b` as the target Availability Zone.
4. **Attach** this new volume to an EC2 instance running in `us-east-1b`.

Result: You've effectively "moved" your data across AZs, though technically what happened is a **copy** was created via the snapshot mechanism.

---

## 3. Amazon EFS (Elastic File System)

### 3.1 What is EFS?

**EFS (Elastic File System)** is AWS's **managed Network File System (NFS)**. Unlike EBS (block storage, one instance at a time), EFS is designed for **sharing**:

- It can be **mounted onto hundreds of EC2 instances simultaneously**.
- This makes it a **shared file system** — multiple instances read and write to the same underlying storage.
- **EFS works only with Linux EC2 instances** (it is not compatible with Windows-based EC2 instances — that's what FSx for Windows File Server is for; see Section 5).
- EFS operates **across multiple Availability Zones within a region** — an instance in `us-east-1a` and another in `us-east-1b` can mount the **exact same EFS file system** at the same time and see the **same files**.

### 3.2 Characteristics

- **Highly available and scalable** by design (this is a managed AWS service — no capacity planning needed for durability/availability).
- **Pricier than EBS**: EFS costs roughly **3x the price of a GP2 EBS Volume** for comparable storage.
- **Pay-per-use pricing model**: Unlike EBS (where you provision and pay for capacity upfront), with EFS **you only pay for what you actually store**. If you store 20 GB, you're billed only for those 20 GB — no need to provision or guess capacity in advance.

### 3.3 EFS Architecture Diagram (Conceptual)

- One **EFS File System** exists, protected by a **Security Group**.
- Multiple EC2 instances across **`us-east-1a`, `us-east-1b`, and `us-east-1c`** all connect to the **same** EFS file system using a **mount target** in each AZ.
- All instances see the **same set of files** — this is what makes it "shared."

### 3.4 EBS vs. EFS — Core Differences (High Exam Relevance)

|Aspect|EBS|EFS|
|---|---|---|
|Storage type|Block storage|Network File System (NFS)|
|Attachment scope|One EC2 instance at a time (CCP level)|Hundreds of EC2 instances simultaneously|
|Availability Zone scope|Bound to a single AZ|Spans multiple AZs within a region|
|Cross-AZ data movement|Requires a Snapshot + Restore (creates a copy)|Natively shared — no copying needed|
|Pricing model|Pay for provisioned capacity|Pay-per-use (only what's stored)|
|OS compatibility|Linux and Windows|Linux only|
|Relative cost|Lower|~3x more expensive than GP2 EBS|

> **Exam Tip:** If a question describes a **shared file system accessed by many Linux instances across multiple AZs**, the answer is **EFS**. If it describes a **single instance with a dedicated, persistent disk**, the answer is **EBS**.

### 3.5 EFS Storage Classes and Cost Optimization

- **EFS Standard** — the default, higher-cost storage class for frequently accessed files.
- **EFS-IA (Infrequent Access)** — a cost-optimized storage class for files that are **not accessed often**.
    - Provides **up to 92% lower cost** than EFS Standard for storing infrequently accessed data.
    - Managed through a **Lifecycle Policy**: you configure a rule (e.g., "after 60 days without access, move the file to EFS-IA").
    - **Fully automatic and transparent**: EFS moves files between Standard and IA tiers behind the scenes based on last-access time.
    - If a file in EFS-IA is accessed again, it can automatically move **back** to EFS Standard, since it's now "frequently" used again.
    - **No application-level changes required** — applications interact with EFS the same way regardless of which storage class a file resides in. This transparency is a key benefit of EFS-IA.

> **Exam Tip:** EFS-IA is a **cost optimization feature**, not a durability or availability feature. Remember the phrase "**up to 92% lower cost**" — this specific number can appear in exam questions.

---

## 4. Shared Responsibility Model for EC2 Storage

This is a recurring theme across **all** AWS services on the CLF-C02 exam, and storage is no exception.

### 4.1 AWS's Responsibilities

- **Infrastructure management**: AWS manages and maintains the underlying physical hardware supporting EBS and EFS.
- **Data replication**: Per AWS's technical specifications, both EBS and EFS **replicate data across multiple pieces of underlying hardware**. AWS is responsible for performing this replication so that if one piece of hardware fails, **you as the customer are not impacted**.
- **Hardware failure & replacement**: If any part of an EBS drive (or the underlying hardware) fails, it is **AWS's responsibility** to detect and **replace the faulty hardware**.
- **Employee access controls**: AWS is responsible for ensuring that **its own employees cannot access your data** stored on these services.

### 4.2 Customer's Responsibilities

- **Backup and snapshot procedures**: Setting up your own backup strategy (e.g., regular EBS Snapshots) is **your responsibility** — AWS does not automatically back up your data for you.
- **Data encryption**: Configuring encryption on your volumes/file systems is an **additional layer of protection** that is your responsibility to enable. This protects your data from unauthorized access — whether from AWS itself or from other AWS customers — even though AWS already has strong security controls in place.
- **Data you place on the disk**: Anything you write to your EBS Volume or EFS file system is entirely **your own responsibility** in terms of correctness, sensitivity handling, and lifecycle management.
- **Instance Store risk awareness**: If you choose to use an **EC2 Instance Store** (see Section 6), it is your responsibility to understand that:
    - Data can be lost due to **underlying hardware failure**.
    - Data is **lost entirely** if the instance is **stopped or terminated**.
    - You must implement your own **backup/replication strategy** if you rely on Instance Store for any important data.

> **Exam Tip:** Whenever a question is framed around "who is responsible for X" regarding storage, remember: **AWS handles the underlying hardware, replication, and physical security; you handle your data, backups, encryption configuration, and any risk associated with ephemeral storage choices.**

---

## 5. Amazon FSx

### 5.1 What is FSx?

**Amazon FSx** is a **managed service that lets you run third-party, high-performance file systems on AWS**. If EFS or Amazon S3 don't fit your specific technical needs (e.g., you need Windows-native file sharing or specialized high-performance computing storage), **FSx** provides purpose-built alternatives.

AWS offers multiple FSx flavors (and may add more over time), but the CCP exam focuses on **two primary offerings**:

1. **FSx for Windows File Server**
2. **FSx for Lustre**

_(A third offering, FSx for NetApp ONTAP, exists but is not a core focus for the CCP exam based on the source material.)_

### 5.2 FSx for Windows File Server

- A **fully managed, highly reliable, and scalable** Windows-native shared file system, built on **Windows File Server** technology.
- Designed specifically for **Windows-based workloads**.
- Typically **deployed across two Availability Zones** for high availability.
- Supports native **Windows protocols**:
    - **SMB (Server Message Block)** protocol.
    - **Windows NTFS** file system semantics.
- This allows Windows clients — whether in your **on-premises corporate data center** or **Windows-based EC2 instances** — to mount and access the file system using familiar, native Windows tooling.
- **Integrates with Microsoft Active Directory** for user authentication and security, since FSx for Windows is inherently a **Microsoft-oriented offering**.
- Can be accessed both **from within AWS** and **from your on-premises infrastructure**.

> **Exam Tip:** Whenever you see "**Windows file share**," "**SMB**," or "**Active Directory integration**" combined with file storage, think **FSx for Windows File Server**.

### 5.3 FSx for Lustre

- A **fully managed, high-performance, and scalable file storage system** designed specifically for **High Performance Computing (HPC)** workloads.
- **Name origin**: "Lustre" is derived from combining **"Linux"** and **"cluster."** (A helpful mnemonic: think of clustered, high-performance Linux compute jobs.)
- **Use cases**: machine learning, big data analytics, video processing, financial modeling, and other compute-intensive workloads.
- **Performance characteristics**:
    - Scales to **hundreds of gigabytes per second** of throughput.
    - Supports **millions of IOPS (I/O Operations Per Second)**.
    - **Sub-millisecond latencies**.
- **Architecture**: FSx for Lustre can connect to your **on-premises data center** or directly to **compute instances within AWS**. Behind the scenes, FSx for Lustre often stores its data on an **Amazon S3 bucket**, integrating tightly with the broader AWS storage ecosystem.

> **Exam Tip:** Whenever you see "**High Performance Computing (HPC)**" combined with a Linux-based file system, think **FSx for Lustre**. Whenever you see "**Windows**" combined with a shared file system, think **FSx for Windows File Server**.

### 5.4 FSx Summary Table

|Feature|FSx for Windows File Server|FSx for Lustre|
|---|---|---|
|Target OS|Windows|Linux / HPC workloads|
|Protocol|SMB, NTFS|High-throughput parallel file system|
|Directory integration|Microsoft Active Directory|N/A|
|Primary use case|Shared corporate Windows file shares|Machine learning, analytics, video processing, financial modeling|
|Backing storage|Managed by FSx|Can integrate with Amazon S3|
|Deployment|Typically across 2 AZs|Connects to on-prem or AWS compute|

---

## 6. EC2 Instance Store

### 6.1 What is Instance Store?

**EC2 Instance Store** is **physical, hardware-attached disk storage** directly connected to the underlying physical server hosting your EC2 instance — as opposed to EBS/EFS, which are **network-attached** storage.

- An EC2 instance is a **virtual machine**, but it runs on a **real, physical hardware server**. Some of these physical servers have **local disks physically wired** to them.
- **Only specific EC2 instance types/families** can leverage Instance Store (it is not available on every instance type — it depends on the underlying hardware configuration of that instance family).

### 6.2 Why Use Instance Store?

- **Extremely high I/O performance**: Because the disk is physically attached (no network hop), Instance Store delivers **much higher IOPS and throughput** than network-attached EBS volumes.
- **Great throughput**: Ideal when raw disk speed matters more than durability.

### 6.3 The Critical Trade-off: Ephemeral Storage

- Instance Store is called **"ephemeral storage"** because:
    - If you **stop** or **terminate** the EC2 instance, **all data on the Instance Store is permanently lost**.
    - It **cannot** be used as durable, long-term storage.
- **Hardware failure risk**: If the underlying physical server fails, the Instance Store data is lost as well — there is **no automatic replication** like there is with EBS/EFS.
- **Full customer responsibility**: If you choose to use Instance Store, **you** are entirely responsible for backing it up and replicating it according to your own needs. AWS does not guarantee durability for this storage type.

### 6.4 Best Use Cases for Instance Store

- **Buffers and caches**.
- **Scratch data** or **temporary content** that doesn't need to survive instance stop/termination.
- **NOT appropriate** for long-term, durable data storage — for that, **use EBS instead**.

### 6.5 Performance Comparison (Illustrative Example)

The source material gives a concrete performance comparison to illustrate _why_ Instance Store exists as an option (exact numbers are illustrative, not something you need to memorize for the exam):

- On instance families like **I3.*** (instance types designed around local NVMe storage), Instance Store can reach:
    - **~3.3 million Read IOPS**
    - **~1.4 million Write IOPS**
- Compare this to a **GP2 EBS Volume**, which tops out around **~32,000 IOPS**.

This illustrates that Instance Store can be **orders of magnitude faster** than network-attached EBS, at the cost of durability.

> **Exam Tip:** Whenever an exam question describes a scenario requiring **extremely high performance, local, hardware-attached storage** — and especially if it mentions that **data loss upon stop/termination is acceptable** — think **EC2 Instance Store**.

### 6.6 Instance Store vs. EBS — Quick Comparison

|Aspect|EBS|Instance Store|
|---|---|---|
|Connection type|Network-attached|Physically attached to host hardware|
|Persistence|Persists independently of instance lifecycle|Lost on stop/terminate|
|Performance|Very good, but network-bound|Extremely high (no network latency)|
|Durability|AWS replicates underlying hardware|No replication — your responsibility|
|Best use case|Durable, long-term data|Caches, buffers, scratch/temporary data|
|Availability scope|Locked to a specific AZ|Tied to the specific physical host|

---

## 7. Amazon Machine Images (AMIs)

### 7.1 What is an AMI?

An **AMI (Amazon Machine Image)** is essentially a **template/snapshot of an EC2 instance's configuration** — it represents a **customization of an EC2 instance** that can be used to launch new instances quickly.

An AMI can include:

- The **operating system** configuration.
- **Software** already installed and configured (e.g., web servers, monitoring agents, security tools, applications).
- **Monitoring tools** and other pre-installed utilities.

### 7.2 Why Use a Custom AMI?

- **Faster boot and configuration time**: Since all desired software is **prepackaged** into the AMI, you skip the time-consuming steps of manually installing software every time you launch a new instance.
- **Consistency**: Every instance launched from the same AMI starts in an identical, known-good state.
- **Region flexibility**: AMIs are built for a **specific region** but **can be copied across regions**, letting you leverage AWS's global infrastructure to launch identical instances anywhere.

### 7.3 Types of AMIs You Can Use

1. **Public AMIs** — provided directly by AWS (e.g., the popular **Amazon Linux 2 AMI**). These are ready-to-use, maintained by AWS.
2. **Your Own AMIs** — created and maintained by you. You are responsible for building, updating, and maintaining these over time (though automation tools exist — see EC2 Image Builder below).
3. **AWS Marketplace AMIs** — AMIs created (and often sold) by **third-party vendors**. These come pre-configured with specific software, saving you setup time. Businesses can even build a revenue model by **selling their own AMIs** on the AWS Marketplace.

### 7.4 The AMI Creation Process

1. **Launch** an EC2 instance.
2. **Customize** it (install software, configure settings, etc.).
3. **Stop** the instance — this ensures **data integrity** before the image is captured.
4. **Create an AMI** from the stopped instance. Behind the scenes, this process also **creates EBS Snapshots** of the instance's attached volumes.
5. **Launch new instances** from this custom AMI whenever needed — each new instance will be an identical copy of the original, customized configuration.

> **Practical scenario:** Launch and customize an instance in `us-east-1a`, create an AMI from it, then use that AMI to launch an identical instance in `us-east-1b` — effectively replicating your custom server setup into another Availability Zone or Region.

### 7.5 Practical Demo Takeaways (from Hands-on Walkthrough)

- When creating an AMI-based instance with a **user data script**, you can skip redundant steps (like reinstalling a web server such as **HTTPD/Apache**) since the AMI **already contains the software** — resulting in **much faster boot-up time** compared to installing everything from scratch via user data scripts.
- This illustrates the core value proposition of AMIs: **pre-bake your configuration once, then launch quickly, repeatedly, anywhere.**

---

## 8. EC2 Image Builder

### 8.1 What is EC2 Image Builder?

**EC2 Image Builder** is a service that **automates the creation, maintenance, validation, and testing of AMIs** (Amazon Machine Images) — and it can also be used for container images.

### 8.2 How It Works (Step-by-Step Flow)

1. **Builder Stage**: EC2 Image Builder automatically **launches a "Builder" EC2 instance**.
2. **Customization**: On this Builder instance, it installs and configures whatever components you've defined — e.g., installing Java, updating the CLI, updating system software/packages, installing firewalls, installing your application, etc.
3. **AMI Creation**: Once customization is complete, an **AMI is automatically created** from that Builder instance.
4. **Validation/Testing Stage**: EC2 Image Builder automatically creates a **Test EC2 instance** from the newly created AMI, then runs a **battery of tests** that you define in advance (e.g., "Is the AMI working correctly?", "Is it secure?", "Is my application running as expected?").
    - You can also **choose to skip testing** if you don't need it.
5. **Distribution Stage**: Once validated, the AMI is **distributed** — and importantly, it can be distributed to **multiple AWS Regions**, enabling truly **global** application deployment workflows.

### 8.3 Additional Features

- **Scheduling**: EC2 Image Builder can run:
    - On a **defined schedule** (e.g., weekly).
    - **Whenever packages are updated**.
    - **Manually**, on demand.
- **Pricing**: EC2 Image Builder itself is a **free service** — you only pay for the **underlying resources it consumes**, such as:
    - The EC2 instances it creates during the build/test process.
    - The **storage cost** of the resulting AMI(s), in every region where it's created and/or distributed.

> **Exam Tip:** EC2 Image Builder = **automation** for building, testing, and distributing AMIs. Remember it is a **free service** in terms of the orchestration itself — you only pay for the compute and storage resources it uses behind the scenes.

---

## 9. Practical/Console Concepts Worth Remembering

These details come from the hands-on demo portions of the course and reinforce theoretical concepts with real console behavior:

- **Storage tab on an EC2 instance** shows the **Root Device** and any additional **Block Devices** (EBS Volumes) attached, including their **Delete on Termination** setting (visible as a column when scrolled to the right).
- **Creating a new EBS Volume** requires selecting:
    - Volume type (e.g., **GP2, GP3**).
    - Size (in GB).
    - **Availability Zone** — and it **must match** the AZ of the EC2 instance you intend to attach it to, or the attachment will fail.
- Attempting to **attach a volume to an instance in a different AZ** will **fail** — this is a direct, hands-on demonstration of the AZ-locking rule for EBS.
- **Deleting volumes/snapshots is nearly instantaneous** — showcasing the elasticity and on-demand nature of cloud infrastructure ("request volumes, delete volumes in a matter of seconds").
- When an **EC2 instance is terminated**, any attached EBS Volume with **Delete on Termination = Yes** (typically the root volume by default) will be **automatically detached and then deleted**. Volumes with this setting set to **No** will simply become "available" (detached) rather than deleted.
- **Snapshots can be copied to other regions** directly from the console (right-click → Copy Snapshot) — a common technique for building a **Disaster Recovery (DR) strategy**, ensuring your data is backed up in a geographically separate AWS region.
- **Recreating a Volume from a Snapshot** allows you to choose a **different target Availability Zone** than the original volume — this is the mechanical basis for "moving" an EBS volume across AZs.
- The **Recycle Bin** (via a Retention Rule) protects both **EBS Snapshots** and **AMIs** from accidental deletion, and recovered resources return to their normal, usable state once restored from the bin.
- **AMI deregistration**: When you no longer need a custom AMI, you **deregister** it (rather than simply "delete" it) via the AMI console.
- Cleaning up demo resources typically involves, in order: **terminate EC2 instances → delete EBS volumes → delete snapshots → deregister AMIs** (snapshots tied to an active AMI cannot be deleted until the AMI is deregistered first).

---

## 10. Summary Table — All EC2 Storage Options at a Glance

|Storage Type|Scope|Attach Target|Durability|Best For|Cost Profile|
|---|---|---|---|---|---|
|**EBS**|Single Availability Zone|One EC2 instance at a time|Persists independent of instance lifecycle (unless Delete on Termination = Yes)|General-purpose durable block storage, boot volumes, databases|Pay for provisioned capacity|
|**EBS Snapshot**|Region (copyable to other regions)|N/A (backup artifact)|Durable backup|Backups, cross-AZ/region data movement, DR|Pay per GB stored; Archive tier ~75% cheaper|
|**EFS**|Multi-AZ (region-wide)|Hundreds of Linux EC2 instances simultaneously|Highly available, managed|Shared file storage across many Linux instances|Pay-per-use; ~3x GP2 EBS cost; EFS-IA up to 92% cheaper for cold data|
|**FSx for Windows**|Multi-AZ|Windows EC2 instances + on-prem|Managed, highly reliable|Windows-native shared file systems, AD integration|Managed service pricing|
|**FSx for Lustre**|Region/on-prem connect|HPC compute instances|Managed, high-performance|Machine learning, analytics, video processing, financial modeling|Managed service pricing|
|**EC2 Instance Store**|Tied to physical host|The single instance running on that host|**Ephemeral** — lost on stop/terminate or hardware failure|Caches, buffers, scratch/temporary data|Often included in instance price; no separate provisioning|
|**AMI**|Region (copyable across regions)|Used to launch new instances|Persistent template|Fast, consistent instance provisioning|Pay for underlying EBS snapshot storage|

---

## 11. Consolidated Exam Tips

- **EBS = network drive, one instance at a time, bound to one AZ.** Use **Snapshots** to move data across AZs/regions.
- **Delete on Termination**: **Root volume = Yes by default; additional volumes = No by default.** Know how to change this to preserve or discard data intentionally.
- **EFS = shared NFS, Linux only, multi-AZ, pay-per-use, ~3x cost of GP2 EBS.** Use **EFS-IA** for up to **92% cost savings** on infrequently accessed files via lifecycle policies.
- **Shared Responsibility Model**: AWS = hardware, replication, physical security, hardware failure replacement. Customer = backups, encryption setup, data content, and understanding ephemeral storage risk.
- **FSx for Windows** = SMB/NTFS + Active Directory integration, for Windows workloads. **FSx for Lustre** = HPC workloads on Linux, extremely high throughput/IOPS/low latency.
- **EC2 Instance Store** = physically attached, extremely high performance, but **ephemeral** — data lost on stop/terminate or hardware failure. Great for caches/buffers/scratch data, never for durable storage.
- **AMI** = reusable, customizable instance template; speeds up launch time; can be copied across regions; building one creates EBS Snapshots behind the scenes.
- **EC2 Image Builder** = free automation service (you only pay for underlying compute/storage) for building, testing, and distributing AMIs across regions on a schedule or on demand.
- **EBS Snapshot Archive** = ~75% cheaper storage tier, but restore takes 24–72 hours — good for rarely-needed, non-urgent backups.
- **Recycle Bin** = protects EBS Snapshots and AMIs from accidental deletion, with configurable retention from 1 day to 1 year.