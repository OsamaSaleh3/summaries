
## Executive Overview

Amazon S3 (Simple Storage Service) is one of the foundational, "must-know" services in all of AWS. It is marketed as **infinitely scaling object storage**, and it quietly powers a huge portion of the internet — countless websites use it as a backbone for hosting content, and a large number of other AWS services (CloudFront, Lambda, Athena, EMR, Elastic Beanstalk, CloudTrail, and many more) integrate with S3 for storing and retrieving data.

Because S3 is so widely used and so heavily tested, the AWS Certified Cloud Practitioner (CLF-C02) exam expects you to understand:

- The core object storage model (buckets, objects, keys)
- The different **storage classes** and when to use each
- **Security** mechanisms (IAM policies, bucket policies, ACLs, encryption, Block Public Access)
- **Versioning** and **Replication**
- **Static website hosting**
- Related services that extend S3: the **AWS Snow Family** and **AWS Storage Gateway**
- The **Shared Responsibility Model** as it applies to S3

At its core, S3 is simply **object storage**: you store "objects" (files) inside "buckets" (containers), and AWS handles the scaling, durability, and availability for you.

### Common Use Cases for Amazon S3

- **Backup and storage** — for files, disks, snapshots, etc.
- **Disaster recovery (DR)** — replicate data to another AWS Region so that if one region becomes unavailable, your data is safe elsewhere.
- **Archiving** — store data cheaply for long-term retention and retrieve it later (via the S3 Glacier storage classes) at a much lower cost than "hot" storage.
- **Hybrid cloud storage** — extend on-premises storage into the cloud (via AWS Storage Gateway).
- **Application hosting** — store and serve static assets used by applications.
- **Media hosting** — host video files, images, and other rich media.
- **Data lakes** — store massive volumes of raw/structured/unstructured data and run big data analytics on top of it (using services like Amazon Athena, EMR, Redshift Spectrum).
- **Software delivery** — distribute software updates and installers.
- **Static website hosting** — host an entire website (HTML/CSS/JS/images) directly from a bucket.

### Real-World Examples Mentioned

- **Nasdaq** stores seven years of data in the **S3 Glacier** service (the archival tier of S3) for compliance and historical recordkeeping.
- **Sysco** runs analytics on data stored in Amazon S3 to generate business insights.

---

## 1. Buckets and Objects — The Core Building Blocks

### 1.1 Buckets

- A **bucket** is essentially a **top-level container/directory** where you store your files.
- Buckets are created **within your AWS account**.
- **Bucket names must be globally unique** — unique not just within your account or your region, but across **every AWS account on the planet**, in every region.
    
    > **Exam Tip:** The S3 bucket **name** is the _only_ resource identifier in AWS that must be globally unique across all accounts and regions.
    
- Even though bucket names are globally unique, **buckets are created and physically reside in a single, specific AWS Region** that you choose at creation time.
    
    > **Common Beginner Mistake:** S3 looks like a "global" service because you can see all your buckets across all regions in one console view, and because names are globally unique — but the underlying **data for each bucket lives in one specific region** you selected when you created it. Latency, compliance, and data residency depend on this region choice.
    

#### S3 Bucket Naming Rules (Naming Convention)

You are not expected to memorize every rule, but you should recognize valid vs. invalid names:

- No uppercase letters.
- No underscores.
- Must be between **3 and 63 characters** long.
- Must **not** be formatted like an IP address (e.g., `192.168.1.1`).
- Must start with a lowercase letter or a number.
- Can only contain lowercase letters, numbers, and hyphens (with a few additional prefix restrictions for special bucket types).

#### Bucket Types

Depending on your region, when creating a bucket you may see a choice between:

