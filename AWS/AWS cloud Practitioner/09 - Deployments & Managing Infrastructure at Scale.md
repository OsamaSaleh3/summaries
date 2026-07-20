
## Executive Overview

Deploying and managing infrastructure is one of the most important operational themes in the AWS Cloud, and it forms a significant knowledge domain for the **AWS Certified Cloud Practitioner (CLF-C02)** exam. In the early stages of learning AWS, most people create resources manually through the AWS Management Console — launching EC2 instances by hand, clicking through wizards, and configuring security groups one at a time. While this is fine for learning, it does **not scale** in real-world production environments where you may need to manage dozens, hundreds, or even thousands of resources, replicate environments across multiple AWS accounts and Regions, and ensure consistency, auditability, and speed.

This topic covers the ecosystem of AWS tools that solve this problem, broadly split into two categories:

1. **Infrastructure Deployment & Management Services** — CloudFormation, the CDK, Elastic Beanstalk, CodeDeploy, and Systems Manager (SSM). These help you provision, configure, and maintain the underlying infrastructure (servers, networking, databases, etc.) at scale.
2. **Developer / CI-CD Tooling Services** — CodeCommit, CodeBuild, CodePipeline, and CodeArtifact. These help developers manage source code, build and test it, orchestrate the release pipeline, and manage software dependencies.

Together, these services embody the AWS best practice of **Infrastructure as Code (IaC)** and **Continuous Integration/Continuous Delivery (CI/CD)** — two concepts that are central not just to the exam, but to how modern cloud teams actually operate. Understanding *what each service does*, *when to use it*, and *how it differs from similar services* is critical exam knowledge.

---

## 1. AWS CloudFormation

### 1.1 What is CloudFormation?

**AWS CloudFormation** is AWS's flagship **Infrastructure as Code (IaC)** service. It provides a **declarative** way to define your AWS infrastructure using text-based templates (written in **YAML** or **JSON**).

> **Declarative vs. Imperative**: In a declarative model, you describe the *end state* you want ("I want two EC2 instances, an S3 bucket, and a load balancer"), and the tool figures out *how* to achieve that state. This differs from an imperative approach, where you would write step-by-step instructions for exactly how to create each resource in order.

**Example concept**: You could write a CloudFormation template that says:
- I want a security group
- I want two EC2 instances that use that security group
- I want an S3 bucket
- I want a load balancer in front of those instances

CloudFormation reads this template and automatically provisions **all** of these resources, **in the correct dependency order**, with the exact configuration you specified — without you needing to manually figure out that, for example, the security group must exist before the EC2 instances that reference it.

### 1.2 Key Benefits of CloudFormation

#### a) Infrastructure as Code
- You **never manually create resources** through the console. Everything is defined in code (templates).
- This is excellent for **control** — every infrastructure change must go through **code review**, just like application code. This enforces governance, traceability, and reduces human error ("ClickOps").
- Because it's code, it can be stored in version control (e.g., Git), enabling rollback, diffing, and collaboration.

#### b) Cost Benefits
- Every resource created within a CloudFormation **stack** automatically receives **tags** that identify it as belonging to that stack. This makes cost allocation and tracking significantly easier (e.g., using AWS Cost Explorer or Cost Allocation Tags).
- You can **estimate costs** of a template before deployment.
- **Automated cost-saving strategies** are possible: for example, you could schedule the deletion of an entire non-production stack at 5:00 PM and recreate it at 8:00 or 9:00 AM the next day. Since all resources tied to the stack are deleted, **you pay nothing for that idle period**, and the exact same environment can be safely recreated the next morning.
- CloudFormation makes it trivial to **create and destroy environments on demand**, which aligns with one of the core cloud computing principles: **elasticity / paying only for what you use**.

#### c) Productivity Benefits
- Ability to **destroy and recreate** infrastructure on the fly (e.g., for testing, disaster recovery drills, or spinning up temporary environments).
- CloudFormation can **auto-generate architecture diagrams** of your templates (via the **Infrastructure Composer**, previously known as Application Composer).
- **Declarative programming** means you don't need to manually sequence resource creation (e.g., "create the DynamoDB table before the EC2 instance"). CloudFormation's internal dependency engine figures out the correct order automatically.

