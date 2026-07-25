
## Executive Overview

**Cloud Integration** refers to the patterns and services used to connect multiple applications, microservices, or system components so that they can communicate, exchange data, and trigger actions in one another. In modern cloud architectures, very few applications operate in isolation — an e-commerce platform, for example, needs its "ordering" system to talk to its "payment" system, its "shipping" system, its "fraud detection" system, and possibly dozens of other services.

How these applications communicate has a direct impact on **scalability, resilience, availability, and cost**. Poorly integrated applications (i.e., applications that are tightly coupled together) tend to fail together: if one component slows down or crashes, it can bring down the entire system. Well-integrated, **decoupled** applications can fail independently, scale independently, and recover independently — which is one of the foundational principles of the **AWS Well-Architected Framework** (specifically the Reliability and Performance Efficiency pillars).

This topic is heavily tested on the **AWS Certified Cloud Practitioner (CLF-C02)** exam because it introduces four core AWS application integration services:

- **Amazon SQS** (Simple Queue Service) — queuing
- **Amazon SNS** (Simple Notification Service) — pub/sub notifications
- **Amazon Kinesis** (Data Streams / Data Firehose) — real-time data streaming
- **Amazon MQ** — managed message broker for legacy/open protocols

Understanding _when_ to use each of these services — and recognizing their associated exam "trigger words" — is one of the most reliable ways to answer application integration questions correctly on the exam.

---

## 1. Application Communication Patterns

When two or more applications need to exchange information, there are two fundamental patterns of communication.

### 1.1 Synchronous Communication

- Definition: One application calls another application **directly** and **waits** for a response before continuing.
- **Example**: An online store has a "Buying Service." When a customer completes a purchase, the Buying Service directly calls the "Shipping Service" via an API call (e.g., an HTTP request) to initiate shipment, and waits for a confirmation response.
- **Characteristics**:
    - Simple and intuitive to understand and implement.
    - Creates **tight coupling** between the two applications — they must both be available and responsive at the same time.
    - If the receiving application slows down, becomes overwhelmed, or fails, the calling application is directly impacted (it may time out, throw errors, or hang).

> **Key Problem**: What happens if there is a **sudden spike in traffic**? For example, imagine you normally need to encode 10 videos, but suddenly you need to encode 1,000 videos. If your video encoding service is called synchronously, it may become **overwhelmed**, causing failures, timeouts, or crashes for the entire system.

### 1.2 Asynchronous (Event-Based) Communication

- Definition: One application sends a message into an intermediary (such as a **queue**), and the message sits there until another application is ready to process it. The sender does **not** wait for the receiver.
- **Example**: The Buying Service places an "order" message into a **queue** every time a purchase is made. The Shipping Service independently reads (polls) messages from that queue whenever it is ready.
- **Characteristics**:
    - The two applications are **decoupled** — they do not talk to each other directly; they communicate through an intermediary component (a queue, topic, or stream).
    - Each application can process messages **at its own pace**, and each side can **scale independently** of the other.
    - This pattern absorbs traffic spikes gracefully: messages simply accumulate in the queue until consumers catch up, instead of overwhelming a downstream service.

### 1.3 Why Decoupling Matters

**Decoupling** is one of the most important architectural concepts for the exam. When applications are decoupled:

- Each component can be scaled, updated, or replaced without directly affecting the other components.
- A failure or slowdown in one component does not cascade into a failure of the entire system.
- Traffic bursts are smoothed out because messages queue up rather than directly hitting a system that can't handle the load.

AWS provides three primary services to implement decoupled, asynchronous, event-driven architectures:

|Service|Model|Primary Purpose|
|---|---|---|
|**Amazon SQS**|Queue (point-to-point)|Decouple producers and consumers using a message queue|
|**Amazon SNS**|Pub/Sub (publish/subscribe)|Broadcast a single message to many subscribers|
|**Amazon Kinesis**|Real-time streaming|Ingest, process, and analyze continuous streams of data at scale|

> **Exam Tip**: Whenever a question describes a scenario with a sudden increase in workload, a need to prevent one service from overwhelming another, or a general need to "decouple" architecture tiers, the answer usually involves **SQS**, **SNS**, and/or **Kinesis**.

---

## 2. Amazon SQS (Simple Queue Service)

### 2.1 What Is a Queue?

A **queue** is a temporary storage location for messages that are waiting to be processed. It works on a **producer/consumer model**:

