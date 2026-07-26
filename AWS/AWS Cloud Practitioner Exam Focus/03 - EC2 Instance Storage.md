
# Executive Summary

In AWS, compute power needs a place to store data. AWS offers multiple storage solutions tailored to different needs: temporary local disks, persistent network drives, and shared file systems. Additionally, Amazon Machine Images (AMIs) capture the exact software state of a server to quickly launch identical copies, while EC2 Image Builder automates this process.

# Core Concepts Explained

### Elastic Block Store (EBS)

EBS is a persistent network drive you attach to a single EC2 instance, much like plugging a network-based USB stick into a computer. Because it uses the network, there can be slight latency, but it allows your data to survive even if the EC2 instance is terminated. You must specify the size and performance capacity upfront, and each EBS volume is strictly locked to a single Availability Zone (AZ).

### EBS Snapshots

Snapshots are point-in-time backups of your EBS volumes. Even though EBS volumes are locked to one AZ, you can take a snapshot of a volume and restore it into a completely different AZ or even copy it to another AWS Region. To save costs, old backups can be moved to a cheaper Snapshot Archive tier, and you can enable a Recycle Bin to protect against accidental deletions.

### EC2 Instance Store

While EBS is a network drive, an Instance Store is a physical hard drive directly attached to the underlying server hosting your EC2 instance. This provides incredibly high performance and speed for reading and writing data. However, it is ephemeral; if the instance is stopped, terminated, or the underlying hardware fails, all data is permanently lost. It is strictly used for temporary data like caches and buffers.

### Elastic File System (EFS)

EFS is a managed, shared network file system designed exclusively for Linux instances. Unlike EBS, which attaches to one instance in one AZ, EFS can be mounted to hundreds of EC2 instances simultaneously across multiple Availability Zones. You do not plan capacity in advance; you simply pay for the data you store. It also offers an Infrequent Access (EFS-IA) storage class to automatically save costs on files you rarely access.

### Amazon FSx

FSx provides fully managed third-party file systems when EFS or standard storage won't fit your specific architecture. FSx for Windows File Server is a native Windows file system supporting the SMB protocol and Microsoft Active Directory for Windows-based applications. FSx for Lustre (Linux + Cluster) is a high-performance file system specifically built for demanding workloads like High Performance Computing (HPC) and machine learning.

### Amazon Machine Images (AMIs) & EC2 Image Builder

An AMI is a pre-configured template containing an operating system and any installed software needed to launch an EC2 instance. Using custom AMIs drastically speeds up boot times because your required software is already prepackaged and installed. EC2 Image Builder is a free service that automatically creates, tests, and distributes these AMIs on a scheduled basis.

# The Big Picture

Think of an EC2 instance as a blank computer. An AMI is the installation drive that loads the operating system and your favorite software. For the computer's storage, you can choose an EBS volume for a standard, persistent network drive, or an Instance Store if you need blazing-fast but temporary local storage. If you add more computers and need them all to access the exact same files at once, you use EFS (for Linux) or FSx (for Windows) as a shared network folder. If your computer fails, you can easily restore its exact drive state using an EBS Snapshot.

### Storage Comparison Table

|**Feature**|**EBS**|**EFS**|**Instance Store**|
|---|---|---|---|
|**Storage Type**|Network block drive|Network file system|Local physical disk|
|**AZ Scope**|Locked to single AZ|Spans multiple AZs|Tied to underlying host|
|**Attachments**|One instance at a time|Hundreds of instances|Fixed to one instance|
|**Data Persistence**|Survives instance termination|Survives instance termination|Lost if instance stops|
|**Best Use Case**|Standard OS and databases|Shared Linux web servers|Temporary caches and buffers|

# Exam Focus

- **Availability Zone Lock:** EBS volumes are locked to one AZ; EFS works across multiple AZs.
    
- **Transferring Data:** Use EBS Snapshots to move EBS volume data to a different AZ or Region.
    
- **Cost Traps:** You pay for provisioned EBS capacity even if empty, while EFS bills only for used storage.
    
- **Ephemeral Data:** EC2 Instance Store loses all stored data immediately upon instance stop or termination.
    
- **HPC Keyword:** Whenever an exam scenario mentions "High Performance Computing (HPC)", choose FSx for Lustre.
    
- **Shared Responsibility:** AWS handles hardware failure and replication; you handle your own backups and encryption.

- **Delete on Termination:** This EBS volume attribute is **ON by default for the root volume** and **OFF by default for any additional attached volumes** — it controls whether that volume is deleted when the EC2 instance is terminated. Called out directly as exam-relevant.
  
- **AMI Sources:** AMIs come from three places — an **AWS-provided public AMI** (e.g., Amazon Linux 2), your own **custom AMI**, or an **AWS Marketplace AMI** (built and sold by third-party vendors).
  
- **EC2 Image Builder cost trap:** The service itself is free, but you still pay for the EC2 instances it launches during build/test and for AMI storage — it is not free end-to-end.


# Quick Reference Table

|**Concept**|**What it is**|**Key thing to remember**|
|---|---|---|
|**EBS**|Persistent network block storage|Locked to one AZ.|
|**Instance Store**|Local physical hardware storage|High performance, ephemeral data.|
|**EFS**|Shared network file system|Linux only, spans multiple AZs.|
|**FSx for Windows**|Native Windows file system|SMB protocol, Active Directory integration.|
|**FSx for Lustre**|High-performance file system|Built for HPC and machine learning.|
|**Snapshots**|Point-in-time EBS volume backup|Use to move volumes across AZs.|
|**AMI**|Pre-configured server software image|Drastically speeds up boot times.|
