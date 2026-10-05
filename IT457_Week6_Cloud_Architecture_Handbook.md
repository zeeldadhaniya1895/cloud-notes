# IT457 Cloud Computing — Week 6, Lecture 1
# Cloud Architecture Handbook

> A complete, beginner-friendly explanation of every slide, with examples, so nothing stays confusing.
> Items marked **(extra)** are added by me for clarity and are not literally on the slides.

---

## Table of Contents

1. [Design Goals for a Cloud Platform](#1-design-goals-for-a-cloud-platform)
2. [A Generic Security-Aware Cloud Architecture](#2-a-generic-security-aware-cloud-architecture)
3. [Layered Cloud Architecture (IaaS / PaaS / SaaS)](#3-layered-cloud-architecture-iaas--paas--saas)
4. [Market-Oriented Cloud Architecture](#4-market-oriented-cloud-architecture)
5. [Infrastructure Requirements for Virtualization](#5-infrastructure-requirements-for-virtualization)
6. [Architectural Design Challenges (6 Challenges)](#6-architectural-design-challenges)
7. [Architecture Styles Overview](#7-architecture-styles-overview)
8. [N-Tier Architecture Style (Deep Dive)](#8-n-tier-architecture-style-deep-dive)
9. [AWS Architecture Example](#9-aws-architecture-example)
10. [Traffic Manager in Multi-Region Deployment](#10-traffic-manager-in-multi-region-deployment)
11. [Three-Tier Layers](#11-three-tier-layers)
12. [Layers vs Tiers](#12-layers-vs-tiers)
13. [Process for Developing an N-Tier Architecture](#13-process-for-developing-an-n-tier-architecture)
14. [Google Cloud and VMware Cloud Architecture](#14-google-cloud-and-vmware-cloud-architecture)
15. [One-Page Revision Cheat Sheet](#15-one-page-revision-cheat-sheet)
16. [Practice Questions](#16-practice-questions)

---

## 1. Design Goals for a Cloud Platform

### What the slide says
- Main goals: **Scalability, Virtualization, Efficiency, Reliability**
- **Cloud management** software:
  1. Receives the user request
  2. Finds the correct resources
  3. Calls the provisioning services
  4. Invokes resources in the cloud
- It must support **both physical and virtual machines**.

### Explanation of each goal

| Goal | Meaning | Everyday example |
|---|---|---|
| **Scalability** | Ability to grow or shrink resources as demand changes | An exam-result website gets 100 visitors normally but 1,00,000 on result day. The cloud adds servers automatically, then removes them afterwards. |
| **Virtualization** | One physical server is split into many "virtual machines" (VMs), each acting like its own computer | One powerful server in a data center runs 10 VMs, each rented by a different customer. |
| **Efficiency** | Use hardware and money well, with no idle waste | Instead of 10 servers each at 10% usage, run 1 server at 100% usage using VMs. |
| **Reliability** | Service keeps working even if parts fail | If one server crashes, another copy takes over and users don't notice. |

### How cloud management works (step by step example)

Imagine you click **"Launch Instance"** on AWS:

1. **Receives the user request**: you ask for 2 CPUs, 4 GB RAM, Linux.
2. **Finds the correct resources**: the system searches which data-center server has free capacity.
3. **Calls provisioning services**: provisioning means *preparing and allocating* the resource (creating the VM, attaching disk and network).
4. **Invokes resources**: the VM starts, and you get an IP address to log in.

```
User request --> Cloud Management --> Find resources --> Provision --> Running VM
```

### Why "physical AND virtual machines"?
Some customers need a whole dedicated physical server (called *bare metal*, for very high performance or strict security). Others are happy with VMs. The management software must handle both.

---

## 2. A Generic Security-Aware Cloud Architecture

### Big picture
This slide shows how a **public cloud** is organised with security in mind. Think of it as a building with different departments.

```
            +---------------------------- PUBLIC CLOUD -----------------------------+
            |   Data centers (many)                                                  |
            |   Cloud platform: virtualized compute + storage + network + software   |
            |   + datasets from multiple data centers for multitenant applications   |
            +------------------------------------------------------------------------+
                         ^                      ^                      ^
                         |                      |                      |
   +--------------------------+   +------------------------------+   +-----------------------------+
   | Trust delegation,        |<->| Resource provisioning,       |<->| Security and performance    |
   | reputation systems,      |   | virtualization, management,  |   | monitoring                  |
   | data coloring            |   | and user interfaces          |   |                             |
   +--------------------------+   +------------------------------+   +-----------------------------+
              ^                                  ^
              |                                  |
           Clients  ------------------->  Services catalogs
```

### The components explained

| Component | What it does | Example |
|---|---|---|
| **Data centers** | Physical buildings full of servers, storage, networking | AWS has data centers in Mumbai, Virginia, etc. |
| **Cloud platform provisioning** | Combines compute, storage, network, software, and datasets from many data centers to serve many customers | Your VM may use compute from one rack and storage from another. |
| **Multitenant applications** | Many customers (tenants) share the same infrastructure, yet stay isolated | Gmail serves millions of users on the same system, but you never see someone else's mail. |
| **Services catalog** | A "menu" of services clients can choose from | AWS console list: EC2, S3, RDS, Lambda... |
| **Resource provisioning, virtualization, management, user interfaces** | The "control room" that creates and manages resources and gives the user a dashboard or API | AWS Management Console |
| **Security and performance monitoring** | Watches for attacks and slowness | Alerts when a server's CPU is at 99%, or when someone tries many wrong passwords. |
| **Trust delegation** | Letting one trusted party vouch for/act on behalf of another | Logging into a website using "Sign in with Google". Google vouches for you. |
| **Reputation systems** | Score providers/users by past behaviour so bad actors are caught | A data center or user with a history of misuse gets a low trust score. |
| **Data coloring** | Tagging data with labels (like color tags) so the system knows its sensitivity and who may access it | "Red" = confidential medical records, "Green" = public info. Red data gets stricter protection. |

### Features listed on the right side

| Feature | Meaning | Example |
|---|---|---|
| **Dynamic provisioning / deprovisioning** | Create resources when needed, remove when not | Spin up 20 servers for a sale, delete them after. |
| **Distributed storage** | Data spread over many machines | Google Drive stores your file in pieces/copies across several machines. |
| **Framework for processing large data** | Software that splits big jobs across many computers | Hadoop / MapReduce. |
| **Distributed File System** | A file system spanning many machines, looking like one | HDFS (Hadoop Distributed File System). |
| **SAN, DBs, Firewalls** | SAN = Storage Area Network (fast dedicated storage network); DBs = databases; Firewalls = traffic filters for security | A firewall blocks all traffic except port 443 (HTTPS). |
| **Web services** | Services accessed over the internet through standard protocols (HTTP/REST/SOAP) | A weather API your app calls. |
| **Monitoring / metering units** | Track usage and performance of provisioned resources | "You used 120 hours of VM time this month" which becomes your bill. |

### Private vs Public clouds (from this slide)
- **Private cloud: easier to manage.** Used by only one organisation, so it controls everything. *Example: a bank's own internal cloud.*
- **Public cloud: easier to access.** Anyone can sign up and use it. *Example: AWS, Azure, Google Cloud.*

---

## 3. Layered Cloud Architecture (IaaS / PaaS / SaaS)

### Picture in words
```
        Public clouds     Hybrid clouds     Private clouds        <- types of clouds (over the Internet)
              ^                 ^                 ^
   +--------------------------------------------------------+
   |  Provisioning of both physical and virtualized resources|
   +--------------------------------------------------------+
         ^                 ^                  ^
         |          Application layer (SaaS)  |
         |        Platform layer (PaaS)       |
         Infrastructure layer (IaaS, HaaS, DaaS, etc.)        <- bottom
```

### Three layers

| Layer | Contains | Who uses it | Example |
|---|---|---|---|
| **Infrastructure (IaaS)**: HW and virtualized HW | Servers, storage, networks (real and virtual) | System admins, DevOps | **Amazon EC2** |
| **Platform (PaaS)**: collection of software resources | OS, runtime, databases, development tools; used to *implement SaaS* | Developers | Google App Engine, AWS Elastic Beanstalk, Heroku |
| **Application (SaaS)** | Finished software used directly | End users | **Salesforce.com** (CRM), Gmail, Zoom |

### The "pizza" analogy (extra)
- **IaaS** = you get the kitchen. You bring ingredients and cook everything.
- **PaaS** = you get a ready kitchen with tools and dough. You add toppings (your code).
- **SaaS** = you order a cooked pizza. You just eat (use the app).

### What is a "layer" depending on another?
Each upper layer is built **on top of** the lower one. SaaS runs on PaaS, and PaaS runs on IaaS. The slide's small arrows show this. That is why the platform layer is said to *"implement SaaS"*.

### Other "aaS" terms in the diagram
- **HaaS**: Hardware as a Service (rent hardware itself)
- **DaaS**: Data as a Service (or Desktop as a Service, depending on context). Rent access to datasets.

### Examples from the slide

**Amazon EC2 (IaaS)**
- Provides **virtualized CPU resources** to users
- Provides **management of these provisioned resources** (start, stop, resize)
- *Example*: you rent a `t3.medium` VM, install Python and Django yourself, and run your website.

**Salesforce.com (SaaS)**
- A **CRM** (Customer Relationship Management) service: track customers, sales, leads.
- Slide note: *"Services at the application layer demand more work from providers."* Why? Because the provider must manage **everything**: hardware at the bottom, software at the top, **and** the platform plus tools for user application development and monitoring in the middle. The user does almost nothing, so the provider does the most work.

### Responsibility comparison (extra)

| You manage → | Applications | Data | Runtime | OS | Virtualization | Servers | Storage | Network |
|---|---|---|---|---|---|---|---|---|
| **IaaS** | You | You | You | You | Provider | Provider | Provider | Provider |
| **PaaS** | You | You | Provider | Provider | Provider | Provider | Provider | Provider |
| **SaaS** | Provider | Provider(*) | Provider | Provider | Provider | Provider | Provider | Provider |

(*) In SaaS you still own your data content, but the provider runs the systems storing it.

### Three cloud deployment types in the picture
- **Public cloud**: shared, over the Internet (AWS).
- **Private cloud**: for one organisation (company's own).
- **Hybrid cloud**: mix of both. *Example: a hospital keeps patient records in its private cloud but uses public cloud for its website.*

---

## 4. Market-Oriented Cloud Architecture

### Definition (from slide)
> A system design that manages and allocates cloud computing resources based on **economic models, supply and demand, and explicit service agreements**.

In simple words: the cloud acts like a **marketplace**. Customers pay for what they use, prices can vary, and a contract (SLA) promises a quality level.

### Key term: SLA
**SLA = Service Level Agreement.** A written promise from the provider. *Example: "99.99% uptime. If we fail, you get a refund/credit."*

### Components in the diagram

```
 Users / Brokers
       |
 +-------------------------------------------------------------+
 |  Service Request Examiner and Admission Control             |
 |   - customer-driven service management                      |
 |   - computational risk management                           |
 |   - autonomic resource management                           |
 +-------------------------------------------------------------+
     |            |             |               |
   Pricing    Accounting    (SLA Resource Allocator)
     |                           |
 VM monitor    Dispatcher    Service request monitor
     |
 Virtual Machines (VMs)
     |
 Physical Machines
```

| Component | Role | Example |
|---|---|---|
| **Users / brokers** | Customers, or middlemen who buy resources on behalf of many customers | A company submits a job; a broker finds the cheapest provider. |
| **Service request examiner & admission control** | Checks each request: *can we accept it without breaking promises to others?* | If servers are already full and accepting one more would violate other SLAs, the request is rejected or delayed. |
| **Customer-driven service management** | Services are shaped around customer needs and priority | A premium customer gets higher priority. |
| **Computational risk management** | Assesses the risk of failing to meet the SLA | "If we accept this heavy job now, is there a risk we miss the deadline?" |
| **Autonomic resource management** | System manages itself automatically with no human needed | Auto-adjusts resources when load increases. |
| **Pricing** | Decides the cost of a service (per-hour, peak/off-peak, spot price) | Cheaper at night, costlier at peak time. |
| **Accounting** | Records actual usage to generate bills | "Customer X used 50 VM-hours." |
| **VM monitor** | Tracks status/health of VMs | Is a VM running, idle, overloaded? |
| **Dispatcher** | Starts the accepted job on selected VMs | Sends your job to VM #7. |
| **Service request monitor** | Tracks progress of each request against SLA | Is the job finishing on time? |
| **SLA resource allocator** | The central block coordinating all of the above and assigning resources according to SLAs | Decides which VMs serve which customer. |
| **VMs and physical machines** | The actual resources at the bottom | Many VMs on one physical server. |

### Real-world example
**AWS Spot Instances**: unused capacity sold at lower, demand-based prices. If demand rises, price rises; that is supply-and-demand economics inside a cloud.

---

## 5. Infrastructure Requirements for Virtualization

The diagram has 3 main parts.

### (a) Infrastructure service layer (top)
Services needed to run the cloud as a business:

| Service | Meaning | Example |
|---|---|---|
| **Mirror management** | Keeping copies (mirrors) of images/data | Copies of OS images in many regions for fast download. |
| **System management** | Overall control and maintenance | Patching, updating servers. |
| **User management** | Accounts, roles, permissions | AWS IAM users and roles. |
| **System provision** | Creating/allocating systems | Launching new VMs. |
| **Account billing** | Charging for usage | Monthly invoice. |

### (b) Virtualized infrastructure (middle)
Controlled by the **Virtualized Integrated Manager**, which has:

| Function | Meaning | Example |
|---|---|---|
| **Load management** | Spread work evenly | If VM1 is at 90% and VM2 at 10%, shift work to VM2. |
| **Resource deployment** | Place resources where needed | Put a new VM on a free host. |
| **Security management** | Enforce security rules | Isolating one customer's VM from another. |
| **Data management** | Handle storage, backup, replication | Daily backup of volumes. |
| **Resource provision** | Allocate CPU/RAM/disk | Give a VM 4 GB RAM. |

Under it are **Virtual Solutions (A, B...)**, each with several **Virtual Machines**, each having an **Agent** (small program inside the VM that talks to the manager and reports status).

### Whitebox vs Blackbox management
- **Whitebox management**: the manager can *see inside* the VM through the agent (CPU use, processes, logs). More control.
  *Example: the manager sees a process in the VM using 100% CPU and responds.*
- **Blackbox management**: the manager treats the VM as a sealed box and only manages it from outside (start/stop/resource limits), with no agent needed.
  *Example: stopping a VM or limiting it to 2 CPUs without knowing what runs inside.*

### (c) Virtualized platform + physical resources (bottom)
Several **virtualized platforms** (hypervisor-based hosts) connected by network switches, backed by real **servers, switches, and storage disks** in the physical cloud.

**Hypervisor (extra):** software that creates and runs VMs. *Examples: VMware ESXi, KVM, Xen, Hyper-V.*

---

## 6. Architectural Design Challenges

### Challenge 1: Service Availability and Data Lock-in
- **Service availability**: users expect 24x7 service. If a single provider goes down, you are stuck. *Example: a major cloud outage takes down many websites.*
- **Data lock-in**: once your data and apps are in one provider's proprietary format, moving out is hard and costly. *Example: you built everything using a provider-specific database; switching to another provider needs a rewrite.*
- **Solutions**: use multiple providers (multi-cloud), use open standards, keep backups portable.

### Challenge 2: Data Privacy and Security Concerns
- Your data sits on someone else's hardware, shared with other tenants.
- Risks: data leaks, hacking, insider misuse, legal issues about where data is stored.
- *Example: a hospital storing patient records on a public cloud must follow privacy laws (HIPAA abroad; in India the Digital Personal Data Protection Act).*
- **Solutions**: encryption (at rest and in transit), access control, auditing, private/hybrid cloud for sensitive data.

### Challenge 3: Unpredictable Performance and Bottlenecks
- Many VMs share the same physical resources, so performance can vary (the "noisy neighbour" problem). Disk I/O and network are common bottlenecks.
- *Example: your VM is fast at 2 PM but slow at 8 PM because a neighbour VM is using the disk heavily.*
- **Solutions**: better scheduling, resource isolation, dedicated instances, monitoring.

### Challenge 4: Distributed Storage and Widespread Software Bugs
- Data is spread over many machines, so keeping copies consistent is difficult. A small software bug can spread across thousands of machines.
- *Example: a bug in the storage software causes data corruption in all replicas.*
- **Solutions**: testing, gradual rollouts, replication, version control, debugging tools for large-scale systems.

### Challenge 5: Cloud Scalability, Interoperability, and Standardization
- **Scalability**: must scale smoothly to large sizes.
- **Interoperability**: different clouds should work together. *Example: moving a VM from AWS to Azure.*
- **Standardization**: common standards (APIs, formats) are lacking; each provider does things differently.
- **Solutions**: standards such as OVF for VM formats, and open APIs.

### Challenge 6: Software Licensing and Reputation Sharing
- **Licensing**: traditional software licences are tied to a machine or user count. In the cloud, VMs appear and vanish, so licensing is complicated. *Example: paying per-core license for a database that runs on 100 temporary VMs.* New models like pay-per-use licensing are needed.
- **Reputation sharing**: if one customer on a shared IP address spams, the whole IP range can be blacklisted, harming innocent customers sharing it. *Example: one user's spam gets the shared email-sending IP blocked for everyone.*

---

## 7. Architecture Styles Overview

Different applications fit different structural styles. Here are all six from the slide.

### 7.1 N-Tier
- Traditional **layered** design.
- A higher layer calls lower ones, **not vice versa**.
- Can be a **liability**: hard to update individual components because layers depend on each other.
- Suitable for **application migration** (moving an existing app to the cloud).
- Generally uses **IaaS**.
- *Example: a college ERP moved from the campus server to cloud VMs.*

### 7.2 Web-Queue-Worker
- Purely **PaaS** (managed services).
- **Web front end (FE)** + **Worker back end (BE)**.
- FE and BE communicate through an **asynchronous message queue**.
- Suitable for **simple domains**.

```
User --> Web Front End --> [ Message Queue ] --> Worker (back end)
              ^                                     |
              +------------- Database <-------------+
```
*Example: an online photo editor. You upload a photo (web FE) and get an instant "received" message. The photo is put in a queue; a worker picks it up later, resizes it, and saves it. Your browser doesn't wait.*

**Async (asynchronous)**: sender doesn't wait for a reply. Opposite of sync, where it waits.

### 7.3 Microservices
- **Many small independent services**.
- Services are **loosely coupled** (changing one doesn't break others).
- Communicate through **API contracts** (agreed rules about requests/responses).
- *Example: Amazon-like shop with separate services for Login, Product Search, Cart, Payment, Delivery. If Payment is updated, Search is unaffected, and each can scale separately.*

### 7.4 Event-Driven Architecture
- Uses the **publish-subscribe (pub-sub)** model.
- **Publishers** and **Subscribers** are independent.
- Suitable for apps that **ingest large volumes of data**.
- Suitable when **different subsystems must perform different actions on the same data**.
- *Example: when you place an order, an "OrderPlaced" event is published. Subscribers: Inventory service reduces stock, Email service sends confirmation, Analytics service records the sale, and Billing service creates an invoice. The order service doesn't know or care about them.*

### 7.5 Big Data
- Specialised architecture for huge datasets.
- **Divides large datasets into chunks, processes them in parallel, and analyses the results.**
- *Example: analysing 10 TB of web logs. Split into 1,000 chunks processed on 1,000 machines at once (Hadoop/Spark).*

### 7.6 Big Compute
- Specialised for heavy computation (HPC, high-performance computing): many CPU/GPU cores working on one big calculation.
- *Example: weather simulation, drug-molecule simulation, training a large AI model, rendering a movie.*

### Quick comparison

| Style | Best for | Key idea | Example |
|---|---|---|---|
| N-tier | Migration, simple web apps | Layered, IaaS | College ERP |
| Web-Queue-Worker | Simple domains | Web + queue + worker (PaaS) | Photo processing site |
| Microservices | Large, evolving apps | Small independent services | Amazon, Netflix |
| Event-driven | High-volume data, many reactions | Pub-sub | Order events |
| Big Data | Huge datasets | Chunk + parallel analysis | Log analysis |
| Big Compute | Heavy calculations | Parallel computing | Weather simulation |

---

## 8. N-Tier Architecture Style (Deep Dive)

### Definition
An N-tier architecture divides an application into **logical layers** and **physical tiers**.

- **Layers**: a way to **separate responsibilities** and **manage dependencies**.
- **Tiers**: **physically separated**, running on **separate machines**.

### The diagram explained

```
Client --> WAF --> Web Tier --> Middle Tier 1 --> Remote Service
                       |             |   ^
                       |             v   |
                       |          Data Tier <-- Cache
                       |             ^
                       +--> Messaging --> Middle Tier 2
```

| Block | Role | Example |
|---|---|---|
| **Client** | User's browser or app | Chrome on your laptop |
| **WAF** (Web Application Firewall) | Filters malicious web requests (SQL injection, XSS) before they reach servers | Blocks a request containing `' OR 1=1 --` |
| **Web Tier** | Receives requests, serves pages | Nginx / Apache |
| **Middle Tier 1 / 2** | Business logic. Can be several, each handling different work | Tier 1: handle orders. Tier 2: handle background tasks. |
| **Messaging** | Queue between tiers for async work | RabbitMQ, Amazon SQS |
| **Cache** | Fast temporary memory for frequently used data, reducing load on the database | Redis / Memcached keeps the "top 10 products" |
| **Data Tier** | Permanent storage | MySQL, PostgreSQL |
| **Remote Service** | An external service the app calls | Payment gateway, SMS service |

### When to use this architecture
- Typically implemented as **IaaS** applications, with **each tier running on a separate set of VMs**.
- Consider N-tier for:
  1. **Simple web applications**
  2. **Migrating an on-premises application to the cloud with minimal refactoring** ("lift and shift"): move without rewriting much code.
  3. **Unified development of on-premises and cloud applications**: same design works in both places.

### Pros and cons (extra)
| Pros | Cons |
|---|---|
| Familiar and easy to understand | Changes can ripple across layers, so updating is harder |
| Each tier can scale independently | Latency from network hops between tiers |
| Good security separation (DB hidden behind other tiers) | Can become rigid or monolithic over time |

---

## 9. AWS Architecture Example

This slide shows how an N-tier app is built on AWS.

### Tiers (from slide)
- **Web Tier**: web servers such as **Apache**.
- **App Tier**: application servers.
- **Database Tier**: database services such as **Amazon RDS** (Relational Database Service).

### Walk through the request flow
Imagine a user opens `yourApp.com`:

1. **Hosted Zone (Route 53)**: DNS converts `yourApp.com` to an IP address.
2. **Elastic Load Balancing (ELB)**: spreads incoming traffic across multiple web servers.
3. **Web Servers in an Auto Scaling Group**: if traffic rises, AWS automatically adds more web servers; if it falls, it removes them.
4. **App Servers in another Auto Scaling Group**: run the business logic, also auto-scaled.
5. **Amazon ElastiCache**: caches frequent data for speed.
6. **RDS DB Instance (Master "M")**: main database for reads/writes.
7. **RDS DB Instance Standby, Multi-AZ ("S")**: standby copy in a different availability zone. If the master fails, the standby takes over.
8. **Bucket (S3)** and **Amazon CloudFront**: store static files (images, videos) and deliver them quickly through a CDN at `media.yourApp.com`.

### Other services in the picture (left side)
| Service | Purpose |
|---|---|
| **CloudWatch** | Monitoring and alarms (CPU, memory, errors) |
| **Email notification (SNS/SES)** | Alerts and emails |
| **Amazon DynamoDB** | NoSQL database for fast key-value data |
| **Amazon SES** | Simple Email Service for sending emails |

### Key words
- **Region**: geographic area (e.g., Mumbai `ap-south-1`).
- **AZ (Availability Zone)**: a separate data center within a region. The diagram uses **AZ1 and AZ2** so that if one fails, the other keeps serving. This gives reliability.
- **Auto Scaling Group**: automatically adjusts the number of servers.
- **CDN (CloudFront)**: copies of static content stored close to users worldwide.

---

## 10. Traffic Manager in Multi-Region Deployment

### Why is it needed?
If your app runs in multiple regions (e.g., India and USA), something must decide **which region serves each user**. That is the Traffic Manager's job.

*(Extra: "Traffic Manager" is the name of Microsoft Azure's service. AWS's equivalent is Route 53; Google has Cloud DNS/Load Balancing.)*

### What it does (from slide)
- Lets you **control traffic distribution** across application endpoints.
- Distributes traffic using the configured **routing method**.
- **Constantly monitors endpoint health.**
- **Automatic failover** if an endpoint fails.
- Works at the **DNS level**: it answers the DNS query with the address of the best endpoint, according to the traffic-routing rules.

### Routing methods

| Method | How it decides | Example |
|---|---|---|
| **Priority** | Always send to the primary; use backup only if primary fails | Primary in Mumbai, backup in Singapore. |
| **Weighted** | Split traffic by percentage | 80% to the old version, 20% to the new version (testing a release). |
| **Performance** | Send user to the endpoint with the lowest latency | A user in Gujarat gets routed to the Mumbai region, the fastest for them. |
| **Geographic** | Based on user's location (legal/regulatory) | EU users always go to EU servers for data-law reasons. |

### How DNS-level routing works
1. User types `myapp.com`.
2. The user's device asks DNS: "What's the IP of `myapp.com`?"
3. Traffic Manager checks health and rules, then replies with the IP of the best endpoint.
4. User connects directly to that endpoint.

Traffic Manager never carries your actual data. It only gives directions, like a traffic police officer pointing you to a road.

### Operation modes

| Mode | Meaning | Example |
|---|---|---|
| **Active / Passive: hot failover** | Backup region is already running and ready; switch is almost instant | Standby servers always on, so failover in seconds. More cost, faster recovery. |
| **Active / Passive: cold failover** | Backup region is off or minimal; it must start up on failure | Servers are powered on only when the primary fails. Cheaper, slower recovery. |
| **Active / Active: load balanced** | Both regions serve users at the same time | India and USA both handle traffic; if one fails, the other takes all load. |

---

## 11. Three-Tier Layers

### The three tiers

| Tier | Contents | Technologies | Example (online book store) |
|---|---|---|---|
| **Presentation / Web tier** | User interface in the browser | HTML, CSS, JavaScript | The page where you browse books and click "Buy" |
| **Application / Logic tier** | Business logic | Python, Java | Calculate total price, apply discounts, check stock |
| **Data / DB tier** | Stores and retrieves data | RDBMS (MySQL, PostgreSQL) or NoSQL (MongoDB) | Table of books, customers, orders |

### The network
- Acts as **the boundary between tiers**.
- A tier must make a **network call** to interact with another tier. *Example: the app tier sends a query over the network to the database tier.*

### Developing a multi-tier application: components needed
1. **Code that defines a message queue** for communication between tiers.
2. **Code that defines an API and a data model**. The API is the doorway to the app, and the data model describes the structure of data (e.g., Book: id, title, price).
3. **Security-related code** ensuring appropriate access (login, roles, authentication/authorization).

### Full flow example: buying a book
1. You click "Buy" (Presentation tier).
2. Browser sends an API request to the Application tier.
3. App tier checks you are logged in (security), checks stock, calculates price.
4. App tier asks the Data tier to save the order.
5. Data tier stores it and confirms.
6. App tier sends success back; the UI shows "Order placed!"

---

## 12. Layers vs Tiers

This is a very common source of confusion. Here it is clearly:

| Term | Meaning |
|---|---|
| **Layer** | A **functional (logical) division** of the software. About *organisation of code*. |
| **Tier** | A functional division that **runs on infrastructure separate** from the other divisions. About *where it runs*. |

### Key point
A **presentation tier runs on one infrastructure, but can have multiple layers** inside it.

### Slide's example
The **Contacts app on your phone** has three layers (UI, logic, data storage), but it is a **single-tier application**, because all three layers run on your phone.

### More examples

| App | Layers | Tiers | Explanation |
|---|---|---|---|
| Phone Contacts app | 3 (UI, logic, data) | **1** | Everything on one device |
| Calculator app on laptop | 2-3 | **1** | Everything runs locally |
| Online book store (typical) | 3 | **3** | Web server, app server, DB server on separate machines |
| Small website with one server running web + app + DB | 3 | **1** | Layers exist in code, but all on one machine |

> **Memory trick:** *Layers = how you organise the code (like chapters in a book). Tiers = where it physically lives (like which shelf/room the book is stored in).*
> Every tier has at least one layer, but layers don't always mean separate tiers.

---

## 13. Process for Developing an N-Tier Architecture

**Problem given:** *Automate the online selling of books to customers.*

### Step I: Define main functions / purpose
What is the system for? Here: let customers browse, order, and pay for books online; let the owner manage inventory.

### Step II: Define necessary business processes that require automation
List the real-world activities to automate:
- Customer registration and login
- Searching and browsing books
- Adding to cart
- Placing orders and payment
- Inventory update
- Order tracking and delivery
- Sending invoices and emails

### Step III: Translate business processes into requirements
Turn each process into technical needs:
- *Functional*: "System must let users search by title, author, ISBN."
- *Non-functional*: "Handle 5,000 concurrent users; pages load in under 2 seconds; payments must be secure."

### Step IV: Define software architecture
Decide the structure: layers, components, technologies, and how they communicate.
- Presentation: React / HTML-CSS-JS
- Logic: Python (Django) or Java (Spring)
- Data: PostgreSQL
- Communication: REST APIs, a message queue for order emails

### Step V: Define number of server tiers
Decide how many separate physical/virtual tiers:
- **2-tier**: client + DB (simple, small)
- **3-tier**: web + app + DB (most common; ideal for the book store)
- **4+ tier**: add cache, messaging, extra middle tiers for larger scale

For the book store: **3 tiers** (maybe 4 with a cache tier as traffic grows).

---

## 14. Google Cloud and VMware Cloud Architecture

The slide only lists these two names as topics, with no detailed content. Here is a short background **(extra)** so you know what they refer to.

### Google Cloud Platform (GCP) architecture
- Built on Google's global infrastructure (the same that runs Search, YouTube, Gmail).
- Organised into **regions** and **zones**, similar to AWS.
- Main service families:
  - **Compute**: Compute Engine (IaaS VMs), App Engine (PaaS), Cloud Run, Google Kubernetes Engine (containers)
  - **Storage**: Cloud Storage (object storage, like S3)
  - **Databases**: Cloud SQL, Bigtable, Spanner, Firestore
  - **Big data / AI**: BigQuery, Dataflow, Vertex AI
- Equivalent of AWS N-tier: Cloud Load Balancing, then Compute Engine instances in managed instance groups, then Cloud SQL.

### VMware Cloud architecture
- VMware is a leader in **virtualization**.
- Core product: **ESXi** hypervisor, which runs VMs on physical servers.
- **vCenter** manages many ESXi hosts centrally.
- **vSphere** is the virtualization suite; **VMware Cloud** extends this to build private/hybrid clouds (also available on public clouds like AWS).
- Typical use: an enterprise runs its own private cloud on VMware and connects it to a public cloud (hybrid cloud).

> **Tip:** If your instructor discusses these in the next lecture, treat the above as background. Update with the lecture's exact diagram.

---

## 15. One-Page Revision Cheat Sheet

### Core ideas
- **Goals**: Scalability, Virtualization, Efficiency, Reliability.
- **Cloud management**: receive request → find resources → provision → invoke.
- **Private** = easier to manage; **Public** = easier to access; **Hybrid** = both.

### Service layers
| IaaS | PaaS | SaaS |
|---|---|---|
| Hardware/VMs (EC2) | Platform tools for developers | Ready apps (Salesforce) |

### Market-oriented cloud
Economics + supply/demand + **SLA**. Components: admission control, pricing, accounting, VM monitor, dispatcher, service request monitor.

### Virtualization infra
Infrastructure services (mirror, system, user, provision, billing) + Integrated manager (load, deployment, security, data, provision) + Agents in VMs. **Whitebox** (inside view) vs **Blackbox** (outside view).

### 6 Challenges
1. Availability and data lock-in
2. Privacy and security
3. Unpredictable performance/bottlenecks
4. Distributed storage and software bugs
5. Scalability, interoperability, standardization
6. Software licensing and reputation sharing

### 6 Architecture styles
N-tier · Web-Queue-Worker · Microservices · Event-driven · Big Data · Big Compute

### N-tier
Logical **layers** + physical **tiers**. IaaS, each tier on separate VMs. Good for simple web apps and lift-and-shift migration.

### AWS 3 tiers
Web (Apache) · App (app servers) · DB (RDS). Plus ELB, Auto Scaling, ElastiCache, S3, CloudFront, CloudWatch.

### Traffic Manager
DNS-level routing. Methods: **Priority, Weighted, Performance, Geographic**. Modes: Active/Passive hot, Active/Passive cold, Active/Active.

### Layers vs Tiers
Layer = logical code division. Tier = runs on separate infrastructure. Contacts app = 3 layers, 1 tier.

### N-tier development steps
1. Define purpose → 2. Business processes → 3. Requirements → 4. Software architecture → 5. Number of tiers.

---

## 16. Practice Questions

**Short answer**

1. Name the four design goals of a cloud platform and give one example of each.
2. What does "multitenant" mean?
3. What is the difference between IaaS, PaaS, and SaaS? Give one example each.
4. Why do application-layer (SaaS) services "demand more work from providers"?
5. Define SLA. Why is it central to a market-oriented cloud architecture?
6. Explain whitebox vs blackbox management of VMs.
7. Explain "data lock-in" and one way to avoid it.
8. What is the difference between a layer and a tier? Why is the phone Contacts app single-tier?
9. List the four routing methods of Traffic Manager, with an example for each.
10. Differentiate hot failover vs cold failover.

**Think-and-apply**

11. A startup wants to move its existing college-admission website from a local server to the cloud quickly, with minimal code changes. Which architecture style would you choose and why?
12. An e-commerce site wants separate emails, inventory updates, and analytics every time an order is placed. Which style fits best? Explain using pub-sub.
13. Design a 3-tier AWS architecture for an online book store. Name the service for each tier, and mention how you'd make it highly available.
14. Your app serves users in India and Europe, and EU law requires EU data to stay in the EU. Which Traffic Manager routing method would you use?

**Sample answers (check yourself)**

- Q11: **N-tier** (IaaS): supports lift-and-shift migration with minimal refactoring.
- Q12: **Event-driven**: the Order service publishes an "OrderPlaced" event; Email, Inventory, and Analytics services are independent subscribers.
- Q13: Web tier = EC2 with Apache behind ELB (Auto Scaling); App tier = EC2 app servers (Auto Scaling); DB tier = RDS with Multi-AZ standby; ElastiCache for caching; S3 + CloudFront for static media; CloudWatch for monitoring; deploy across at least two AZs.
- Q14: **Geographic** routing.

---

*End of handbook. Good luck with IT457!*
