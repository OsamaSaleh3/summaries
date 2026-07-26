
## Executive Summary

AWS employs a layered security model dividing operational burdens between the customer and AWS, while providing comprehensive tooling to detect threats, enforce compliance, protect infrastructure, and encrypt data. This module covers how to monitor configuration drift, guard against network attacks like DDoS, identify vulnerabilities, discover sensitive data, and secure credentials at the foundational CLF-C02 exam level.

## Core Concepts Explained

- **The Shared Responsibility Model**: Security is a partnership. AWS is responsible for the security _of_ the cloud (physical infrastructure, hardware, hypervisors, managed services, and global networking), while the customer is responsible for security _in_ the cloud (data encryption, OS patching on EC2, firewall rules, and IAM policies). Think of it like a landlord who secures the building structure (AWS), while you are responsible for locking your own apartment door (Customer).
    
- **AWS Artifact**: A self-service portal providing on-demand access to AWS’s security and compliance documentation. It allows you to download third-party auditor reports (like ISO, SOC, and PCI certifications) and accept legal agreements (like HIPAA BAA) to prove to internal auditors or regulators that your cloud infrastructure meets strict industry benchmarks.
    
- **Amazon GuardDuty**: An intelligent, continuous threat detection service that uses machine learning and anomaly detection to protect your AWS accounts. It monitors foundational logs (CloudTrail, VPC Flow Logs, and DNS logs) without installing any software to spot malicious activity, such as unauthorized deployments or cryptocurrency mining.
    
- **Amazon Inspector**: An automated vulnerability management service that scans workloads for software vulnerabilities and unintended network exposure. It continuously assesses running EC2 instances (using the Systems Manager agent), container images pushed to Amazon ECR, and Lambda function code against a database of known exposures (CVEs).
    
- **AWS Config**: A fully managed service that records and audits the configuration histories of your AWS resources to monitor compliance over time. You define specific rules (e.g., "no public S3 buckets" or "restrict SSH port 22"), and Config alerts you immediately if a resource drifts into a non-compliant state, documenting who made the change via CloudTrail integration.
    
- **Amazon Macie**: A data security and data privacy service that uses machine learning and pattern matching to discover and protect sensitive data. It scans your Amazon S3 buckets specifically to identify and alert you to Personally Identifiable Information (PII), such as social security numbers or credit card details.
    
- **AWS Security Hub**: A central security management tool that provides a comprehensive view of your security posture across multiple AWS accounts. It aggregates security alerts and findings from various AWS services (like GuardDuty, Inspector, Macie, and Config) and partner solutions into a single, unified dashboard.
    
- **Amazon Detective**: A security investigation service that helps you isolate the root cause of potential security issues or suspicious activities. It automatically collects data from VPC Flow Logs, CloudTrail, and GuardDuty, using machine learning and graph visualizations to map complex security events so you can investigate them rapidly.
    
- **AWS Abuse Team**: A dedicated AWS team you contact when you suspect AWS resources (like specific IP addresses or EC2 instances) are being used for abusive, unauthorized, or illegal purposes. Prohibited behaviors include hosting malware, sending spam, conducting unauthorized port scanning, or launching DDoS attacks.
    
- **Root User Privileges**: The root user is the account owner identity created when the account is first setup, possessing ultimate, absolute access. Because it cannot be restricted by standard permissions, it should be locked away with MFA enabled and never used for daily tasks. Critical actions reserved _only_ for the root user include closing the account, changing account settings (like the account name or email), altering the AWS Support plan, and registering as a seller in the Reserved Instance Marketplace.
    
- **IAM Access Analyzer**: A tool within the IAM console that helps you identify resources within your zone of trust (such as an account or organization) that are shared with external entities. It reviews resource-based policies (like S3 bucket policies or SQS queues) and flags any public or cross-account access so you can remediate unintended security exposures.
    
- **DDoS Protection (AWS Shield & WAF)**: Distributed Denial-of-Service (DDoS) protection stops malicious bots from overwhelming your application servers. AWS Shield Standard provides free, automatic protection against common layer 3 and 4 network attacks for all customers. For premium defenses, Shield Advanced offers 24/7 access to a DDoS Response Team and financial protection against cost spikes, while AWS WAF (Web Application Firewall) operates at Layer 7 (HTTP/HTTPS) to block malicious traffic patterns like SQL injections based on customizable web ACL rules.
    