- **Producers** send (write) messages into the queue.
    - There can be **one producer** or **multiple producers** sending into the same queue.
- **Consumers** read (poll) messages from the queue.
    - **Polling** means the consumer actively requests/checks the queue for new messages (as opposed to the queue pushing messages to the consumer).
    - There can be **one consumer** or **multiple consumers**.
    - When there are multiple consumers, they **share the work** — each consumer typically receives a different message, allowing messages to be processed in parallel.
- Once a consumer finishes processing a message (for example, finishing the encoding of a video), it must explicitly **delete** the message from the queue. Only then is it removed permanently.

This mechanism fully **decouples** producers from consumers: they never communicate directly, they don't need to be online at the same time, and they can process work at completely different speeds.

### 2.2 Core Characteristics of Amazon SQS

- **One of AWS's oldest services** — it has existed for over 10 years and was among the very first services offered as part of AWS, reflecting how foundational and battle-tested it is.
- **Fully managed / Serverless**: AWS manages all the underlying infrastructure. You never provision, patch, or manage servers — you simply create a queue and start sending/receiving messages.
- **Purpose**: Used specifically to **decouple applications**.
    
    > **Exam Tip**: If an exam question uses the word **"decouple,"** the correct answer is almost always **SQS** (unless it specifically involves streaming or broadcasting to many subscribers).
    
- **Seamless, virtually unlimited scaling**: SQS can scale from **one message per second** to **tens of thousands of messages per second** without any manual configuration or capacity planning.
- **Message retention**:
    - **Default retention period**: 4 days.
    - **Maximum retention period**: 14 days.
    - Messages must be processed (read and deleted) within this retention window, or they will be automatically purged.
- **No limit** to how many messages can be stored in a queue at any given time.
- **Low latency**: Publishing and receiving messages typically takes **less than 10 milliseconds**.
- **Horizontal scalability**: Multiple consumers can read from the same queue simultaneously, sharing/splitting the message load — enabling horizontal scaling of the consumer side of the architecture.

### 2.3 Classic SQS Architecture: Decoupling Application Tiers

A very common exam scenario illustrates SQS decoupling two "tiers" of an application:

```
[Users] --> [Application Load Balancer] --> [Web Servers in Auto Scaling Group]
                                                        |
                                                        v
                                              [Amazon SQS Queue]
                                                        |
                                                        v
                                [Video Processing Servers in a 2nd Auto Scaling Group]
```

**How it works:**

1. Users send requests through an Application Load Balancer to a fleet of **EC2 web server instances** running inside an **Auto Scaling Group**.
2. When a user requests something resource-intensive (e.g., video processing), instead of the web server processing the video directly (synchronously), it inserts a **message into an SQS queue**.
3. A **separate layer** of EC2 instances (also running in their own Auto Scaling Group) continuously polls the SQS queue and performs the actual video processing work.

**Key architectural benefits:**

- The **video processing Auto Scaling Group can scale completely independently** from the web server Auto Scaling Group. If there's a sudden burst of video processing requests, only the video-processing tier needs to scale up — the web tier is unaffected and continues serving user requests normally.
- Scaling of the video processing tier can be triggered dynamically based on a **CloudWatch metric: the number of messages visible in the SQS queue** (i.e., if the queue depth grows, add more processing instances).
- This results in:
    - **Better user experience** (web servers stay responsive, never blocked waiting on slow video processing).
    - **Better cost efficiency** (you only pay for/run video processing capacity when there is actual work to do).
    - True **decoupling** between application tiers.

### 2.4 SQS Queue Types: Standard vs. FIFO

Amazon SQS offers two types of queues:

#### Standard Queue (Default)

- Messages may be delivered in a **different order** than they were sent.
- Nearly unlimited throughput.
- This is the default queue type and is what most beginner/exam scenarios will use.

#### FIFO Queue (First In, First Out)

- **FIFO** = **F**irst **I**n, **F**irst **O**ut.
- Guarantees that the **order of messages is preserved**: if a producer sends messages in the order 1, 2, 3, 4, then consumers will receive them in exactly that same order (1, 2, 3, 4).
- Used whenever the **exact sequence of message processing matters** (e.g., financial transactions that must be applied in order).

> **Exam Tip**: You mainly need to remember the _distinction_ — Standard = no ordering guarantee (but higher throughput); FIFO = strict ordering guarantee. The deep configuration details of FIFO queues (message deduplication IDs, message group IDs, exactly-once processing) are **beyond the scope of the Cloud Practitioner exam**, but knowing that FIFO exists and preserves order is required knowledge.

