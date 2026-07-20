
## Executive Overview

Databases are the backbone of virtually every application running on AWS. While storage services like **EBS**, **EFS**, **EC2 Instance Store**, and **Amazon S3** let you store data as _files_ or _blocks_, a database lets you store data in a **structured** way — with indexes, defined relationships, and efficient query capabilities. AWS offers a huge portfolio of **fully managed database services**, each purpose-built for a specific data model and workload pattern (transactional, analytical, key-value, graph, time-series, ledger, in-memory, etc.).

For the Cloud Practitioner exam, you are **not** expected to know deep configuration details. Instead, you need to:

1. Recognize the **name and purpose** of each AWS database/analytics service.
2. Match a **use case or keyword** in an exam question to the correct service.
3. Understand the **general benefits of "managed" services** versus self-hosting a database on EC2.
4. Understand basic **shared responsibility** implications for databases.

This topic matters because database and analytics services are one of the largest and most frequently tested domains in the AWS ecosystem — almost every real-world architecture involves storing, querying, migrating, or analyzing data.

---

## 1. Database Fundamentals

### 1.1 What Is a Database?

A database is a structured way to store data that:

- Allows you to build **indexes** for fast lookups.
- Lets you efficiently **query or search** through data.
- Can define **relationships** between different sets of data.

This differs from raw file/block storage (EBS, EFS, EC2 Instance Store, S3), which only supports storing and retrieving whole files or blocks — there is no native concept of relationships, indexing, or structured querying built into the storage layer itself.

> **Exam takeaway:** If a question implies structured data, relationships, indexing, or querying — think **database**, not raw storage.

### 1.2 Two Major Database Families

#### A. Relational Databases (SQL)

- Conceptually similar to a set of **linked Excel spreadsheets**.
- Data is organized into **tables** (like spreadsheet tabs), each with **rows** and **columns**.
- Tables are connected via **relations** — e.g., a `students` table has a `department_id` column that links to the `department_id` column in a `departments` table. You can chain further relations (e.g., a `subjects` table linked to `students`).
- Because tables are linked/related to one another, this model is called a **relational database**.
- Queries and lookups are performed using **SQL (Structured Query Language)**.

> **Exam keyword rule:** Whenever you see **"SQL"** in a question → think **relational database**.

**Deeper context (beyond transcript):** Relational databases enforce a fixed **schema** (table structure must be defined ahead of time) and typically guarantee **ACID** properties (Atomicity, Consistency, Isolation, Durability), making them ideal for transactional systems like banking, order processing, and inventory management.

#### B. NoSQL Databases (Non-Relational)

- **NoSQL = "Non-SQL" = non-relational.**
- A more modern class of databases, purpose-built for a **specific data model**, with a **flexible schema** suited to modern applications.
- **Schema** = the "shape" of the data (which fields exist, their types, and structure).

**Benefits of NoSQL:**

|Benefit|Explanation|
|---|---|
|**Flexibility**|Easy to evolve the data model over time; no rigid table structure.|
|**Scalability**|Designed to scale **horizontally** (scale-out) by adding more distributed servers — unlike relational databases, which are harder to scale out and typically rely on **vertical scaling** (bigger instance).|
|**High performance**|Optimized for a specific data access pattern.|
|**Highly functional**|Data types are tailored/optimized for the specific model in use.|

**Categories of NoSQL databases** (covered in this AWS section):

- **Key-value** databases (e.g., DynamoDB)
- **Document** databases (e.g., DocumentDB)
- **Graph** databases (e.g., Neptune)
- **In-memory** databases (e.g., ElastiCache)
- **Search** databases

**JSON — the common NoSQL data format:**

- **JSON** = **JavaScript Object Notation** (also used earlier in the course for IAM policy documents).
- Unlike a flat spreadsheet, JSON supports:
    - **Nested objects** (e.g., an `address` field nested inside a larger user object).
    - **Fields that can change over time** — you can freely add new fields to a document without altering every other record.
    - **Arrays** — e.g., a `cars` field containing a list like `["Ford", "BMW", "Fiat"]`.

> **Exam takeaway:** JSON's flexible, nested, and evolvable structure is exactly why it's the natural fit for NoSQL databases, in contrast to the rigid tabular structure of relational databases.