- **AWS Network Firewall & Firewall Manager**: AWS Network Firewall provides stateful network protection for your entire VPC across layers 3 to 7, sitting at the VPC level rather than individual subnets. AWS Firewall Manager is a central management service that allows you to easily scale and enforce these firewall rules, security groups, and WAF rules across all current and future accounts within an AWS Organization.
    
- **Penetration Testing**: AWS allows customers to conduct security assessments and penetration tests against their own infrastructure for a specific list of 8 core services (like EC2, RDS, and CloudFront) without prior approval. However, performing actual high-impact attacks like simulated DDoS, zone walking, or protocol flooding remains strictly prohibited to protect the underlying shared environment.
    
- **Encryption (KMS vs CloudHSM)**: AWS secures data at rest (stored on disks) and in transit (moving over the network). AWS KMS (Key Management Service) is a managed multi-tenant service where AWS handles the underlying hardware while you manage permissions to keys. CloudHSM provides dedicated, single-tenant cryptographic hardware (FIPS 140-2 Level 3 compliant) where you retain absolute ownership and management of the encryption keys.
    
- **AWS Certificate Manager (ACM)**: A service that handles provisioning, managing, and deploying SSL/TLS certificates. It provides in-flight encryption for your web traffic by exposing HTTPS endpoints on front-end services like Application Load Balancers or CloudFront distributions, and it offers free public certificates with automatic renewal.
    
- **AWS Secrets Manager**: A secure credential management service designed to store, protect, and rotate sensitive secrets like database passwords or API keys. It features native integration with Amazon RDS to automatically rotate passwords using AWS Lambda functions every X days, keeping credentials encrypted using KMS keys.
    

### Comparisons of Commonly Confused Services

| **Service**          | **Primary Scope**            | **What It Analyzes**                                                                                         |
| -------------------- | ---------------------------- | ------------------------------------------------------------------------------------------------------------ |
| **Amazon GuardDuty** | Intelligent Threat Detection | Account activity and network logs (CloudTrail, VPC Flow Logs, DNS logs) for active threats or crypto-mining. |
| **Amazon Inspector** | Vulnerability Assessment     | Software bugs, known vulnerabilities (CVEs), and unintended network paths in EC2, ECR, and Lambda.           |
| **Amazon Macie**     | Data Privacy & Protection    | Specifically scans **Amazon S3 buckets** to discover and classify sensitive data like PII.                   |
| **AWS Config**       | Configuration & Compliance   | Tracks configuration changes over time and audits them against predefined compliance rules.                  |

|**Feature**|**AWS Shield**|**AWS WAF (Web Application Firewall)**|
|---|---|---|
|**Layer of Operation**|Layer 3 & 4 (Network & Transport)|Layer 7 (Application layer - HTTP/HTTPS)|
|**Primary Protection**|Mass-scale volumetric DDoS attacks|Web exploits, SQL injections, Cross-Site Scripting (XSS), bad bots|
|**Pricing Tier**|Standard (Free/Default) & Advanced ($3k/mo)|Paid per Web ACL, rule, and request volume|

|**Feature**|**AWS KMS**|**AWS CloudHSM**|
|---|---|---|
|**Tenancy & Management**|Shared, multi-tenant managed service. AWS manages hardware/software.|Dedicated, single-tenant physical Hardware Security Module.|
|**Key Control**|Keys can be AWS-managed or Customer-managed within KMS.|You have exclusive, absolute control over the keys inside the hardware.|
|**Compliance Level**|Managed security standards|FIPS 140-2 Level 3 tamper-resistant hardware|

|**Service**|**Core Responsibility**|**Scale / Purpose**|
|---|---|---|
|**AWS Network Firewall**|Filters traffic across the entire VPC|High-performance stateful network filtering (Layers 3-7)|
|**AWS Firewall Manager**|Centrally manages rules across an Organization|Scales security groups, WAF rules, and Network Firewall rules to multiple accounts|