#### d) "Don't Reinvent the Wheel"
- You can leverage a huge ecosystem of **existing/community templates** available online.
- CloudFormation has extensive **official AWS documentation**.
- It supports **almost all AWS resources**.
- For resources CloudFormation doesn't natively support, you can define a **Custom Resource**, which lets you plug in custom provisioning logic (often backed by an AWS Lambda function) to manage that resource within the same stack lifecycle.

> CloudFormation is considered the **base/foundation of Infrastructure as Code on AWS**.

### 1.3 Visualizing Templates — AWS Infrastructure/Application Composer

AWS provides a visual tool (referred to in the lecture as **Infrastructure Composer**, and in the console commonly as **Application Composer**) that lets you:
- Upload or view a CloudFormation template
- See a **visual canvas** of every resource defined in the template (e.g., ALB Listener, database security group, RDS database, other security groups, Launch Configuration, Application Load Balancer, etc.)
- See the **relationships/connections** between these components (e.g., which security group is attached to which instance)

This is extremely useful for understanding complex architectures at a glance without manually parsing hundreds of lines of YAML/JSON.

### 1.4 Hands-On Walkthrough (Core Mechanics)

The instructor demonstrated CloudFormation using the AWS Console:

1. **Creating a Stack**: In the CloudFormation console, you choose to create a stack, and you can:
   - Use an existing template (upload a file)
   - Use an AWS-provided sample template
   - Build visually using Application/Infrastructure Composer

2. **Region and AMI Scoping**: The demo emphasized that **AMI IDs are Region-scoped** — an AMI ID that works in `us-east-1` will *not* necessarily be valid in another Region. This is why templates that hardcode an AMI ID or an Availability Zone (e.g., `us-east-1a`) must be deployed in the matching Region.

3. **Simple Template Example**: A template with a single `AWS::EC2::Instance` resource, specifying properties like Availability Zone, Image ID (AMI), and Instance Type (`t2.micro`).

4. **Stack Parameters**: CloudFormation templates can define **Parameters** — placeholders that are filled in at deploy time (e.g., asking the user "What is your security group description?"). This makes templates **reusable** across environments.

5. **Tags on Stacks**: You can apply custom tags (e.g., `CFDemo`) to a stack, and CloudFormation automatically propagates:
   - The **stack name**
   - The **logical ID** of the resource
   - The **stack ID**
   - Any custom tags you defined
   
   ...onto every resource created within that stack.

6. **Updating a Stack & Change Sets**: When you update a stack with a modified template (e.g., adding an Elastic IP and two security groups to an existing EC2 instance), CloudFormation generates a **Change Set** — a preview of exactly what will be **added**, **modified**, or **removed**, *before* you apply it.
   - Critically, the Change Set will flag if a resource requires **Replacement** (`Replacement: True`). This means the existing resource will be **deleted and a brand-new one created** in its place — a crucial thing to know if that resource holds important data (e.g., an EC2 instance with local, non-persistent storage).
   - In the demo, changing certain EC2 instance properties forced a replacement: a **new EC2 instance was created first**, then the **old one was terminated** (cleanup) — CloudFormation intelligently sequences these operations.

7. **Stack Deletion**: Deleting a CloudFormation stack automatically deletes **all** resources within it, in the **correct reverse-dependency order**. 
   > **Best Practice / Exam Tip**: Never manually delete or modify resources that were created by a CloudFormation stack. Always update or delete via CloudFormation itself — manual changes cause **configuration drift**, where the real-world state no longer matches what's defined in the template.

### 1.5 Exam Perspective

- Use CloudFormation when you need:
  - **Infrastructure as Code**
  - To **repeat/replicate an architecture** across different **environments**, **Regions**, or even different **AWS accounts**
- CloudFormation is **AWS-only** (it cannot manage non-AWS/third-party or on-premises resources the way some other IaC tools like Terraform can).

---

## 2. AWS Cloud Development Kit (CDK)

### 2.1 What is the CDK?

The **AWS Cloud Development Kit (CDK)** allows you to define your cloud infrastructure using a **familiar general-purpose programming language** — such as **JavaScript, TypeScript, Python, Java, or .NET** — instead of writing raw YAML/JSON CloudFormation templates.

### 2.2 How It Works