### 2.5 Hands-On: Using SQS in the AWS Console

The transcript demonstrates a simple hands-on walkthrough of SQS, useful for building intuition:

1. **Navigate to the SQS console** and click "Create Queue."
2. Choose between a **Standard Queue** or **FIFO Queue** (the console lets you pick a type when creating the queue). For basic/exam purposes, a Standard queue is sufficient.
3. Give the queue a name (e.g., `demo-sqs`).
4. Advanced configuration options (encryption settings, access policies, delivery delay, visibility timeout, dead-letter queue redrive policy, etc.) can generally be **left at their defaults** — these are considered beyond Cloud Practitioner exam scope.
5. Click **Create Queue** — SQS is free to create and only costs money based on usage (API requests), so testing is essentially free at low volume.
6. Once created, you can view **queue details**: queue type, name, encryption status, and message statistics such as:
    - **Messages Available** — messages ready to be retrieved.
    - **Messages in Flight** — messages that have been received by a consumer but not yet deleted.
    - **Messages Delayed** — messages that are not yet visible to consumers due to a delay setting.
7. Use the **"Send and Receive Messages"** button to:
    - **Produce**: Type a message body (e.g., "hello world") and click **Send Message**. The "Messages Available" counter increases with each message sent.
    - **Consume**: Click **Poll for Messages** to retrieve them. This simulates what a consumer application would do programmatically.
    - Click on a retrieved message to inspect its **Message ID**, **body**, **MD5 hash/attributes**, and other metadata.
    - After "processing" a message (in a real application, this is where your custom code would run), you must manually **delete** it from the queue — after deletion, "Messages Available" returns to zero.
8. When finished, you can **delete the queue** entirely (type "delete" to confirm) — this does not incur any cost, but it's good practice to clean up unused resources.

> **Exam Tip**: The actual hands-on mechanics of the SQS console (delay queues, visibility timeout, dead-letter queues, access policies) are explicitly called out as being **beyond Cloud Practitioner exam scope**. Focus on the conceptual purpose of SQS rather than deep configuration.

---

## 3. Amazon Kinesis (Real-Time Data Streaming)

### 3.1 Overview

**Amazon Kinesis** is AWS's service family for collecting, processing, and analyzing **real-time streaming data at massive scale**. Unlike SQS (which is designed for discrete task/work messages), Kinesis is designed for continuous, high-volume streams of data such as clickstreams, IoT sensor data, or application logs.

> **Exam Tip**: Anytime you see the keyword **"real-time"**, **"streaming"**, or **"big data"** in a Cloud Practitioner exam question, think **Kinesis**.

### 3.2 Key Components

#### Amazon Kinesis Data Streams

- The core service used to **collect, process, and analyze real-time streaming data** at any scale.
- This is the primary Kinesis concept you need to know for the Cloud Practitioner exam.

#### Amazon Data Firehose (formerly "Kinesis Data Firehose")

- A service used to **load/deliver streaming data** from Kinesis Data Streams (or directly from data sources) into downstream **target destinations**, such as:
    - **Amazon S3** (data lake storage)
    - **Amazon Redshift** (data warehousing)
    - **Amazon OpenSearch Service** (search and analytics)
    - and other supported destinations.
- Essentially, it acts as the **delivery/loading mechanism** that moves streamed data from Kinesis into a place where it can be stored, queried, and further analyzed.

### 3.3 Conceptual Architecture

```
[Fast Data Sources]                    [Kinesis Data Streams]        [Amazon Data Firehose]      [Destinations]
- Website clickstreams        -->      Real-time collection    -->   Loads data into      -->   - Amazon S3
- IoT / connected devices               & processing of data           destinations              - Amazon Redshift
- Application logs & metrics                                                                      - OpenSearch, etc.
```

- **"Fast data sources"** refers to data that is generated continuously and in real time — for example:
    - User clicks on a website.
    - Telemetry from internet-connected ("IoT") devices.
    - Logs and metrics emitted by application servers.
- This constant stream of data is ingested by **Kinesis Data Streams**, where it can be processed and analyzed in real time.
- Optionally, **Amazon Data Firehose** can then be used to automatically deliver that streaming data into long-term storage or analytics destinations (S3, Redshift, OpenSearch) without you having to write custom delivery code.