## The Big Picture

Securing a cloud application requires multiple layers working in tandem. First, **AWS Shield** and **WAF** filter malicious traffic at the edge before hitting your architecture. Inside the network, **AWS Network Firewall** protects the VPC, while traffic reaching your application load balancer is encrypted in transit using certificates managed by **AWS Certificate Manager (ACM)**. The application handles data using credentials safely rotated by **AWS Secrets Manager**, and writes records to an S3 bucket encrypted via **AWS KMS**.

Simultaneously, operational security tools monitor the environment: **AWS Config** tracks infrastructure changes, **Amazon Inspector** scans the server for operating system flaws, **Amazon Macie** watches for exposed PII, and **Amazon GuardDuty** keeps an eye out for account-level threats. Finally, all these alerts flow into **AWS Security Hub** for a single pane of glass view, while **Amazon Detective** stands ready to trace the root cause if an incident occurs.

## Exam Focus

- **Keywords**: "PII/Sensitive data in S3" = Macie. "Cryptocurrency/Anomaly/Threat detection" = GuardDuty. "Vulnerabilities/CVE/EC2 software scan" = Inspector. "Configuration history/Compliance drift" = Config. "Compliance reports/ISO/SOC/HIPAA" = Artifact. "Rotate RDS passwords" = Secrets Manager.
    
- **Cost Traps**: AWS Shield Standard is free, but Shield Advanced costs a steep $3,000/month. AWS Config charges per configuration item recorded, and AWS KMS charges for customer-managed keys ($1/month) plus API usage, whereas AWS-managed keys are free. CloudHSM requires paying for a dedicated physical appliance and is significantly more expensive than KMS.
    
- **Scenario Triggers**: If a question mentions enforcing security groups across _multiple accounts in an AWS Organization_, select Firewall Manager. If the question involves finding out _which internal resources are shared with the public or external accounts_, choose IAM Access Analyzer. If an AWS IP address is launching attacks on you, contact the Abuse Team.
    

## Quick Reference Table

|**Concept**|**What it is**|**Key thing to remember**|
|---|---|---|
|**Shared Responsibility**|Split security model|AWS = _Of_ the Cloud; Customer = _In_ the Cloud|
|**AWS Artifact**|Portal for compliance documents|Download ISO/SOC reports & accept agreements|
|**Amazon GuardDuty**|ML threat detection service|Catches crypto-mining via logs; no agents needed|
|**Amazon Inspector**|Vulnerability scanner|Scans EC2, ECR images, and Lambda for CVEs|
|**AWS Config**|Resource recorder and rule auditor|Tracks configuration history and drift|
|**Amazon Macie**|PII identification tool|Only scans S3 buckets for sensitive data|
|**Security Hub**|Central alert aggregator|Dashboard showing multi-account security postures|
|**Amazon Detective**|Root-cause analysis graph|Simplifies security investigations with ML|
|**AWS Abuse Team**|Illegal resource usage reporting|Contact for spam, malware, or attacks from AWS|
|**Root User**|Primary account owner identity|Must lock away; handles closing account and billing|
|**IAM Access Analyzer**|External sharing detector|Finds resources exposed outside Zone of Trust|
|**AWS Shield**|Volumetric DDoS mitigation|Standard is free; Advanced offers a 24/7 team|
|**AWS WAF**|Layer 7 web filter|Blocks SQL injections/XSS with rate rules|
|**Network Firewall**|Whole-VPC network filter|Replaces subnets-only rules with VPC-wide security|
|**Firewall Manager**|Policy scaler across an Organization|Automatically applies security groups to new accounts|
|**Pen Testing**|Customer-led security testing|Permitted on 8 core services; DDoS tests banned|
|**AWS KMS**|Shared key management system|Standard option for encryption at rest|
|**CloudHSM**|Dedicated tamper-proof hardware|Single-tenant; absolute key ownership|
|**ACM**|SSL/TLS certificate manager|Handles in-flight HTTPS encryption automatically|
|**Secrets Manager**|Credential locker with auto-rotation|Integrates with RDS to rotate passwords via Lambda|