1. You write your infrastructure definitions as code, in your chosen programming language, using CDK "constructs" (e.g., defining a Lambda function, a DynamoDB table, a VPC, an ECS cluster, etc.).
2. You run the **CDK CLI**, which **compiles/synthesizes** your code into a standard **CloudFormation template** (in JSON or YAML).
3. That generated CloudFormation template is then deployed through CloudFormation as usual, provisioning the actual AWS resources.

### 2.3 Why Use the CDK?

- **Same language for infrastructure and application code**: This is particularly powerful for services like **AWS Lambda** (serverless functions) and **containerized workloads on ECS/EKS**, where your infrastructure definition and your actual runtime application logic can be written and maintained in the same programming language and repository.
- **Type safety**: Using a real programming language gives you compiler/IDE support, catching errors before deployment.
- **Familiar constructs**: Developers can use loops, conditionals, functions, and classes — things not natively available in declarative YAML/JSON — to reduce duplication and increase reusability.
- **Faster development**: Reusable components and abstractions (called *Constructs*) speed up building complex architectures.

### 2.4 Example

A CDK application in Python might define a Lambda function and a DynamoDB table. A CDK application in JavaScript/TypeScript might define a VPC, an ECS Cluster, and an Application Load Balancer with a Fargate service. In both cases, running the CDK CLI transforms this code into a deployable CloudFormation template.

### 2.5 Exam Tip
- **CDK compiles down to CloudFormation** — it is *not* a replacement for CloudFormation, but rather a developer-friendly abstraction layer on top of it.
- Know the distinction: CloudFormation = declarative templates (YAML/JSON); CDK = imperative code (real programming languages) that generates those templates.

---

## 3. AWS Elastic Beanstalk

### 3.1 The Problem Beanstalk Solves

Most web applications deployed on AWS follow a similar **3-tier architecture**:
- Users connect to a **Load Balancer** (spanning multiple Availability Zones for high availability)
- The Load Balancer forwards traffic to multiple **EC2 instances** managed by an **Auto Scaling Group (ASG)**
- The EC2 instances read/write data to a **database**, typically:
  - **Amazon RDS** for relational data
  - **Amazon ElastiCache** for in-memory caching (e.g., session data)

You *could* build this manually, or automate it with raw CloudFormation — but as a developer, you often don't want to be responsible for configuring load balancers, ASGs, and databases from scratch every time. You just want to **deploy your code** and have AWS handle the infrastructure.

### 3.2 What is Elastic Beanstalk?

**AWS Elastic Beanstalk** is a **developer-centric** service for deploying web applications, and it is AWS's primary example of a **Platform as a Service (PaaS)** offering.

> **Service Model Recap**:
> - **IaaS (Infrastructure as a Service)** — you manage the OS, runtime, and applications; AWS manages the hardware (e.g., raw EC2).
> - **PaaS (Platform as a Service)** — AWS manages the underlying infrastructure, OS patches, and provisioning; you focus only on your application code (e.g., **Elastic Beanstalk**).
> - **SaaS (Software as a Service)** — a fully finished software product is delivered to you (e.g., Gmail, or many AWS managed services consumed as end products).

Behind the scenes, Beanstalk still uses the **same underlying components** as a manually built architecture — EC2 instances, an Auto Scaling Group, an Elastic Load Balancer, an RDS database, etc. — but presents them through **one unified, developer-friendly view**, while still allowing you to configure the underlying components if needed.

### 3.3 Key Characteristics

- **Beanstalk itself is free** — you only pay for the **underlying resources** it provisions (EC2 instances, load balancer, RDS, etc.).
- It is a **managed service**: 
  - EC2 instance configuration and the OS are handled by Beanstalk.
  - The **deployment strategy** is configurable, but the actual deployment execution is performed by Beanstalk.
  - **Capacity provisioning** (via Auto Scaling Group) and **load balancing** are automatically handled.
  - **Application health monitoring** is built directly into the Beanstalk dashboard.
- **Your responsibility as the developer**: essentially just the **application code**. This is what makes Beanstalk highly developer-friendly.

### 3.4 The Three Architecture Models

| Model | Description | Best For |
|---|---|---|
| **Single Instance Deployment** | One EC2 instance, no load balancer | **Development** environments (free-tier eligible) |
| **Load Balancer + Auto Scaling Group (ASG)** | Full high-availability web tier | **Production** or pre-production web applications |
| **ASG only (no load balancer)** | For non-web-facing background processing | **Worker/non-web applications** (e.g., queue processors) |

