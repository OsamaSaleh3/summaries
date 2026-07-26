
### What is Docker?

Docker is a software platform used to package applications into standardized units called **containers**. A container bundles the application code along with all its dependencies, ensuring it runs identically on any operating system or environment without compatibility issues. Unlike traditional Virtual Machines (EC2), containers do not include a full guest operating system; instead, they share the host system's resources via a Docker Daemon, making them lightweight and fast to scale.

### Core Concepts Explained

- **Amazon ECS (Elastic Container Service):** AWS's native container management service used to launch and orchestrate Docker containers. When using standard ECS, **you must provision and manage the underlying EC2 instances** where your containers run.
    
- **AWS Fargate:** A **serverless** compute engine for containers that works with ECS. With Fargate, you do not manage or see any EC2 instances; AWS automatically provisions and scales the underlying infrastructure based on the CPU and RAM you specify.
    
- **Amazon ECR (Elastic Container Registry):** A secure, private Docker image repository on AWS. Think of it as a storage locker where you upload your packaged container images so they can be easily retrieved and run by ECS or Fargate.
    

### ECS Launch Types Compared

|**Feature**|**ECS (EC2 Launch Type)**|**AWS Fargate (Serverless Launch Type)**|
|---|---|---|
|**Server Management**|You manage and scale EC2 instances.|Serverless; AWS handles everything.|
|**Configuration**|Higher control over instance types.|Simplified setup via CPU/RAM requirements.|
|**Ideal For**|Custom OS/networking needs.|Quick deployment without server overhead.|

### The Big Picture

Your developer packages their application code into a Docker image and pushes it into **Amazon ECR**. When it is time to run the application, **Amazon ECS** pulls that image from ECR. Depending on your setup, ECS will either place that container onto an **EC2 instance** you managed yourself, or hand it to **AWS Fargate** to run seamlessly without managing infrastructure.

### Exam Focus

- **Keywords:** Look for "Docker containers on AWS" to trigger ECS or Fargate. Look for "run containers without managing servers/EC2" to trigger Fargate.
    
- **ECR Trigger:** Any mention of a "private repository" or "storing container images" on AWS maps to ECR.
    

## Elastic Kubernetes Service (EKS)

### What is Kubernetes?

Kubernetes is an open-source system used to automate the deployment, scaling, and management of containerized applications across clusters of hosts. It acts as an industry-standard alternative to AWS's native ECS, designed to run containers across multiple clouds or on-premises environments.

### Core Concepts Explained

- **Amazon EKS:** A fully managed service that makes it easy to run Kubernetes on AWS without needing to install or operate your own Kubernetes control plane. It is **cloud-agnostic**, meaning applications built for EKS can easily migrate to other clouds (like Azure or Google Cloud) or on-premises environments running Kubernetes. Under the hood, EKS can host its workloads using either managed EC2 instances or AWS Fargate for a serverless experience.
    

### Exam Focus

- **Keywords:** If the exam scenario specifically uses the phrase **"Kubernetes"** or mentions a **"cloud-agnostic"** container strategy, the answer is always Amazon EKS.
    

## AWS Lambda & Amazon API Gateway

### What is a Serverless Function?

In traditional computing, a server runs 24/7 waiting for work, meaning you pay for idle time. Serverless functions change this paradigm: you write a specific piece of code (a function), and AWS only runs it when an external "event" triggers it. When the work is done, the function turns off, and you stop paying.

### Core Concepts Explained

- **AWS Lambda:** A serverless, event-driven compute service that executes your code in response to triggers. It scales automatically from zero to thousands of concurrent requests.
    
- **Execution Time Limit:** Lambda functions are built for short, quick tasks and have a **maximum execution timeout of 15 minutes**.
    
- **Amazon API Gateway:** A fully managed service that allows developers to create, secure, and maintain HTTP or REST APIs. It acts as the "front door" for external clients (like mobile apps or websites) to securely interact with backend services like Lambda.
    

### Common Serverless Architecture Patterns

- **Event-Driven Processing:** A user uploads a photo to an Amazon S3 bucket. The upload event triggers a Lambda function, which instantly resizes the image into a thumbnail and updates a DynamoDB database table.
    
- **Serverless CRON Job:** Using a scheduler (like Amazon EventBridge/CloudWatch Events) to trigger a Lambda function on a set schedule (e.g., "every hour") to perform automated cleanup or scripts without keeping an EC2 instance running.
    

### The Big Picture