> **Exam Tip**: For the Cloud Practitioner exam, it is enough to know that **Kinesis = real-time data streaming**, and that it also offers **data persistence** and the ability to run **analytics in real time** on the incoming data. Deep technical details (shards, partition keys, consumer types, enhanced fan-out) are not required at this exam level.

---

## 4. Amazon SNS (Simple Notification Service)

### 4.1 The Problem SNS Solves

Imagine a "Buying Service" that, upon a completed purchase, needs to notify **multiple different services**:

- Send an email notification to the customer.
- Notify a fraud detection service.
- Notify the shipping service.
- Insert a message into an SQS queue for further processing.

**Without SNS**, the Buying Service would need to build and maintain **four separate direct/point-to-point integrations** — one for each downstream service. This is complex, hard to maintain, and creates tight coupling between the Buying Service and every consumer of its data.

### 4.2 The Pub/Sub Solution

**Amazon SNS (Simple Notification Service)** solves this with the **Publish/Subscribe (Pub/Sub)** pattern:

- The Buying Service (the **event publisher**) sends **one single message** to an **SNS Topic**.
- The SNS Topic automatically **fans out** — it forwards that single message to **every subscriber** currently subscribed to the topic (e.g., simultaneously notifying the fraud service, the shipping service, sending an email, and delivering the message into an SQS queue).

```
                          --> Email (fraud service notified)
                         /
[Buying Service] --> [SNS Topic] --> Shipping Service (via HTTP/HTTPS)
                         \
                          --> Amazon SQS Queue
```

### 4.3 Core Characteristics of Amazon SNS

- **Event publishers** only ever need to send their message to **one place**: the SNS topic. They do not need to know who or how many subscribers exist.
- **Event subscribers**: You can have as many subscribers as you want listening to a single topic.
- **Every subscriber receives every message** sent to the topic — this is the critical distinction from SQS.

> **Exam Tip / Key Distinction**:
> 
> - **SQS**: Multiple consumers **share/split** the messages amongst themselves (each message is processed by only one consumer).
> - **SNS**: Multiple subscribers **each receive a full copy** of every message published to the topic (broadcast model).

- **Scale**:
    - Each SNS topic supports **more than 12 million subscriptions per topic**.
    - There is a **soft limit of 100,000 topics per AWS account** (a "soft limit" means it can be increased via an AWS support request).

### 4.4 SNS Supported Subscriber/Destination Types

SNS can deliver (publish) messages to a wide variety of destinations, including:

- **Amazon SQS** (queues)
- **AWS Lambda** (serverless functions)
- **Amazon Data Firehose** (for streaming into storage/analytics destinations)
- **Email** (direct email notifications)
- **SMS** (text messages)
- **Mobile push notifications** (mobile app notifications)
- **HTTP / HTTPS endpoints** (generic webhook-style delivery)

> **Exam Tip**: Anytime an exam question mentions the keywords **"notification," "publish/subscribe," "pub/sub," "subscribers,"** or **"fan-out,"** think **Amazon SNS**.

### 4.5 Hands-On: Using SNS in the AWS Console

1. Navigate to the **Simple Notification Service (SNS) console** and click **Create Topic** (e.g., name it `demo-sns`). Default settings can be left as-is.
2. Once the topic is created, you'll see it currently has **zero subscriptions**.
3. Click **Create Subscription** and choose a **protocol**. Supported protocols shown in the console include:
    - HTTP
    - HTTPS
    - Email
    - Email-JSON
    - Amazon SQS
    - AWS Lambda
4. For a simple demo, choose the **Email** protocol and provide an email address as the **endpoint** (the transcript uses a temporary/disposable email address from **Mailinator**, a free temporary-inbox service, purely for testing purposes).
5. After creating the subscription, its status shows as **"Pending Confirmation"** — SNS requires subscribers to explicitly confirm they want to receive messages (this prevents spam/abuse of arbitrary email addresses or endpoints).
6. Check the target inbox for a confirmation email from AWS, and click the confirmation link. The subscription then becomes **Confirmed**.
7. Publish a test message to the topic:
    - Provide a **Subject** (e.g., "demo subject line").
    - Provide a **Message/payload** (e.g., "hello world").
    - Click **Publish Message**.
8. All confirmed subscribers immediately receive the message — in this case, the email arrives in the Mailinator inbox with the specified subject and body.
9. When finished testing, you can **delete the topic** at no cost.