### 3.5 Supported Platforms

Elastic Beanstalk supports many application platforms, including:
- **Go**
- **Java**
- **.NET**
- **Node.js**
- **PHP**
- **Python**
- **Ruby**
- **Packer**
- **Docker** (single container)
- **Multi-container Docker**
- **Preconfigured Docker**

*(You do not need to memorize this full list for the exam — the key takeaway is that Beanstalk supports a wide range of languages and container-based deployments.)*

### 3.6 Health Monitoring

- Each EC2 instance managed by Beanstalk runs a **health agent** that pushes metrics to **Amazon CloudWatch**.
- Within the Beanstalk console, you get a unified view for **monitoring**, **health checks**, and **application health events**.

### 3.7 Hands-On Walkthrough Highlights

The instructor demonstrated creating a Beanstalk application via the console:

1. **Environment Tiers**: When creating an application, you choose between:
   - **Web server environment** (for running a website/web application)
   - **Worker environment** (for processing tasks off of a queue)

2. **Application & Environment Naming**: An "Application" (e.g., `MyApplication`) can have multiple "Environments" underneath it (e.g., `MyApplication-dev`, `MyApplication-prod`), allowing the same codebase to be deployed differently across dev, test, and prod.

3. **Domain Name**: Beanstalk automatically generates a domain name to access your deployed web application.

4. **Platform Selection**: You select a managed platform (e.g., Node.js) and either upload your own code or use a **sample application**.

5. **Presets/Configuration**: 
   - **Single instance** (free-tier eligible, simplest)
   - **High availability** (load balancer + ASG)
   - **Custom configuration** (full control)

6. **IAM Roles Required**:
   - A **Service Role** (`elasticbeanstalk-service-role`) — allows the Beanstalk service itself to perform actions on your behalf.
   - An **EC2 Instance Profile** (`aws-elasticbeanstalk-ec2-role`) — attached to the EC2 instances themselves, with policies like **WebTier**, **WorkerTier**, and **MulticontainerDocker** permissions, allowing the instances to function correctly within Beanstalk.

7. **Under the Hood — CloudFormation**: A key insight from the demo: **Elastic Beanstalk uses AWS CloudFormation behind the scenes** to actually provision all the resources (Auto Scaling Group, Launch Configuration, Elastic IP, Security Groups, etc.). You can view the generated CloudFormation stack, its events (`CREATE_IN_PROGRESS` → `CREATE_COMPLETE`), and even visualize it in Application Composer.

8. **Console Features After Deployment**:
   - **Upload and Deploy a new version**: Push new application code, which Beanstalk automatically deploys to the underlying EC2 instances.
   - **Health**: Health check status of all instances.
   - **Logs**: View application logs.
   - **Monitoring**: View CloudWatch metrics for the application.
   - **Alarms**: Set up alerting.
   - **Managed updates**: Configure how/when Beanstalk updates the underlying platform/environment.
   - **Configuration**: View and modify all underlying configuration settings.

9. **Cleanup**: You can delete a Beanstalk application via **Actions → Delete Application**, which cleans up all associated resources.

### 3.8 Beanstalk vs. CloudFormation

| | **Elastic Beanstalk** | **CloudFormation** |
|---|---|---|
| Focus | **Code and environments** for your application | **Arbitrary infrastructure stacks** — any AWS resources, not just web app patterns |
| Level of abstraction | High — developer-centric, opinionated architecture | Low — you define everything explicitly |
| Underlying mechanism | Uses CloudFormation internally | Is the underlying IaC engine |

### 3.9 Exam Tip
- Beanstalk = **PaaS**, developer-focused, code + environment management.
- Beanstalk is **AWS-only** and limited to supported programming languages/Docker.
- Remember: Beanstalk **provisions using CloudFormation under the hood**.

---

## 4. AWS CodeDeploy

### 4.1 What is CodeDeploy?

**AWS CodeDeploy** is a service purpose-built to **automate application deployments and upgrades** onto servers — moving an application from **Version 1 to Version 2**, for example — **without requiring Beanstalk or CloudFormation**. It operates completely independently of those services.

### 4.2 Where CodeDeploy Works