### 1.3 Managed Databases & the Shared Responsibility Model

AWS offers many **managed database services**. Using a managed database (instead of self-hosting a database engine on an EC2 instance) provides major benefits:

- **Quick provisioning** — a database can be up and running in minutes.
- **High availability** built in by design.
- **Easy scaling** — both **vertical** (bigger instance) and **horizontal** (more instances/replicas), depending on the service.
- **Automated backup and restore** utilities.
- **Automated operations and upgrades.**
- **OS-level patching is AWS's responsibility**, not yours.
- **Integrated monitoring and alerting** (e.g., via Amazon CloudWatch).

**Contrast — Self-managed database on EC2:** If you install and run a database engine (e.g., MySQL) yourself on an EC2 instance, **you** are fully responsible for:

- Resiliency and fault tolerance
- Backups
- OS and database patching
- High availability configuration
- Scaling

> **Exam Tip (Shared Responsibility Model):** For AWS-_managed_ database services (RDS, DynamoDB, Aurora, Redshift, etc.), AWS manages the underlying infrastructure, OS patching, and database engine patching (customer chooses when for RDS engine patching, but AWS handles OS patching). For a self-managed database on **EC2**, the customer is responsible for everything above the hypervisor — OS, patching, backups, scaling, and high availability, per the standard **shared responsibility model** ("security **in** the cloud" vs. "security **of** the cloud").

---

## 2. Relational Databases: RDS & Aurora

### 2.1 Amazon RDS (Relational Database Service)

**RDS** = a **managed** service for relational databases that use **SQL** as the query language.

**Supported database engines:**