> **Note**: In real-world production scenarios, you'd typically have multiple, more sophisticated subscription types (SQS, Lambda, HTTPS webhooks) rather than just email, but email is the simplest to demonstrate.

### 4.6 SNS Message Durability

- **SNS does NOT retain/store messages** — it is **not a durable message store**. If a message is published and there are no active/confirmed subscribers, or a subscriber's endpoint is down, that message may effectively be lost for that subscriber (this is a key conceptual difference from SQS, where messages persist in the queue for up to 14 days until explicitly processed and deleted).
- This reinforces why SNS is often paired with SQS: SNS can "fan out" a message to multiple SQS queues, and each of those queues will then durably retain the message until each downstream consumer processes it (a common pattern known as **"fan-out to SQS"**).

---

## 5. Amazon MQ

### 5.1 Why Amazon MQ Exists

Amazon SQS and Amazon SNS are **cloud-native** AWS services — meaning they use **proprietary AWS APIs and protocols**. They were built specifically for AWS and are not based on industry-standard open messaging protocols.

However, many **traditional, on-premises applications** are built using **open, standardized messaging protocols**, such as:

- **MQTT** (Message Queuing Telemetry Transport — commonly used in IoT)
- **AMQP** (Advanced Message Queuing Protocol)
- **STOMP** (Simple/Streaming Text Oriented Messaging Protocol)
- **OpenWire**
- **WSS** (WebSocket Secure)

When a company wants to **migrate such an application to AWS**, rewriting/re-engineering the application to use SQS's or SNS's proprietary APIs instead of these open protocols can be costly, time-consuming, and risky.

### 5.2 What Amazon MQ Provides

**Amazon MQ** is a **managed message broker service** that supports two well-known open-source broker technologies:

- **RabbitMQ**
- **ActiveMQ**

These are the same broker technologies many companies already run **on-premises** to support open protocols like MQTT, AMQP, and STOMP. Amazon MQ essentially provides a **fully managed version of these brokers in the AWS cloud**, so companies can "lift and shift" their existing messaging infrastructure to AWS **without having to rewrite their application code or change protocols**.

### 5.3 Key Characteristics & Trade-offs

- **Does not scale as much as SQS or SNS.** SQS and SNS are designed for near-infinite, serverless scaling. Amazon MQ, by contrast, **runs on actual underlying servers** (brokers), so its scaling is bounded by the size/capacity of those servers.
- **Because it runs on servers, it can experience server-related issues** (e.g., a broker instance failing).
    - To achieve high availability, you can deploy Amazon MQ in a **Multi-AZ (Availability Zone) configuration with automatic failover**, similar in concept to how you'd achieve HA with a traditional relational database.
- **Combines both queue and topic models in a single broker**:
    - It supports **queue-style** messaging (similar in concept to SQS).
    - It also supports **topic-style pub/sub** messaging (similar in concept to SNS).
    - Both capabilities are provided by the **same underlying broker**, unlike AWS where SQS and SNS are two entirely separate services.

### 5.4 When to Use Amazon MQ vs. SQS/SNS

> **Exam Rule of Thumb**:
> 
> - Use **Amazon MQ** **only** when a company is **migrating an existing on-premises application** to AWS, and that application **already depends on** an open protocol (MQTT, AMQP, STOMP, OpenWire, WSS) — and rewriting the application isn't feasible or desired.
> - In **all other cases**, prefer **SQS and SNS**, because they:
>     - Scale far better (virtually unlimited vs. broker/server-bound).
>     - Are more deeply and natively integrated with the rest of the AWS ecosystem (Lambda, EC2 Auto Scaling, CloudWatch, etc.).
>     - Are fully serverless, requiring no server/broker management at all.

---

## 6. Summary & Comparison Table