CodeDeploy is a **Hybrid Service** — it works with:
1. **Amazon EC2 instances** — upgrading many EC2 instances from V1 to V2 of an application.
2. **On-Premises servers** — upgrading applications on physical/virtual servers outside of AWS.

This dual capability makes CodeDeploy especially valuable for organizations undergoing a **hybrid cloud transition**, allowing them to use the **same deployment mechanism and workflow** whether the target is an on-premises server or an EC2 instance in AWS.

### 4.3 Requirements

- **You must provision the servers ahead of time** — CodeDeploy does not create the underlying compute for you (unlike Beanstalk or CloudFormation, which can provision infrastructure from scratch).
- **You must install and configure the CodeDeploy Agent** on each target server. This agent is what carries out the actual deployment/upgrade instructions issued by the CodeDeploy service.

### 4.4 Exam Tip
- CodeDeploy = deploying/upgrading application code **onto already-existing servers** (EC2 or on-premises).
- It is considered an **advanced service** — for the exam, you mainly need to remember: it upgrades applications from one version to another, across EC2 and on-premises servers, from a single interface, and it requires the CodeDeploy Agent to be installed beforehand.

---

## 5. AWS Systems Manager (SSM)

### 5.1 What is Systems Manager?

**AWS Systems Manager (SSM)** helps you manage your **fleet** of EC2 instances *and* **on-premises systems** at scale. Because it manages both AWS and non-AWS infrastructure, it is classified as a **Hybrid AWS Service**.

SSM offers a large suite (10+) of products/features, but for the Cloud Practitioner exam, the most important capabilities are:

- **Automated Patching** — Patch your fleet of servers/instances automatically for **compliance** purposes.
- **Run Command** — Execute a command across your **entire fleet** of servers simultaneously, directly from SSM.
- **Parameter Store** — Securely store configuration data and secrets.

SSM works across multiple operating systems: **Linux, Windows, macOS, and Raspberry Pi**.

### 5.2 How SSM Works (Architecture)

1. You must install the **SSM Agent** on every system you want to manage (both EC2 instances and on-premises VMs).
   - The SSM Agent comes **pre-installed by default** on **Amazon Linux AMIs** and some **Ubuntu AMIs**.
2. The SSM Agent continuously reports back to the **SSM service** in AWS.
3. Because the agent connects both EC2 instances *and* on-premises virtual machines back to the same central SSM service, this architecture is what makes SSM a **hybrid service**.
4. If an instance "cannot be controlled by SSM," the most likely cause is a **missing or misconfigured SSM Agent**.
5. Once the agent is properly installed and reporting, you can use SSM to **run commands**, **patch**, or **configure** all connected servers consistently and at scale.

### 5.3 Exam Perspective
- See "patch a fleet of EC2 instances or on-premises servers" → **think SSM**.
- See "run a command consistently across many servers" → **think SSM**.

---

## 6. SSM Session Manager

### 6.1 What is Session Manager?

**Session Manager** is a feature within Systems Manager that lets you start a **secure shell session** on your EC2 instances (or on-premises servers) **without**:
- Needing SSH access
- Needing a bastion host
- Needing SSH keys

Because SSH access is no longer required, you can **close port 22 entirely** on your EC2 instance's security group — resulting in a **significantly improved security posture**.

### 6.2 How It Works

- The EC2 instance runs the **SSM Agent**, which connects to the **Session Manager** service.
- Authorized users connect through the **Session Manager service** (not directly to the instance via a network port) to execute commands.
- Session Manager supports **Linux, macOS, and Windows**.
- **Session log data** can optionally be sent to **Amazon S3** or **Amazon CloudWatch Logs** for auditing and security purposes.

### 6.3 Hands-On Walkthrough Highlights

1. **EC2 instance launched with a security group that has zero inbound rules** (no HTTP, HTTPS, or SSH) — proving that Session Manager truly doesn't need any inbound network access.
2. **IAM Instance Profile is required**: The EC2 instance must have an attached IAM Role with a policy such as **AmazonSSMManagedInstanceCore**, which grants the instance permission to communicate with the SSM service.
3. **Fleet Manager**: Within the SSM console, the **Fleet Manager** page shows all EC2 instances (and on-premises servers) registered with SSM — these are called **Managed Nodes**. Once the instance boots and the agent registers, it appears here along with details like SSM Agent status, OS platform, and agent version.
4. **Starting a Session**: From **Session Manager**, you select the managed instance and "Start Session" to get an interactive secure shell — all **without SSH keys or open ports**.
5. **Session History**: After ending a session, the **session history/logs** are retained for auditing.