- PostgreSQL
- MySQL
- MariaDB
- Oracle
- Microsoft SQL Server
- IBM DB2
- **Amazon Aurora** (AWS's own proprietary relational engine — covered separately below)

**Why use RDS instead of self-hosting a database on EC2?**

- **Automatic provisioning.**
- **OS patching handled by AWS.**
- **Continuous backups** with **Point-in-Time Restore** (restore your database to any specific second within your backup retention window).
- **Monitoring dashboards** (e.g., CPU utilization, connections, storage).
- **Read scaling** via **Read Replicas**.
- **Multi-AZ** setup for disaster recovery / high availability.
- **Maintenance windows** for controlled upgrades.
- Supports both **vertical** and **horizontal** scaling.
- Storage is backed by **Amazon EBS**.

**Key limitation:** You **cannot SSH** into the underlying instance of an RDS database. AWS manages the entire database infrastructure for you — you only interact with it through the database engine's native protocols/tools.

**Typical 3-tier architecture with RDS:**

```
[Users] → [Load Balancer] → [EC2 instances in an Auto Scaling Group] → [RDS Database]
```

The load balancer distributes web requests to backend EC2 instances (application logic tier), which then read and write structured data to/from the RDS database (data tier).

#### RDS Deployment Options (High-Value Exam Topic)

**1. Read Replicas — Scale Reads**

- Used to **offload read traffic** from the main ("primary") database.
- You can create **up to 15 Read Replicas**.
- Applications can **read** from any replica, spreading read load across multiple copies.
- **Writes still only go to the main/primary database** — replicas are read-only.
- Replicas can be within the **same region** (or, as described below, across regions).

**2. Multi-AZ — High Availability / Failover**

- Creates a **synchronous, standby replica in a different Availability Zone**.
- Purpose: **failover** in case the primary AZ or database instance has an outage.
- The application reads and writes **only to the main database**; the standby (failover) database is **passive** and not directly accessible under normal operation.
- If the primary database fails (instance issue or AZ-level outage), RDS automatically **triggers a failover**, and the application is redirected to the standby database in the other AZ.
- You can only have **one** failover AZ configured for Multi-AZ.

**3. Multi-Region (Cross-Region Read Replicas)**

- Read replicas placed in an **entirely different AWS Region**.
- Applications in that remote region can **read locally** with **lower latency**.
- **Writes must still go back across regions** to the primary database's region.
- **Use cases:**
    - **Disaster Recovery (DR):** if the primary region has a regional-level outage, you have a standby copy elsewhere.
    - **Performance:** local reads reduce latency for geographically distributed users.
- **Trade-off:** Cross-region replication incurs **data transfer (network) costs**.

> **Exam Tip:** Know the difference clearly:
> 
> - **Read Replica** → scale **reads**, same or different region, asynchronous replication, read-only copies.
> - **Multi-AZ** → **high availability/failover**, synchronous replication, standby is passive, same region but different AZ.
> - **Multi-Region** → **disaster recovery + reduced latency for global users**, incurs replication network costs.

### 2.2 Amazon Aurora

**Aurora** is AWS's own **proprietary**, cloud-native relational database engine (not open source). It works the same way as RDS from an architecture standpoint — applications connect to it directly — but it is engineered by AWS for the cloud.

**Key facts:**

- Supports **PostgreSQL** and **MySQL** compatible engines.
- **Performance:** ~**5x performance improvement** over standard MySQL on RDS, and ~**3x performance improvement** over standard PostgreSQL on RDS.
- **Storage auto-scales** automatically in **increments of 10 GB**, up to **128 TB** — no manual storage provisioning needed.
- **Cost:** Aurora is typically about **20% more expensive** than RDS, but its performance/efficiency gains often make it **more cost-effective overall**.
- **Not included in AWS Free Tier** (unlike RDS, which does have free tier options).

#### Aurora Serverless

- A deployment option where **database instantiation is automated** and the database **auto-scales based on actual usage/demand**.
- Supports both **PostgreSQL and MySQL** compatible engines.
- **No capacity planning required**, and **no server management** — you don't manage any underlying instances.
- **Billing is per-second**, based on actual database capacity consumed, which can be far more cost-effective than provisioning a fixed Aurora cluster.
- **Ideal use case:** **infrequent, intermittent, or unpredictable workloads** — you don't want to pay for idle capacity.
- **How it works:** Clients connect to a **proxy fleet** managed by Aurora. Behind the scenes, Aurora automatically instantiates database compute capacity as needed to scale up or down, while **all instances share the same underlying storage volume**.

> **Exam Tip:** Keywords like **"no management overhead," "unpredictable workloads,"** or **"pay per second"** for an Aurora-based scenario → think **Aurora Serverless**.

---

## 3. In-Memory & NoSQL Databases

### 3.1 Amazon ElastiCache

**ElastiCache** is AWS's managed **in-memory database/caching** service, supporting two engines: **Redis** and **Memcached**.

- Provides **high performance, low-latency** access to data by keeping it in memory (RAM) rather than on disk.
- **Primary use case:** **Reduce load on a backend database** (like RDS) that experiences **read-intensive** workloads. Instead of hitting the database repeatedly with the same queries, results are cached in ElastiCache for fast, repeated retrieval.
- As a **fully managed service**, AWS handles OS maintenance, patching, optimization, setup, configuration, monitoring, failure recovery, and backups.

**Typical architecture:**

```
[Users] → [Load Balancer] → [EC2 (ASG)] → reads/writes → [RDS (slow, persistent)]
                                          → cached reads → [ElastiCache (fast, in-memory)]
```

The application checks the cache first; if data is present ("cache hit"), it's served quickly from memory, reducing pressure on the primary database.

> **Exam Tip:** Any mention of **"in-memory database"** or **"in-memory cache"** → think **ElastiCache**.

### 3.2 Amazon DynamoDB

**DynamoDB** is AWS's flagship **fully managed, serverless, key-value NoSQL database**.

**Key characteristics:**

- **Fully managed and highly available**, with data **replicated across 3 Availability Zones** automatically.
- **NoSQL** — not a relational database; **no SQL joins between tables**. All the data your application needs must generally live within a single, well-designed table (denormalized data model).
- **Serverless** — you don't provision any server instances (though servers exist behind the scenes, they are invisible to you).
- **Massive scalability:** millions of requests per second, trillions of rows, and hundreds of terabytes of storage.
- **Performance:** consistent **single-digit millisecond latency**.
- **Security:** integrated with **IAM** for authentication, authorization, and access administration.
- **Cost:** low cost with **auto-scaling** capability.
- **Table classes:** **Standard** and **Standard-Infrequent Access (IA)** — choose based on data access patterns for cost optimization.

**Data model:**

- **Primary key** made of:
    - A **partition key** (required)
    - An optional **sort key** (composite primary key) — _noted as out of scope for the Cloud Practitioner exam_.
- **Attributes** = the additional columns/fields you define for each item (flexible — items in the same table can have different attributes).
- Data is organized **row by row** as **items** within a **table**.

**Hands-on demo takeaways (from the transcript):**

- Creating a table in DynamoDB requires **no server provisioning** — you simply define a table name and a partition key (e.g., `user_id`), and the table is ready.
- You can insert items ("rows") with varying attributes — e.g., one item might have `first_name`, `last_name`, and a `number`, while another item in the same table might only have `first_name`. DynamoDB **automatically infers the schema per item** — this demonstrates its flexible schema.
- Because DynamoDB has **no joins**, all relevant data for a query pattern typically needs to be modeled/stored together within the same table — this is a fundamentally different design approach compared to relational databases.

> **Exam Tip:** Keywords like **"key-value database," "serverless," "single-digit millisecond latency,"** or **"low latency retrieval"** → think **DynamoDB**.

#### DynamoDB Accelerator (DAX)

- A **fully managed, in-memory cache built specifically for DynamoDB**.
- **Not the same as ElastiCache** — DAX is purpose-built and tightly integrated only with DynamoDB.
- Provides up to a **10x performance improvement**, reducing DynamoDB read latency from single-digit milliseconds down to **microseconds**.
- Fully **secure, highly scalable, and highly available**.

> **Exam Tip:** For caching **specifically in front of DynamoDB** → use **DAX**, not ElastiCache. ElastiCache is the general-purpose caching layer usable with other databases (like RDS); DAX is DynamoDB-only.

#### DynamoDB Global Tables

- A feature that makes a DynamoDB table **accessible with low latency across multiple AWS Regions**.
- Supports replication across **1 to 10 regions**.
- Example: A table created in `us-east-1` can be configured as a Global Table with a replica in `eu-west-3` (Paris). Users near Paris get low-latency access to a **local** copy of the data.
- **Active-Active replication:** Users can **read AND write to the table in any configured region**, and changes are actively **replicated bidirectionally** between all configured regions.

> **Exam Tip:** "Global," "multi-region," "low latency writes and reads worldwide" for DynamoDB → think **DynamoDB Global Tables** with **active-active replication**.

### 3.3 Amazon DocumentDB

**DocumentDB** is AWS's cloud-native, managed **document database**, positioned as **"the Aurora of MongoDB."**

- It is **compatible with MongoDB**, a popular open-source NoSQL document database used to store, query, and index **JSON**-formatted data.
- Follows a similar managed deployment model as Aurora:
    - **Fully managed**
    - **Highly available**
    - Data **replicated across 3 Availability Zones**
    - **Storage auto-scales** in increments of **10 GB**
- Engineered to scale to workloads handling **millions of requests per second**.

> **Exam Tip:** Anytime you see **"MongoDB"** → think **DocumentDB**. It's a **NoSQL / document database**, distinct from DynamoDB (key-value).

### 3.4 Amazon Neptune

**Neptune** is a **fully managed graph database**.

- A **graph database** models highly interconnected data — for example, a **social network** where users have friends, create posts, leave comments, and give likes; all these entities and their relationships form a "graph" of interconnected nodes and edges.
- **Replication across 3 Availability Zones**, with up to **15 read replicas**.
- Optimized to run **complex queries** over **highly connected datasets**.
- Can store **billions of relationships** and query them with **millisecond latency**.
- **Highly available** across multiple AZs.

**Use cases:**

- Social networking applications
- **Knowledge graphs** (e.g., Wikipedia's interconnected article database)
- Fraud detection
- Recommendation engines

> **Exam Tip:** "Graph database" or "highly connected/relationship-heavy data" → think **Neptune**.

### 3.5 Amazon Timestream

**Timestream** is a **fully managed, fast, scalable, serverless database purpose-built for time-series data**.

- **Time-series data** = data points that change/evolve **over time** (e.g., a metric plotted against a timeline).
- **Automatic scaling** up and down based on workload/compute needs.
- Can store and analyze **trillions of events per day**.
- Roughly **1,000x faster** and about **1/10th the cost** compared to using a relational database for the same time-series workload.
- Includes **built-in time-series analytics functions** to detect patterns in real time.

> **Exam Tip:** Anytime you see **"time series data"** → think **Amazon Timestream**.

### 3.6 Amazon QLDB (Quantum Ledger Database)

**QLDB** is a **fully managed ledger database** designed to record and track a verifiable history of application data changes.

- A **ledger** = a book/record of transactions — historically used for financial record-keeping.
- **Serverless, highly available**, with data replicated across **3 Availability Zones**.
- Used to review the **complete history of all changes** made to application data over time.
- **Immutable:** once data is written, it **cannot be removed or modified**.
- Uses a **cryptographic hash** for every modification, stored in an internal **journal** (a sequential, append-only log of changes) — this makes the change history **cryptographically verifiable** by anyone with access.
- Delivers **2–3x better performance** than common ledger/blockchain frameworks used for similar purposes.
- Supports **SQL** for manipulating/querying data.

**Key distinction from Amazon Managed Blockchain:**

- QLDB has a **single, central authority** — it is a **centralized** ledger owned and operated by AWS (no decentralization). This centralized-but-verifiable model aligns well with many **financial regulatory requirements**.

> **Exam Tip:** "Financial transactions," "immutable ledger," "cryptographically verifiable history" → think **QLDB** (centralized ledger, no decentralization).

### 3.7 Amazon Managed Blockchain

- **Blockchain** enables applications where **multiple parties can execute and validate transactions without needing a trusted central authority** — i.e., it has a **decentralization** component.
- **Amazon Managed Blockchain** lets you:
    - Join **public blockchain networks**, or
    - Create and manage your own **scalable private blockchain network** within AWS.
- Compatible with two blockchain frameworks:
    - **Hyperledger Fabric**
    - **Ethereum**

> **Exam Tip:** "Blockchain," "Hyperledger Fabric," or "Ethereum" → think **Amazon Managed Blockchain** (a **decentralized** ledger, in contrast to QLDB's centralized model).

---

## 4. Analytics Services

### 4.1 Amazon EMR (Elastic MapReduce)

- Despite the name resembling a database, **EMR is not a database** — it's a platform for creating a **managed Hadoop cluster** to perform **big data** processing.
- **Hadoop** is an **open-source big data framework** that allows **multiple servers to work together in a cluster** to collaboratively process/analyze massive datasets.
- With EMR, you can spin up a cluster made of **hundreds of EC2 instances** that work together on big data analysis.
- Part of the broader **Hadoop ecosystem** are related open-source projects such as:
    
    - **Apache Spark**
    - **HBase**
    - **Presto**
    - **Flink**
    
    These all run _on top of_ the Hadoop cluster that EMR provisions and manages.
- **What EMR does for you:** provisions and configures the underlying EC2 instances so that they work together correctly as a cluster.
- Supports **auto-scaling** and integrates with **Spot Instances** for cost savings.

**Use cases:** big data processing, machine learning, web indexing, general large-scale data analytics.

> **Exam Tip:** "Hadoop cluster" in a question → think **Amazon EMR**, no matter the specific use case mentioned.

### 4.2 Amazon Athena

**Athena** is a **serverless query service** used to run analytics **directly against objects stored in Amazon S3**, using **SQL**.

**Key facts:**

- No need to "load" the data into a database first — files simply need to exist in **S3**, and Athena queries them in place.
- Supports multiple file formats: **CSV, JSON, ORC, Avro, and Parquet**.
- Built on the **Presto** query engine.

**How it works:**

1. Users load data into **Amazon S3**.
2. **Athena** queries and analyzes that data directly using SQL.
3. (Optional) Results can be visualized using **Amazon QuickSight** for reporting/dashboards.

**Pricing:** approximately **$5 per terabyte of data scanned**. Using **compressed** or **columnar** file formats (like Parquet or ORC) reduces the amount of data scanned per query, which **directly lowers cost**.

**Use cases:**

- Business intelligence, analytics, and reporting
- Analyzing **VPC Flow Logs**, **ELB (Elastic Load Balancer) logs**, **CloudTrail logs**, and other AWS platform logs stored in S3

> **Exam Tip:** "Serverless," "analyze data in S3," "SQL" → think **Amazon Athena**.

### 4.3 Amazon QuickSight

**QuickSight** is a **serverless, machine-learning-powered Business Intelligence (BI)** service used to build **interactive dashboards**.

**Key facts:**

- Lets you visually represent data (charts, graphs, dashboards) to give business users insights.
- **Fast**, **automatically scalable**, and **embeddable** (you can embed dashboards into your own applications).
- **Pricing:** per-session pricing — no servers to provision.
- Broad integration with many data sources: **RDS, Aurora, Athena, Redshift, Amazon S3**, and more.

**Use cases:** business analytics, building visualizations, ad-hoc analysis, generating business insights from data.

> **Exam Tip:** "Dashboards," "business intelligence," "visualize data" → think **Amazon QuickSight**.

### 4.4 Amazon Redshift

**Redshift** is AWS's managed **data warehousing** service, based on **PostgreSQL** under the hood — but functionally very different from a typical PostgreSQL/RDS database.

**OLTP vs. OLAP (important conceptual distinction):**

||RDS|Redshift|
|---|---|---|
|**Processing type**|**OLTP** — Online **Transaction** Processing|**OLAP** — Online **Analytical** Processing|
|**Purpose**|Handling many small, frequent read/write transactions|Running complex analytical queries and aggregations over large datasets|
|**Data loading pattern**|Continuous read/write|Data is typically loaded periodically (e.g., **every hour**), not continuously|

**Key facts:**

- Claims roughly **10x better performance** than typical data warehouse solutions and can scale to **petabytes** of data.
- Uses **columnar storage** (data organized and stored by column, rather than row) — this is highly efficient for analytical queries that aggregate values across many rows but only a few columns.
- Uses a **Massively Parallel Query Execution (MPP)** engine to execute complex computations very quickly across many nodes in parallel.
- **Pricing:** pay-as-you-go, based on the compute instances you provision.
- Supports a **SQL interface** for querying.
- Integrates with **BI (Business Intelligence) tools** such as **QuickSight** and **Tableau** for building dashboards on top of warehoused data.

> **Exam Tip:** "Data warehouse," "analytics," "OLAP," or "columnar storage" → think **Amazon Redshift**.

#### Redshift Serverless

- Runs Redshift **without needing to provision or manage the underlying data warehouse infrastructure or capacity**.
- You simply enable **Redshift Serverless** on your account, connect via the **Redshift Query Editor** (or another compatible tool), and start running queries.
- Redshift Serverless **automatically provisions and scales capacity** based on the actual workload of your queries.
- **Billing:** you pay only for the **compute and storage actually used** during analysis — a very cost-efficient model.

**Use cases:** reporting, dashboarding applications, real-time analytics — all **without managing underlying infrastructure**.

> **Exam Tip:** "Redshift" + "no infrastructure management" or "pay only for what you use" → think **Redshift Serverless**.

### 4.5 AWS Glue

**AWS Glue** is a **fully managed, serverless ETL (Extract, Transform, Load) service**.

**What is ETL?**

- **Extract** data from one or more sources
- **Transform** it into the correct/required format for downstream analytics
- **Load** it into a target destination for analysis

**How Glue fits into an architecture:**

```
[S3 Bucket]  ─┐
              ├─→ [AWS Glue: Extract → Transform] → [Amazon Redshift] (or other target)
[RDS DB]     ─┘
```

- Glue extracts raw data from multiple sources (e.g., an S3 bucket and an RDS database).
- You write a **transformation script** to reshape/clean the data.
- The transformed data is then **loaded** into a destination — for example, **Amazon Redshift** — for proper analytics.
- Traditionally, ETL required managing servers; with **Glue, the service is fully serverless**, so you only focus on the data transformation logic itself.

#### AWS Glue Data Catalog

- A **catalog/inventory of your datasets** across your AWS data infrastructure.
- Stores metadata: **column names, field names, field types**, etc.
- Used by other services — including **Athena, Redshift, and EMR** — to **discover datasets** and automatically build the appropriate query schemas.
- _(Noted in the transcript as likely outside the strict Cloud Practitioner exam scope, but useful context for understanding the Glue "family" of tools.)_

> **Exam Tip:** "Managed ETL service" → think **AWS Glue**.

### 4.6 AWS Database Migration Service (DMS)

**DMS** helps you **migrate databases** into (or between) AWS with minimal downtime.

**How it works:**

- You run an **EC2 instance hosting the DMS software**.
- DMS **extracts data from a source database**.
- DMS then **inserts/loads that data into a target database**, which could be a different service or engine entirely.

**Key benefits:**

- **Quick and secure** database migration.
- **Resilient and self-healing** — the migration process can recover from interruptions.
- **The source database remains available/operational during the migration** — no downtime required on the source side.

**Types of migrations supported:**

|Type|Description|Example|
|---|---|---|
|**Homogeneous migration**|Source and target use the **same** database technology|Oracle → Oracle|
|**Heterogeneous migration**|Source and target use **different** database technologies|Microsoft SQL Server → Aurora|

For heterogeneous migrations, DMS is intelligent enough to **convert/translate data structures** from the source engine format into the target engine format (often used alongside the **AWS Schema Conversion Tool** for more complex schema translations, though this wasn't explicitly named in the transcript).

> **Exam Tip:** "Migrate a database into AWS" or "migrate between database engines" → think **AWS DMS**.

---

## 5. Quick-Reference Summary Table (Exam Cram Sheet)

|Service|Category|Key Exam Keyword(s)|
|---|---|---|
|**RDS**|Relational (OLTP), SQL|Managed relational database, multiple engines (MySQL, PostgreSQL, MariaDB, Oracle, SQL Server, IBM DB2)|
|**Aurora**|Relational (OLTP), SQL|Cloud-native, high performance, MySQL/PostgreSQL compatible|
|**Aurora Serverless**|Relational, SQL|No management overhead, pay-per-second, unpredictable workloads|
|**ElastiCache**|In-memory cache|"In-memory database/cache," Redis/Memcached|
|**DynamoDB**|NoSQL (key-value)|Serverless, single-digit millisecond latency, key-value|
|**DAX**|Cache for DynamoDB|Cache specifically for DynamoDB, microsecond latency|
|**DocumentDB**|NoSQL (document)|MongoDB compatibility, JSON documents|
|**Neptune**|NoSQL (graph)|Graph database, highly connected data, social network|
|**Timestream**|NoSQL (time-series)|Time series data|
|**QLDB**|Ledger (centralized)|Financial transactions, immutable, cryptographically verifiable, central authority|
|**Managed Blockchain**|Ledger (decentralized)|Blockchain, Hyperledger Fabric, Ethereum, decentralized|
|**Redshift**|Data warehouse (OLAP), SQL|Data warehousing, analytics, columnar storage, OLAP|
|**Redshift Serverless**|Data warehouse|No infrastructure management, pay for what you use|
|**EMR**|Big data processing|Hadoop cluster, Spark, big data|
|**Athena**|Serverless query engine|Query S3 data with SQL, serverless analytics|
|**QuickSight**|BI / dashboards|Dashboards, visualizations, business intelligence|
|**Glue**|ETL|Managed ETL, extract-transform-load, Glue Data Catalog|
|**DMS**|Database migration|Migrate database into/between AWS databases|

---

## 6. Exam Tips — Final Recap

- **Managed vs. self-managed databases:** Managed AWS database services relieve you of provisioning, patching, backups, and HA setup — this is a recurring theme across nearly every AWS database service and a common exam angle on **cost/operational efficiency** and the **shared responsibility model**.
- **SQL = Relational** (RDS, Aurora, Redshift all support SQL, but RDS/Aurora are OLTP while Redshift is OLAP).
- **NoSQL = Non-relational**, flexible schema, horizontally scalable (DynamoDB, DocumentDB, Neptune, Timestream).
- Remember the **RDS deployment options**: Read Replicas (scale reads), Multi-AZ (high availability/failover), Multi-Region (disaster recovery + reduced latency, at the cost of replication network fees).
- **DAX is exclusively for DynamoDB**; **ElastiCache** is more general-purpose (works with RDS and other databases too).
- **QLDB (centralized) vs. Managed Blockchain (decentralized)** — both provide immutable/verifiable transaction history, but differ in trust/authority model.
- For **analytics on data already in S3** without loading it elsewhere → **Athena** (serverless SQL on S3).
- For a **big data / Hadoop** cluster → **EMR**.
- For a **fully managed ETL pipeline** → **Glue**.
- For **dashboards and visualization** → **QuickSight**.
- For **data warehousing / OLAP** → **Redshift** (and **Redshift Serverless** for infrastructure-free operation).
- For **migrating any database into or within AWS**, regardless of source/target engine → **DMS**.