- **General Purpose** buckets — the standard, default bucket type. This is what you should choose (or what is used automatically if you don't see the option).
- **Directory buckets** — a specialized bucket type built for very low-latency, high-performance workloads (used with S3 Express One Zone). This is **not covered** in the Cloud Practitioner exam, so you can safely ignore it for CLF-C02 purposes.

#### Object Ownership & Public Access at Bucket Creation

When creating a bucket, AWS presents several security-related defaults:

- **ACLs disabled** (recommended) — object ownership defaults to the bucket owner, simplifying permissions.
- **Block all public access — enabled by default** — this is a strong, safe default that prevents accidental public exposure of your data.
- **Bucket Versioning — disabled by default** (must be explicitly turned on).
- **Default encryption** — Server-side encryption with an Amazon S3-managed key (**SSE-S3**) is applied by default, meaning **every object you upload is encrypted automatically**, even without extra configuration.

### 1.2 Objects

- Objects are the actual **files** stored inside a bucket.
- Every object has a **key**, which represents the **full path** of the file within the bucket.
    - Example: if a file sits at the root of the bucket, its key is simply `myfile.txt`.
    - If nested inside "folders," the key becomes the full path: `myfolder1/myfolder2/myfile.txt`.
- The key is composed of two conceptual parts:
    - **Prefix** — everything before the final slash (e.g., `myfolder1/myfolder2/`)
    - **Object name** — the final segment (e.g., `myfile.txt`)

> **Important Conceptual Point:** Amazon S3 **does not have a true concept of directories/folders**. Everything is a **flat namespace of objects**, each identified by a unique _key_. The AWS Console _simulates_ a folder structure using prefixes and slashes in the key name to make navigation feel familiar, but under the hood there is no real nested file system — just very long key names.

#### Object Composition

- **Value** — the actual content/data (the "body") of the file.
- **Key** — the unique identifier/full path of the object within the bucket.
- **Version ID** — assigned if bucket versioning is enabled (see the Versioning section below).
- **Metadata** — a list of key-value pairs describing the object; can be system-defined or user-defined.
- **Tags** — Unicode key-value pairs, **up to 10 per object**, useful for **security** (fine-grained IAM permission conditions) and **lifecycle management** (e.g., transition rules based on tags).

#### Object Size Limits

- **Maximum object size: 5 TB** (5,000 GB) — this is the largest a single object can be.
- **Maximum single PUT upload size: 5 GB.** If an object is **larger than 5 GB**, you are **required** to use **Multi-Part Upload**.

#### Multi-Part Upload

- Multi-Part Upload breaks a large file into smaller parts (chunks) and uploads them in parallel, then reassembles them on S3's side.
- **Recommended** for files > 100 MB, and **mandatory** for files larger than 5 GB.
- Example: a 5 TB file must be split into **at least 1,000 parts** (since each part can be up to 5 GB).
- **Benefits:** improved upload speed via parallelization, resilience (only failed parts need to be retried, not the entire file), and it's the only way to upload objects that exceed the single-PUT 5 GB limit.

---

## 2. Amazon S3 Storage Classes

S3 offers multiple **storage classes** (tiers), each with different trade-offs between **cost, availability, durability, and retrieval speed**. You choose the storage class per object — either at upload time, by manually editing it later, or automatically via **Lifecycle Rules**.

### 2.1 Durability vs. Availability — Key Definitions

- **Durability**: the likelihood that S3 will **not lose** your object over time. All S3 storage classes offer the same extremely high durability: **99.999999999%**, commonly called **"11 nines."**
    - In practical terms: if you store **10 million objects** on S3, you can statistically expect to lose **a single object roughly once every 10,000 years**.
    - Durability is **identical across all storage classes** — a Glacier Deep Archive object is exactly as durable as a Standard object.
- **Availability**: how readily/consistently the service is **available for you to use** at any given moment. Availability **varies by storage class**.
    - Example: S3 Standard offers **99.99% availability**, meaning the service could be unavailable roughly **53 minutes per year**. Applications should be designed to tolerate occasional errors/retries.

### 2.2 The Storage Classes in Detail

|Storage Class|Availability|Key Characteristics|Typical Use Case|
|---|---|---|---|
|**S3 Standard – General Purpose**|99.99%|Low latency, high throughput; can sustain the loss of **two concurrent Availability Zone (AZ) facilities**|Frequently accessed data: big data analytics, mobile/gaming apps, content distribution|
|**S3 Standard-Infrequent Access (Standard-IA)**|99.9%|Lower storage cost than Standard, but you pay a **retrieval fee**; rapid access when needed|Disaster recovery, backups|
|**S3 One Zone-Infrequent Access (One Zone-IA)**|99.5%|Stored in only **a single Availability Zone** — data is **lost if that AZ is destroyed**; lower cost than Standard-IA|Secondary backup copies of on-premises data, or data that can be re-created|
|**S3 Glacier Instant Retrieval**|High (archive tier)|**Millisecond** retrieval; minimum storage duration **90 days**|Archival data accessed roughly **once a quarter** but needing instant access|
|**S3 Glacier Flexible Retrieval** (formerly "Amazon S3 Glacier")|High (archive tier)|Three retrieval speed options; minimum storage duration **90 days**|Backups/archives where some retrieval delay is acceptable|
|**S3 Glacier Deep Archive**|High (archive tier)|Cheapest storage class; minimum storage duration **180 days**|Long-term archival (e.g., compliance retention for 7+ years)|
|**S3 Intelligent-Tiering**|99.9%|Automatically moves objects between tiers based on access patterns; small monitoring/auto-tiering fee; **no retrieval charges**|Unpredictable or unknown access patterns|

#### Glacier Retrieval Speed Options (details)

- **S3 Glacier Instant Retrieval**
    
    - Retrieval time: **milliseconds**
    - Minimum storage duration: **90 days**
    - Best for archive data that must still be available almost instantly (e.g., accessed once per quarter).
- **S3 Glacier Flexible Retrieval** (three speed tiers):
    
    - **Expedited**: 1–5 minutes
    - **Standard**: 3–5 hours
    - **Bulk**: 5–12 hours (this tier is **free**)
    - Minimum storage duration: **90 days**
- **S3 Glacier Deep Archive** (two speed tiers):
    
    - **Standard**: 12 hours
    - **Bulk**: 48 hours
    - Minimum storage duration: **180 days**
    - This is the **lowest-cost** S3 storage class, intended for data you almost never need to touch (e.g., regulatory/compliance archives).

> **Mnemonic:** "Instant" = you need the data back **instantly**. "Flexible" = you're **flexible/willing to wait**, anywhere from minutes to half a day. "Deep Archive" = you're willing to wait even longer (up to 48 hours) for the **cheapest** possible storage.

#### S3 Intelligent-Tiering Sub-Tiers

Intelligent-Tiering automatically moves objects between access tiers based on **actual usage patterns**, without performance impact and without retrieval charges. It has these sub-tiers:

1. **Frequent Access tier** — default tier, automatic.
2. **Infrequent Access tier** — automatic, for objects not accessed for **30 days**.
3. **Archive Instant Access tier** — automatic, for objects not accessed for **90 days**.
4. **Archive Access tier** — _optional_, configurable for objects not accessed between **90 and 700+ days**.
5. **Deep Archive Access tier** — _optional_, configurable for objects not accessed between **180 and 700+ days**.

> **Why use Intelligent-Tiering?** It lets you "set it and forget it" — ideal when you don't know or can't predict your data's access patterns, since AWS automatically optimizes cost for you without manual lifecycle management.

#### Deprecated Storage Class

- **Reduced Redundancy Storage (RRS)** is a **legacy/deprecated** storage class. It is no longer recommended and is not covered in depth for the exam — if you see it in the console, know that it should not be used for new workloads.

### 2.3 Storage Class Management

- You can **set** the storage class when uploading an object.
- You can **manually change** an object's storage class after the fact (via the console, CLI, or API).
- You can **automate** storage class transitions using **S3 Lifecycle Rules** (also called Lifecycle Configurations).

#### S3 Lifecycle Rules

- Defined at the bucket level (Management tab in the console).
- Allow you to define **transition actions**: automatically move objects between storage classes after a specified number of days.
    - Example policy: move to **Standard-IA after 30 days** → move to **Intelligent-Tiering after 60 days** → move to **Glacier Flexible Retrieval after 180 days**.
- Can apply to **all objects** in a bucket or be scoped with filters (e.g., prefix or tag-based).
- Lifecycle rules can also define **expiration actions** (permanently deleting objects after a certain time), and can manage the deletion/transition of **non-current (older) versions** when versioning is enabled.

> **Exam Tip:** You are **not** expected to memorize exact pricing figures or the precise number of days for each transition threshold — focus on **understanding the relative cost/availability/retrieval-speed trade-offs** between classes, and knowing that Lifecycle Rules automate movement between them.

---

## 3. Amazon S3 Security

Security in S3 revolves around **four layers**: user-based permissions, resource-based permissions, access control lists, and encryption.

### 3.1 User-Based Security (IAM Policies)

- Standard **IAM policies** attached to IAM users, groups, or roles specify **which S3 API calls** (e.g., `s3:GetObject`, `s3:PutObject`) that principal is permitted to perform.
- This is how you'd grant an **EC2 instance** access to S3: since IAM _users_ are not appropriate for applications running on EC2, you attach an **IAM Role** to the EC2 instance with the necessary S3 permissions.

### 3.2 Resource-Based Security (S3 Bucket Policies)

- **S3 Bucket Policies** are **JSON-based**, resource-attached policies configured directly from the S3 console (or via CLI/API/IaC).
- They define **bucket-wide rules**: who can do what, on which objects, within a specific bucket.
- Common use cases:
    - **Granting public read access** to a bucket (e.g., for a static website).
    - **Cross-account access** — allowing a user or role from a _different_ AWS account to access your bucket.
    - **Forcing encryption** — denying uploads (`PutObject`) that are not encrypted.

#### Anatomy of a Bucket Policy

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "PublicRead",
      "Effect": "Allow",
      "Principal": "*",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::example-bucket/*"
    }
  ]
}
```

- **Resource** — which bucket/objects the policy applies to (e.g., `arn:aws:s3:::example-bucket/*` targets **every object** inside `example-bucket`, thanks to the trailing `/*`).
- **Effect** — `Allow` or `Deny`.
- **Action** — the specific API call(s) being allowed or denied (e.g., `s3:GetObject` = read/download an object).
- **Principal** — _who_ the policy applies to. A value of `"*"` means **anyone in the world** (public access); it could instead be a specific AWS account ID or IAM ARN for restricted/cross-account access.

> This example bucket policy grants **public read access** to every object in the bucket — this is exactly the type of policy used to enable static website hosting or public file distribution.

### 3.3 Access Control Lists (ACLs)

- **Object ACLs** — finer-grained, per-object access rules. **Can be disabled** (and disabling them, i.e., relying on bucket policies instead, is now the AWS-recommended best practice).
- **Bucket ACLs** — bucket-wide access rules, **less commonly used** today, and can also be disabled.
- **Modern best practice:** Use **Bucket Policies** rather than ACLs for the vast majority of access control needs. ACLs are considered a legacy mechanism.

### 3.4 When Can an IAM Principal Access an S3 Object?

An IAM principal (user, role, etc.) can perform an action on an S3 object **if and only if**:

> Either the **IAM permissions** (user-based policy) explicitly allow it, **OR** the **resource policy** (bucket policy) explicitly allows it — **AND** there is **no explicit `Deny`** anywhere in the evaluation (IAM policy, bucket policy, or Service Control Policy).

This mirrors the general AWS IAM evaluation logic: **explicit deny always wins**, regardless of any allow statements elsewhere.

### 3.5 Block Public Access Settings

- A dedicated set of **bucket-level (or account-level) settings** that AWS introduced as an **extra safety net** to prevent accidental data leaks.
- **Key behavior:** Even if a bucket policy is configured to grant public access, if **Block Public Access** is enabled, the **bucket will never actually become public** — Block Public Access **overrides** the bucket policy.
- Can be configured:
    - **Per bucket** — useful when some buckets legitimately need to be public (e.g., static websites) and others don't.
    - **Account-wide** — recommended if **none** of your buckets should ever be public, as a blanket safeguard against misconfiguration.

> **Exam Tip:** This is a frequently tested concept — remember that Block Public Access is a **safety override** that takes precedence over bucket policies, protecting against human error when writing overly permissive JSON policies.

### 3.6 Presigned URLs

- The **object URL** (public link) of an object will **not** work by default if the bucket/object is private — attempting to open it returns an **Access Denied** error.
- A **presigned URL**, by contrast, **does** work for a private object because it **embeds a temporary signature** derived from the credentials of the user who generated it.
- AWS validates the embedded signature and grants time-limited access to whoever holds that specific URL — without needing to make the object or bucket public.
- Common use case: temporarily sharing a private file (e.g., allowing someone to download a document or upload a file directly to S3) without exposing the bucket to the public.

### 3.7 IAM Access Analyzer for S3

- A **monitoring/auditing feature** that helps ensure only the **intended parties** have access to your S3 buckets.
- It works by analyzing:
    - **Bucket Policies**
    - **S3 ACLs**
    - **S3 Access Point Policies**
- It **surfaces findings** such as:
    - Which buckets are **publicly accessible**.
    - Which buckets are **shared with other AWS accounts**.
- You review these findings and decide whether the exposure is **intentional** or represents a **security misconfiguration**, then take corrective action.
- It is powered by the broader **IAM Access Analyzer** service, which identifies resources across your account that are shared with entities outside your "zone of trust."

### 3.8 Encryption

Two encryption models exist for S3 objects, and understanding the distinction is important for the exam:

- **Server-Side Encryption (SSE)** — **the default and most common approach**.
    - The object is uploaded to S3 in its unencrypted form, and **Amazon S3 itself performs the encryption** once the data arrives at the bucket.
    - S3 buckets/objects are **encrypted by default** with SSE using an **Amazon S3-managed key (SSE-S3)** — no configuration required.
    - (Beyond SSE-S3, AWS also offers **SSE-KMS**, using AWS Key Management Service keys for additional auditability and control, and **SSE-C**, where the customer supplies their own encryption key — these variants go deeper than what's needed at the Cloud Practitioner level, but it's good to know they exist.)
- **Client-Side Encryption**
    - The **user encrypts the file themselves** _before_ uploading it to S3.
    - S3 simply stores the already-encrypted blob; it has no visibility into the unencrypted content.
    - The user is responsible for managing encryption/decryption keys entirely outside of AWS's involvement.

> **Exam Tip:** Remember the key fact — **server-side encryption is always on by default** for new S3 buckets/objects. You don't have to manually turn on basic encryption; you may only need to configure _which type_ (SSE-S3, SSE-KMS, etc.) if you want a stronger key management model.

---

## 4. S3 Versioning

**Versioning** is a bucket-level setting (disabled by default) that, once enabled, causes S3 to keep **every version** of an object whenever it's overwritten, rather than replacing it.

### 4.1 How It Works

- Once enabled at the bucket level, uploading a file to an existing key creates **Version 1**.
- Re-uploading (overwriting) that same key creates **Version 2**, then **Version 3**, and so on — **nothing is actually overwritten**; old versions are preserved.
- Any files that existed in the bucket **before** versioning was enabled will show a version ID of **`null`**.

### 4.2 Why Use Versioning (Best Practice)

- **Protects against unintended deletes**: deleting an object (without specifying a version) doesn't actually erase the data — it simply adds a **delete marker** on top of the version history. The underlying versions remain recoverable.
- **Easy rollback**: you can restore or revert to any previous version of a file at any time (e.g., "go back to what this file looked like two days ago").

### 4.3 Delete Behavior — Two Different Operations

|Operation|What Happens|Reversible?|
|---|---|---|
|**Delete (no version specified)**|Adds a **delete marker** on top of the object; the previous versions remain in the bucket, just hidden from normal view|**Yes** — deleting the delete marker restores the object|
|**Delete a specific Version ID** ("Permanently delete")|**Genuinely and permanently removes** that specific version's data|**No** — this is destructive and cannot be undone|

> In the console, deleting a specific version ID requires typing "**permanently delete**" as a confirmation, emphasizing that this action is irreversible — this is different from a normal delete, which merely types "delete" and adds a delete marker.

### 4.4 Important Notes / Exam Gotchas

- Any object uploaded **before** versioning was enabled will have version ID **`null`**.
- **Suspending** versioning does **not** delete any previously created versions — it is a **safe operation** that simply stops the creation of new versions going forward.
- Toggling **"Show versions"** in the console reveals hidden version history, including delete markers.

---

## 5. S3 Replication (CRR and SRR)

**S3 Replication** automatically and **asynchronously** copies objects from a **source bucket** to a **destination (target) bucket**.

### 5.1 Two Flavors

|Type|Full Name|Regions|
|---|---|---|
|**CRR**|Cross-Region Replication|Source and destination buckets are in **different** AWS Regions|
|**SRR**|Same-Region Replication|Source and destination buckets are in the **same** AWS Region|

### 5.2 Requirements

- **Versioning must be enabled on both the source and destination buckets** before replication can be configured — this is a hard prerequisite.
- Source and destination buckets **can belong to different AWS accounts**.
- An **IAM Role/permissions** must be granted to the S3 replication service so it has permission to **read** from the source bucket and **write** to the destination bucket.
- Replication happens **asynchronously**, in the background — there will be a short delay (seconds, sometimes longer for larger objects) before data appears in the destination.

### 5.3 Key Behavioral Detail: Only New Objects Are Replicated by Default

- When you **enable** a replication rule, it only replicates objects uploaded **after** the rule was created — it does **not** automatically backfill/replicate pre-existing objects.
- To replicate objects that existed **before** the replication rule was set up, you must use an **S3 Batch Operation**, explicitly choosing to "replicate existing objects." This is a **separate feature** from standard replication.
- Version IDs are preserved during replication: the replicated object in the destination bucket carries the **same Version ID** as in the source bucket.

### 5.4 Use Cases

**Cross-Region Replication (CRR):**

- **Compliance** requirements (e.g., legally mandated data redundancy in a separate geography).
- Providing **lower-latency access** to data for users located near the destination region.
- Replicating data **across AWS accounts**.

**Same-Region Replication (SRR):**

- **Aggregating logs** from multiple buckets into a single, centralized bucket.
- **Live replication** between a **production account** and a **test/staging account** — keeping a duplicate environment in sync for testing.

---

## 6. S3 Static Website Hosting

Amazon S3 can host an entire **static website** (i.e., a site made up of static HTML, CSS, JavaScript, and media files with no server-side processing) directly, and serve it over the public internet.

### 6.1 How It Works

1. Upload your website files (HTML, images, etc.) to a bucket.
2. Enable **Static Website Hosting** (found under the bucket's **Properties** tab).
3. Specify an **Index Document** (e.g., `index.html`) — this acts as the default homepage.
4. Optionally specify an **Error Document** for custom error pages.
5. AWS generates a **bucket website endpoint URL** you can use to access the site.

### 6.2 Website Endpoint URL Format

- The exact URL format **depends on the AWS Region** the bucket lives in, and there are two legacy formatting conventions (one using a **dash**, the other using a **dot** to separate the region from the rest of the URL). The exact format is not critical to memorize — just know that a **dedicated public endpoint URL** is generated once website hosting is enabled.

### 6.3 Public Access Is Mandatory

- Static website hosting **will not work** unless the bucket content is **publicly readable**.
- This requires:
    1. Disabling **Block Public Access** on the bucket.
    2. Attaching a **Bucket Policy** that grants public `s3:GetObject` access (see the Security section above for the exact JSON structure).
- **Troubleshooting tip:** If you get a **`403 Forbidden`** error after enabling website hosting, it almost always means the bucket policy is **not** granting public read access — this is one of the most common configuration mistakes.

---

## 7. Shared Responsibility Model for Amazon S3

|AWS's Responsibility|Customer's Responsibility|
|---|---|
|Infrastructure (global network of data centers/AZs)|Enabling and configuring **S3 Versioning**|
|Durability and availability guarantees of S3|Setting the correct **S3 Bucket Policy**|
|Ability to sustain the concurrent loss of two facilities (data resilience)|Enabling **encryption** (choosing SSE-S3, SSE-KMS, or client-side)|
|Internal configuration and vulnerability management of S3's infrastructure|Enabling **logging and monitoring** (optional — it's off by default)|
|Internal compliance validation of the S3 service itself|Choosing the **most cost-effective storage class** for your data|
||Configuring appropriate **access control** (public vs. private)|

> **Exam Tip:** AWS is responsible for the **security OF the cloud** (the underlying infrastructure), while **you** are responsible for **security IN the cloud** (how you configure and use S3 — versioning, bucket policies, encryption settings, logging, and storage class selection). This is the general Shared Responsibility Model pattern applied specifically to S3.

---

## 8. AWS Snow Family

The **AWS Snow Family** is a set of **physical, portable hardware devices** used to move large volumes of data into or out of AWS, or to perform **compute at the edge** in disconnected/limited-connectivity environments.

### 8.1 Why Use Snow Family Devices? (The Data Transfer Problem)

Transferring very large datasets **over a network connection** can be impractical:

- Example: transferring **100 TB over a 1 Gbps connection** would take approximately **12 days** of continuous, uninterrupted transfer.
- Real-world networks compound this with: **limited bandwidth**, **high network costs**, **shared bandwidth** with other applications/users, and **unstable connections**.
- **Rule of thumb:** if a network-based data transfer would take **more than about a week**, it's generally recommended to consider a **Snowball** device instead.

### 8.2 Snowball Edge Device Types

|Device|Storage Capacity|Primary Purpose|
|---|---|---|
|**Snowball Edge Storage Optimized**|~80 TB or 210 TB (varies by generation)|Large-scale **data migration/storage**|
|**Snowball Edge Compute Optimized**|~28 TB|**Edge computing** — running compute workloads at remote/disconnected sites|

> Device specs and availability change over time and vary by region — the exam does not require memorizing exact current capacities, just the **conceptual distinction** between storage-optimized and compute-optimized devices.

### 8.3 Two Core Use Cases

**1. Data Migration (Import/Export)**

- AWS ships a physical Snowball device to your location.
- You load your data onto the device using your own infrastructure.
- You ship the device back to AWS.
- AWS performs an **import process**, transferring the data from the device into your target Amazon S3 bucket (import) — or the reverse, exporting data from S3 onto the device to send to you (export).

**2. Edge Computing**

- Useful when data is generated **at a remote location with no or limited internet access** — e.g., a truck on the road, a ship at sea, or a remote mining station.
- Because Snowball Edge Compute Optimized devices have onboard compute capability, you can actually **run EC2 instances or AWS Lambda functions directly on the device**.
- This enables **pre-processing data**, running **machine learning inference at the edge**, or **transcoding media** on-site before (or without) sending it back to AWS.

### 8.4 Ordering a Snow Device (Console Workflow)

When ordering a device via the console, you configure:

1. **Job name / job title**
2. **Job type**: Import into S3, Export from S3, or Local compute & storage only (edge computing)
3. **Device selection**: Storage Optimized vs. Compute Optimized
4. **Pricing option**: On-demand or Committed/upfront pricing
5. **S3 data transfer details**: the destination bucket(s)
6. **Security**: encryption settings and a **service IAM role** granting the device permission to write to your S3 buckets
7. **Shipping address and shipping speed** (e.g., one-day or two-day shipping)
8. **Notifications** for tracking job status

### 8.5 Snowball Edge Pricing

- **Data transfer INTO Amazon S3 from a Snowball device is FREE ($0/GB)** — this is a critical exam fact.
- **Data transfer OUT of AWS** (i.e., exporting data from S3 onto a Snowball device to receive it) **does incur charges**.
- **Device usage pricing** comes in two models:
    - **On-Demand**: a one-time service fee per job, which includes a set number of days of usage (e.g., 10 days for the 80 TB device, 15 days for the 210 TB device). **Shipping time does not count** against this usage window (shipping is free/separate). Extra usage beyond the included days is billed per additional day.
    - **Committed/Upfront**: pay in advance for **monthly, 1-year, or 3-year** usage commitments (primarily intended for **edge computing** scenarios). This can offer discounts of **up to ~62%** compared to on-demand pricing, in exchange for a long-term commitment.

> **Exam Tip:** The one line to remember: **everything about Snowball costs money — except transferring data INTO Amazon S3, which is $0/GB.**

### 8.6 Other Snow Family Members (Referenced)

Beyond the Snowball Edge devices discussed in detail, the broader Snow Family also includes:

- **AWS Snowcone** — a much smaller, lightweight, portable device (suitable for extremely tight spaces or when only a small volume of data/edge compute is needed).
- **AWS Snowmobile** — an **exabyte-scale** data transfer solution using a **shipping container pulled by a semi-trailer truck**, intended for the most massive imaginable data migrations.
- **AWS OpsHub** — a **desktop application** that provides a graphical interface to manage Snow Family devices and load/manage data on them, without needing to use the CLI directly.

---

## 9. AWS Storage Gateway (Hybrid Cloud Integration)

While S3 is a fully cloud-native, proprietary object storage technology (unlike, say, Amazon EFS, which speaks the standard NFS protocol usable directly by on-premises servers), many organizations need to **bridge on-premises infrastructure with AWS cloud storage** — this is called a **hybrid cloud** architecture.

### 9.1 Why Hybrid Cloud?

Common reasons an organization keeps part of its infrastructure on-premises and part in the cloud:

- A **long-running or gradual cloud migration** — legacy on-premises systems that haven't been fully migrated yet.
- **Security or compliance requirements** mandating that certain data remain on-premises.
- A deliberate **long-term IT strategy** that blends on-premises and cloud infrastructure.

### 9.2 AWS Storage Options Recap (For Context)

|Storage Type|AWS Service|
|---|---|
|**Block storage**|Amazon EBS, or an EC2 **Instance Store**|
|**File storage** (network file system)|Amazon EFS|
|**Object storage**|Amazon S3 / S3 Glacier|

### 9.3 What Is AWS Storage Gateway?

- **AWS Storage Gateway** is the service that **bridges on-premises data/systems with AWS cloud storage**, allowing on-premises applications to seamlessly leverage the cloud to **extend their storage capacity**.
- Under the hood, Storage Gateway uses **Amazon EBS, Amazon S3, and Amazon S3 Glacier** as its backing storage in AWS.
- **Use cases**: disaster recovery, backup and restore, and tiered storage strategies that combine fast local access with cheap cloud-based capacity.

### 9.4 Types of Storage Gateway (Mentioned, Not Deep-Dived)

- **File Gateway**
- **Volume Gateway**
- **Tape Gateway**

> **Exam Tip:** For the **Cloud Practitioner (CLF-C02)** exam, you do **not** need to memorize the technical differences between File, Volume, and Tape Gateway. You only need to understand the **high-level concept**: Storage Gateway is how you connect on-premises file systems/storage to the AWS Cloud, enabling hybrid cloud storage architectures. (Deeper technical knowledge of these gateway types would be expected at the **Solutions Architect Associate** level, not Cloud Practitioner.)

---

## 10. Consolidated Summary — What You Need to Know for the Exam

- **Buckets vs. Objects**: buckets need a **globally unique name** and live in a **specific region**; objects live inside buckets and are identified by a **key**.
- **Security layers**: **IAM policies** (user-based), **S3 Bucket Policies** (resource-based, JSON), **ACLs** (legacy, finer-grained but largely deprecated in favor of bucket policies), and **encryption** (server-side by default, client-side optional).
- **Public access**: controlled via bucket policies **and** the overriding **Block Public Access** settings.
- **Static website hosting**: requires public read access + an index document; a `403 Forbidden` typically means the bucket policy isn't granting public reads.
- **Versioning**: protects against accidental overwrite/deletion and enables rollback; must be explicitly enabled; deletes create a **delete marker** unless you permanently delete a specific version.
- **Replication (CRR/SRR)**: requires **versioning enabled on both buckets**; only replicates **new** uploads (use S3 Batch Operations for existing objects); asynchronous.
- **Storage Classes**: Standard → Standard-IA → One Zone-IA → Glacier Instant Retrieval → Glacier Flexible Retrieval → Glacier Deep Archive → Intelligent-Tiering — trading off cost, availability, and retrieval speed, with **identical (11 nines) durability** across all of them.
- **Snow Family**: physical devices (**Snowcone, Snowball Edge Storage/Compute Optimized, Snowmobile**) for **data migration** and **edge computing**; managed via **AWS OpsHub**; transferring data **into S3 is free**.
- **Storage Gateway**: the hybrid-cloud bridge between on-premises systems and AWS storage (EBS/S3/Glacier behind the scenes).
- **Shared Responsibility**: AWS secures the underlying S3 infrastructure; **you** are responsible for configuring versioning, bucket policies, encryption, logging, and choosing cost-optimal storage classes.