### 6.4 The Three Ways to Access an EC2 Instance (Summary/Comparison)

| Method | Requires Port 22 Open? | Requires SSH Keys? | Notes |
|---|---|---|---|
| **Traditional SSH** | Yes | Yes | Classic terminal-based SSH access |
| **EC2 Instance Connect** | Yes | No (temporary keys pushed automatically) | Still requires port 22 open |
| **SSM Session Manager** | **No** | **No** | Most secure; requires SSM Agent + IAM role; no open inbound ports needed |

### 6.5 Exam Tip
- Session Manager = **more secure alternative to SSH**, no need for bastion hosts, SSH keys, or open port 22.
- Requires: (1) SSM Agent installed and running, (2) an IAM Role/Instance Profile granting the instance access to SSM.

---

## 7. SSM Parameter Store

### 7.1 What is Parameter Store?

**SSM Parameter Store** is a secure, serverless way to store **configuration data and secrets** on AWS — such as API keys, database passwords, and application configuration values.

### 7.2 Key Characteristics

- **Serverless** — no infrastructure to manage.
- **Scalable** — can respond to a very high volume of API calls.
- **Durable** and **easy to use**.
- **Secure** — access to each individual parameter is controlled via **IAM policies**.
- **Version Tracking** — parameter values are versioned, so you can track how a configuration has changed over time.
- **Optional Encryption** — supports both **plaintext** and **encrypted** values (encryption is handled via **AWS KMS – Key Management Service**).

### 7.3 Parameter Types

- **String** — a plain configuration value (e.g., a URL, a setting).
- **StringList** — a list of values.
- **SecureString** — an **encrypted** value (via KMS), recommended for sensitive data like API keys and passwords.

### 7.4 Tiers
- **Standard** tier — free, used for typical configuration/parameter storage needs.
- **Advanced** tier — paid, supports higher parameter size and other advanced features (policies, more parameters per account).

### 7.5 Hands-On Walkthrough Highlights

- Navigate to **Systems Manager → Parameter Store → Create Parameter**.
- Provide a **name**, choose a **tier** (Standard/Advanced), choose a **type** (String, StringList, SecureString), choose a **data type** (text or EC2 image type), and enter the **value**.
- Once created, the parameter's value can be retrieved on demand, and prior **versions** are tracked if the parameter is edited over time.

### 7.6 Exam Tip
- Use **Parameter Store** for centrally managing **configuration values and secrets** across many applications in one place, with fine-grained IAM-based access control.
- For more advanced secrets management with **automatic rotation**, note that AWS also offers **Secrets Manager** as a separate, more feature-rich (but paid) service — Parameter Store is the simpler/cheaper general-purpose configuration store.

---

## 8. AWS CodeCommit *(Discontinued Service — Important Context)*

### 8.1 What Was CodeCommit?

**AWS CodeCommit** was AWS's own fully-managed **Git-based source code repository** service — effectively AWS's answer to **GitHub**. It allowed teams to:
- Store code in a **version-controlled**, private Git repository
- Enable developer collaboration
- Automatically version code changes and support rollback
- Keep code **private and secure** within your AWS account, tightly integrated with other AWS services

### 8.2 Important Exam Note — Service Discontinuation

> **As of July 25, 2024, AWS discontinued CodeCommit for new customers.** New customers can no longer sign up to use the service, and AWS recommends migrating to another Git solution such as **GitHub**, **GitLab**, or another third-party Git provider.

Despite this, **CodeCommit may still appear on the exam** (since exam content updates lag behind real-world product changes). The key exam guidance:
- Wherever you see "CodeCommit" mentioned in course materials or exam questions, mentally treat it as **interchangeable with a GitHub integration** — CodeCommit was simply AWS's own Git repository offering, and its conceptual role in a pipeline (source code storage) is unaffected by the discontinuation.

### 8.3 Exam Tip
- Know that CodeCommit = a (now discontinued) **fully managed private Git repository service**.
- Know that AWS's suggested replacements are **GitHub, GitLab,** or **third-party Git providers**.