|Feature|**Amazon SQS**|**Amazon SNS**|**Amazon Kinesis**|**Amazon MQ**|
|---|---|---|---|---|
|**Category**|Queuing service|Pub/Sub notification service|Real-time data streaming|Managed message broker|
|**Communication Model**|Point-to-point (queue)|Publish/Subscribe (broadcast/fan-out)|Continuous stream ingestion & analysis|Queue + Topic (broker-based)|
|**Message Consumption**|Consumers **share/split** messages|**Every** subscriber gets **every** message|Multiple consumers can read/process the stream|Depends on queue or topic mode used|
|**Message Retention/Durability**|Yes — 4 days default, up to 14 days max|**No** — not a durable store|Yes — supports data persistence|Depends on broker configuration|
|**Scaling**|Virtually unlimited, serverless|Virtually unlimited, serverless|Scales to handle massive real-time throughput|Limited by underlying server/broker capacity|
|**Underlying Infrastructure**|Fully managed/serverless|Fully managed/serverless|Fully managed|Runs on managed servers (can be Multi-AZ)|
|**Primary Use Case**|Decoupling application components/tiers|Fan-out notifications to many subscribers (email, SMS, SQS, Lambda, HTTP/S, mobile push)|Real-time analytics on streaming/big data (clickstreams, IoT, logs)|Migrating on-premises apps using open protocols (MQTT, AMQP, STOMP, OpenWire, WSS) to the cloud|
|**Exam Trigger Words**|"decouple," "queue"|"notification," "pub/sub," "subscribers," "fan-out"|"real-time," "streaming," "big data"|"MQTT/AMQP/STOMP," "RabbitMQ/ActiveMQ," "migrate legacy messaging"|

---

## 7. Use Cases & Business Benefits

### When to use SQS

- Buffering work between application tiers so that a traffic spike doesn't overwhelm downstream services (e.g., web tier vs. video-processing tier).
- Decoupling microservices so each can be deployed, scaled, and maintained independently.
- Building resilient systems where a temporary outage in a downstream consumer doesn't result in data/message loss (since messages persist in the queue for up to 14 days).

### When to use SNS

- Broadcasting a single event to multiple different systems simultaneously (e.g., order confirmation triggering an email, a fraud check, and a shipping workflow at the same time).
- Sending direct notifications to end users via email, SMS, or mobile push.
- Implementing "fan-out" architectures, commonly combined with SQS (SNS fans out to multiple SQS queues, each durably storing messages for a different downstream consumer).

### When to use Kinesis

- Ingesting and analyzing high-volume, continuously generated data such as website clickstream analytics, application/server log aggregation, or IoT sensor telemetry.
- Building real-time dashboards or real-time analytics pipelines.
- Automatically delivering streaming data into a data lake (S3) or data warehouse (Redshift) via Data Firehose for later batch analysis.

### When to use Amazon MQ

- Migrating a legacy, on-premises messaging system (built on RabbitMQ or ActiveMQ) to AWS with minimal code changes.
- Preserving existing investment in applications built against open standard protocols like MQTT, AMQP, or STOMP.
- Situations where extreme serverless-style scaling is not the primary requirement, but protocol compatibility is.

---

## 8. Key Exam Tips & Takeaways

- **"Decouple" → SQS.** This is one of the most reliable keyword associations on the exam.
- **"Notification / Pub-Sub / Subscribers / Fan-out" → SNS.**
- **"Real-time / Streaming / Big Data" → Kinesis.**
- **"Legacy protocol / MQTT / AMQP / RabbitMQ / ActiveMQ / Migration" → Amazon MQ.**
- **SQS consumers share the workload** (each message is processed once, by one consumer); **SNS subscribers each get a full copy** of every message (broadcast model). This distinction is one of the most commonly tested contrasts between the two services.
- **SQS is durable** (messages persist up to 14 days by default/max); **SNS is NOT durable** (no message retention — if there's no subscriber to receive it, the message is effectively lost).
- **SQS and SNS are both fully serverless** — no infrastructure to manage, and both scale automatically and virtually without limit.
- **Amazon MQ runs on servers**, so it does **not** scale as elastically as SQS/SNS, but it supports open, industry-standard protocols (MQTT, AMQP, STOMP, OpenWire, WSS) that SQS/SNS do not natively support.
- **Amazon MQ can be made highly available** using a **Multi-AZ deployment with automatic failover** — an important consideration since it is server-based rather than serverless.
- **Cost consideration**: Both SQS and SNS follow a pay-per-use (pay-for-what-you-use) pricing model with no upfront cost and no charge for idle/unused queues or topics — creating and deleting them for testing purposes, as demonstrated in the hands-on labs, does not incur charges beyond minimal API request usage.
- **Shared Responsibility Model context**: For all these services (SQS, SNS, Kinesis, Amazon MQ), AWS is responsible for managing, patching, and securing the underlying infrastructure (the "security **of** the cloud"), while the customer is responsible for configuring appropriate access policies, encryption settings, and correct application-level usage (the "security **in** the cloud").