An external mobile app client sends an HTTP web request. **Amazon API Gateway** intercepts this request, validates security, and routes it to **AWS Lambda**. Lambda executes the specific backend logic code and securely interacts with a serverless database like **Amazon DynamoDB** to read or update data, passing the response back through the gateway.

### Exam Focus

- **Keywords:** Look for "event-driven," "reactive processing," or "run code without provisioning servers".
    
- **Cost Traps:** Lambda pricing is strictly based on **the number of requests (calls)** and **compute duration** (the RAM allocated multiplied by how long the function runs in seconds). It does not charge when idle.
    
- **Scenario Trigger:** If asked how to expose a serverless backend to the internet via an HTTP API endpoint, the answer is the combination of **API Gateway + Lambda**.
    

## AWS Batch

### What is Batch Processing?

Batch processing is the execution of a series of automated data jobs that have a clear start and end point (unlike a continuous streaming job). These workloads are usually high-volume, compute-heavy tasks—like processing a queue of millions of financial transactions at 2:00 AM.

### Core Concepts Explained

- **AWS Batch:** A managed service that handles batch computing jobs at any scale. You define your job as a Docker image, and AWS Batch automatically provisions the optimal quantity of EC2 or Spot Instances to handle the work queue, shutting them down immediately when the jobs finish. It is **not serverless** because you can see the underlying EC2 instances, but AWS manages their scaling and lifecycle entirely.
    

### AWS Lambda vs. AWS Batch

|**Feature**|**AWS Lambda**|**AWS Batch**|
|---|---|---|
|**Time Limit**|Max 15 minutes.|No time limit.|
|**Infrastructure**|Serverless.|Managed EC2 / Spot Instances.|
|**Runtime Pack**|Code scripts or specific container APIs.|Any runtime via standard Docker image.|
|**Storage**|Very limited temporary disk space.|Scalable via attached EBS volumes.|

### Exam Focus

- **Scenario Trigger:** If a scenario involves heavy, long-running processing jobs (exceeding 15 minutes) packaged as Docker containers, look for **AWS Batch** over Lambda.
    

## Amazon Lightsail

### What is Lightsail?

AWS can be complex, requiring you to manually configure networking, storage, security groups, and instances separately. Amazon Lightsail simplifies this by packaging compute, storage, databases, and networking into an all-in-one, easy-to-use bundle.

### Core Concepts Explained

- **Amazon Lightsail:** Designed as a lightweight, entry-level alternative for users with little to no cloud infrastructure experience who need to deploy simple applications quickly. It operates outside the standard complex AWS ecosystem, featuring a highly simplified management console and **low, predictable monthly flat-rate pricing**.
    
- **Limitations:** Lightsail offers no auto-scaling capabilities and has very limited built-in integrations with advanced AWS infrastructure services. It includes pre-configured click-to-launch blueprints for popular stacks like WordPress, Magento, and LAMP.
    

### Exam Focus

- **Keywords:** Look for "no cloud experience," "predictable low pricing," or "quickly launch a simple WordPress site".
    
- **Distractor Warning:** In most CLF-C02 questions, Lightsail is a distractor answer. Only choose it if the scenario explicitly emphasizes simplicity, an unconfigured flat-rate cost, or an extreme lack of cloud expertise.
    
- **Lambda Free Tier:** 1 million free invocations and 400,000 GB-seconds of compute time every month.

- **API Gateway WebSocket support:** In addition to REST/HTTP APIs, API Gateway also supports **WebSocket APIs** for real-time, two-way communication.

## Quick Reference Table

|**Concept**|**What it is**|**Key thing to remember**|
|---|---|---|
|**Docker Container**|Standardized app package.|Identical behavior across all operating systems.|
|**Amazon ECS**|AWS Container Orchestrator.|Requires you to manage the EC2 instances.|
|**AWS Fargate**|Serverless Container Compute.|Runs containers without managing EC2 servers.|
|**Amazon ECR**|Private Docker Registry.|The place to store container images.|
|**Amazon EKS**|Managed Kubernetes.|Use for cloud-agnostic container architectures.|
|**AWS Lambda**|Serverless Functions.|15-minute limit; priced by requests and duration.|
|**API Gateway**|Managed API Front Door.|Pairs with Lambda for serverless web APIs.|
|**AWS Batch**|Heavy Batch Job Controller.|No time limit; automatically scales EC2/Spot.|
|**Amazon Lightsail**|Simple All-In-One Bundle.|Predictable monthly price for simple workloads.|