---

## 9. AWS CodeBuild

### 9.1 What is CodeBuild?

**AWS CodeBuild** is a fully managed service that **builds your code in the cloud**. This means:
1. Source code is **compiled**
2. **Tests** are run
3. The output is packaged into **deployable artifacts**, ready to be used by a deployment tool such as **CodeDeploy**

### 9.2 Typical Flow

```
CodeCommit (or GitHub) → CodeBuild (retrieves code, runs build script) → Deployable Artifact → CodeDeploy (deploys to servers)
```

CodeBuild retrieves the source code (e.g., from a Git repository), runs a **build script that you define**, and produces a **ready-to-deploy artifact**.

### 9.3 Key Benefits

- **Fully managed and serverless** — no build servers to provision or maintain.
- **Continuously scalable** and **highly available**.
- **Secure**.
- **Pay-as-you-go pricing** — you only pay for the **compute time your build actually consumes**. There are no idle servers to pay for.

### 9.4 Exam Tip
- CodeBuild = **compiling, testing, and packaging** code in a serverless, pay-per-use manner.
- It is typically triggered every time new code is pushed to a repository (e.g., CodeCommit).

---

## 10. AWS CodeArtifact

### 10.1 The Problem: Artifact/Dependency Management

Software packages that developers write typically **depend on other packages** to build and run — these are called **code dependencies**. Managing where these dependencies are stored and retrieved from is called **artifact management**.

Traditionally, teams had to build and maintain their own artifact management infrastructure — for example, hosting packages on **Amazon S3** or running custom software on **EC2 instances**. This is complex and operationally burdensome.

### 10.2 What is CodeArtifact?

**AWS CodeArtifact** is a **secure, scalable, and cost-effective managed artifact management service** for software development. It removes the need to build and operate your own dependency-hosting infrastructure.

### 10.3 Supported Tools

CodeArtifact integrates with the dependency management tools developers already use, including:
- **Maven**
- **Gradle**
- **npm**
- **yarn**
- **twine**
- **pip**
- **NuGet**

### 10.4 How It Fits into the Pipeline

Once code is pushed to a repository (e.g., CodeCommit) and built with **CodeBuild**, CodeBuild can retrieve any needed dependencies **directly from CodeArtifact** during the build process. This gives development teams a **single, secure, centralized location** to both **store** and **retrieve** their code dependencies.

### 10.5 Exam Tip
- CodeArtifact = managed **artifact/dependency management** — think of it whenever a scenario mentions a team needing a central, secure place to store and retrieve software package dependencies.

---

## 11. AWS CodePipeline

### 11.1 What is CodePipeline?

**AWS CodePipeline** is the **orchestration layer** that connects together the various stages of getting code from a repository into production. It answers the question: *"How do CodeCommit and CodeBuild (and other services) actually get connected into an automated workflow?"*

### 11.2 The CI/CD Concept

**CI/CD** stands for **Continuous Integration and Continuous Delivery (or Deployment)**. It is the overarching practice where:
- Every time a developer **pushes code** to a repository
- The code is automatically **built**, **tested**, and **deployed** to servers

This automation removes manual, error-prone release processes and enables **fast, frequent, and reliable releases**.

### 11.3 Example Pipeline Flow

A typical CodePipeline might:
1. Pull source code from **CodeCommit** (or GitHub)
2. Build it with **CodeBuild**
3. Deploy it with **CodeDeploy**
4. Possibly deploy into an **Elastic Beanstalk** environment

> Note: This is just **one example** flow — CodePipeline is flexible and supports many different combinations of stages and tools.

### 11.4 Key Benefits

- **Fully managed** service.
- **Broad compatibility**: integrates with CodeCommit, CodeBuild, CodeDeploy, Elastic Beanstalk, CloudFormation, **GitHub**, and other **third-party services and custom plugins**.
- Enables **fast delivery and rapid updates** by automating the entire release process.
- Sits at the **core of CI/CD on AWS**.

### 11.5 Exam Tip
- Any time you see the words **"orchestration of a pipeline"** or a description of an automated, multi-stage release process in an exam question → **think AWS CodePipeline**.

---

## 12. Summary Comparison Table — Deployment & Developer Services

| Service | Category | Purpose | AWS-Only or Hybrid? |
|---|---|---|---|
| **CloudFormation** | Deployment | Infrastructure as Code — declarative provisioning of nearly any AWS resource; repeatable across Regions/accounts | AWS-only |
| **CDK** | Deployment | Define infrastructure using real programming languages; compiles to CloudFormation | AWS-only |
| **Elastic Beanstalk** | Deployment | PaaS — deploy code with a known, managed architecture (LB + ASG + EC2 + RDS); developer-centric | AWS-only |
| **CodeDeploy** | Deployment | Automate application version upgrades (V1 → V2) on existing servers | **Hybrid** (EC2 + On-Premises) |
| **Systems Manager (SSM)** | Deployment/Ops | Patch, configure, and run commands at scale across a fleet of servers | **Hybrid** (EC2 + On-Premises) |
| **CodeCommit** *(discontinued)* | Developer | Private, managed Git repository for source code | AWS-only |
| **CodeBuild** | Developer | Build/compile/test code in a serverless environment | AWS-only |
| **CodeDeploy** | Developer | Deploy built code/artifacts onto servers | Hybrid |
| **CodePipeline** | Developer | Orchestrate the full CI/CD pipeline: source → build → test → deploy | AWS-only |
| **CodeArtifact** | Developer | Store and retrieve software package dependencies | AWS-only |

*(Note: CodeDeploy is intentionally listed in both categories since it bridges deployment and developer/CI-CD tooling.)*

---

## 13. Consolidated Exam Tips & Key Takeaways

- **CloudFormation** = the foundational **Infrastructure as Code** engine on AWS. Declarative, YAML/JSON, supports nearly all AWS resources, uses **Change Sets** to preview updates, tags resources automatically, and enables repeatable deployments across accounts/Regions.
- **CDK** = write infrastructure in a **real programming language**; it **compiles down into CloudFormation templates** — it does not replace CloudFormation, it generates it.
- **Elastic Beanstalk** = **PaaS**; developer only worries about **application code**; Beanstalk handles provisioning, scaling, load balancing, and health monitoring; **uses CloudFormation internally**; free to use (pay only for underlying resources).
- **CodeDeploy** = automates deploying/upgrading application **versions** onto **already-provisioned** EC2 instances *or* on-premises servers (a **hybrid** service); requires the **CodeDeploy Agent** pre-installed.
- **Systems Manager (SSM)** = **hybrid** service for managing fleets of EC2 + on-premises servers: patching, running commands, and centralized configuration via **Parameter Store**. Requires the **SSM Agent**.
- **SSM Session Manager** = secure, keyless, portless (no port 22) shell access to instances — improves security posture significantly compared to traditional SSH.
- **SSM Parameter Store** = secure, serverless, IAM-controlled storage for configuration data and secrets, with optional **KMS encryption** (SecureString) and version tracking.
- **CodeCommit** = discontinued as of **July 25, 2024** for new customers; conceptually equivalent to a private GitHub repository; may still appear on exam questions as a legacy/background concept.
- **CodeBuild** = **serverless**, pay-as-you-go **build/test/package** step of the CI/CD pipeline.
- **CodePipeline** = the **orchestration** service tying together source → build → test → deploy stages; broadly compatible with AWS-native and third-party tools (including GitHub); the answer whenever "pipeline orchestration" is mentioned.
- **CodeArtifact** = managed storage/retrieval of **software package dependencies** (Maven, Gradle, npm, yarn, twine, pip, NuGet).

### Shared Responsibility & Cost Considerations Recap
- **CloudFormation, CodePipeline, CodeBuild, CodeArtifact**: no charge for the orchestration service itself in most cases beyond usage-based pricing (e.g., CodeBuild bills for build compute time); you still pay for the **underlying resources** created (EC2, RDS, S3, etc.).
- **Elastic Beanstalk**: the Beanstalk service itself is **free** — you pay only for the EC2 instances, load balancers, and other resources it provisions on your behalf.
- **Parameter Store**: the **Standard tier is free**; the **Advanced tier** has additional costs and capabilities.
- Across all of these services, **AWS manages the provisioning engine/orchestration**, while **you remain responsible for the configuration and content of what you deploy** — a recurring theme consistent with the AWS **Shared Responsibility Model**.