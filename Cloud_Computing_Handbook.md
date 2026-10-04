# Cloud Computing Handbook
### Elastic Computing & the Cloud Landscape
*Based on: Week 5 – Lecture 1 (Chapters 2 & 3)*

---

## How to Use This Handbook

Every section follows the same pattern:

1. **The idea**: what the concept is, in plain words
2. **Example**: a real-life or business example
3. **Why it matters / Common confusion**: so you don't get stuck later

At the end you will find a **comparison cheat sheet**, a **glossary**, a **self-test with answers**, and a **one-page revision summary**.

---

## Table of Contents

1. [Why and When Do We Use Cloud?](#1-why-and-when-do-we-use-cloud)
2. [What Is a Data Center?](#2-what-is-a-data-center)
3. [Why Is It Called "Cloud" Computing?](#3-why-is-it-called-cloud-computing)
4. [Multi-Tenant Clouds](#4-multi-tenant-clouds)
5. [Elastic Computing](#5-elastic-computing)
6. [Virtualization](#6-virtualization)
7. [Who Benefits from Virtualization, and How](#7-who-benefits-from-virtualization-and-how)
8. [Service Models: IaaS, PaaS, SaaS (and DaaS)](#8-service-models-iaas-paas-saas-and-daas)
9. [Cloud Types: Private vs Public](#9-cloud-types-private-vs-public)
10. [Why Choose Public Cloud?](#10-why-choose-public-cloud)
11. [Why Choose Private Cloud?](#11-why-choose-private-cloud)
12. [Hybrid Cloud and Multi-Cloud](#12-hybrid-cloud-and-multi-cloud)
13. [Hyperscalers: The Big Players](#13-hyperscalers-the-big-players)
14. [Master Comparison Cheat Sheet](#14-master-comparison-cheat-sheet)
15. [Common Confusions Cleared](#15-common-confusions-cleared)
16. [Glossary](#16-glossary)
17. [Self-Test (with Answers)](#17-self-test-with-answers)
18. [One-Page Revision Summary](#18-one-page-revision-summary)

---

## 1. Why and When Do We Use Cloud?

### The idea
Cloud computing means **renting computing resources** (servers, storage, software) over the Internet instead of buying and running them yourself. You use them when you need them and pay for what you use.

### The use cases from the lecture, each explained

| Use case from slide | What it means | Concrete example |
|---|---|---|
| **Startup leases cloud facilities for its website, pays as required** | A new company does not buy servers up front. It rents capacity and pays based on actual usage. | A food-delivery startup launches with 2 rented servers. After a viral social media post, it rents 20. It never bought hardware. |
| **Smartphone and Smart Home systems** | Your device is small. The heavy work (voice recognition, storing history, syncing) happens in the cloud. | You say "Hey Google, turn off the lights". Your voice goes to cloud servers, which interpret it and send a command back to your bulb. |
| **An enterprise leases facilities and software for business functions** (payroll, accounting, billing) | A big company rents ready-made business software rather than building and hosting its own. | A company uses a cloud payroll service. Employees' salaries are processed by the vendor's software on the vendor's servers. |
| **Browser-based editing of shared documents** | The document lives in the cloud, and many people edit the same copy. | Five students edit one Google Doc at once and everyone sees changes instantly. |
| **Medical device that periodically uploads readings for analysis; alert systems** | A small device collects data and the cloud analyzes it and raises alerts. | A wearable heart monitor uploads readings every minute. If the cloud detects a dangerous pattern, a doctor gets an alert. |
| **Seasonal company leases computing for 5 peak months; no payment at other times** | Demand is not constant. Pay only when you need extra capacity. | An online Diwali-shopping store rents big capacity from September to January and almost none for the other 7 months. |
| **Log into a social media site and upload photos** | You use cloud services every day without noticing. | Uploading a photo to Instagram means it is stored in the provider's data centers and served to your followers. |

### Why it matters
The common thread in all examples is **pay as required**. The cloud turns computing from a *big upfront purchase* (capital expense) into a *usage-based bill* (operating expense), like electricity or a taxi rather than buying a car.

---

## 2. What Is a Data Center?

### The idea
> A **data center** is a dedicated physical facility used by organizations to house their critical applications, data, and IT infrastructure.

The cloud is *not magic*. Behind every cloud service there is a **physical building full of machines**.

### What makes up a data center

| Component | What it does | Everyday analogy / example |
|---|---|---|
| **Server racks** | Tall metal frames that hold servers, storage and network devices stacked vertically. | A bookshelf, but for computers. |
| **Servers** | The computers that run applications and process requests. | The "brains" answering your request when you open a website. |
| **Storage** | Devices that hold data (disks, SSD arrays). | The warehouse where all your photos and files are kept. |
| **Networking devices** | Switches and routers connecting servers to each other and to the Internet. | The road system connecting buildings. |
| **Cooling system** | Removes the heat servers produce. | Like AC in a crowded hall. Without it, machines overheat and fail. |
| **Power backup (DG, UPS)** | **UPS** (Uninterruptible Power Supply) covers the first seconds or minutes of a power cut using batteries. **DG** (Diesel Generator) takes over for longer outages. | Power cut at home: inverter (UPS) switches on instantly, and a generator runs the building for hours. |
| **Smoke detectors & fire-fighting equipment** | Detect and put out fires, using special systems that don't ruin electronics. | Fire alarms and sprinklers in a building, but gas-based so water doesn't destroy servers. |
| **Fireproof walls** | Stop a fire from spreading between rooms. | Fire-resistant doors in hospitals. |
| **Anti-rodent measures** | Rodents chew cables and cause shorts. | Pest control, but for cables. |
| **Centralized monitoring** | A control room that watches temperature, power, network and security. | Air-traffic control, but for the facility. |

### Rack and server sizes, explained step by step

- A standard rack is about **6.2 feet tall and 19 inches wide**, usually a **42U** rack.
- **1U (one rack unit) = 1.75 inches** of height.
- Servers and storage devices come in sizes like **1U, 2U, 4U**.
- A **chassis** (about **7U**) is a box in which servers are stored **vertically**, like blades. This packs more servers into less space.

**Worked example:**
- A 42U rack has 42 × 1.75 = **73.5 inches** of usable height (about 6.1 ft, roughly the 6.2 ft on the slide, which includes the frame).
- If every server is 1U, you can fit up to **42 servers** in one rack.
- If every server is 2U, you can fit **21 servers**.
- If you use a 7U chassis, you can fit **6 chassis** (6 × 7 = 42U), and each chassis holds many servers, so the rack holds far more computers.

### Common confusion
**"Is the data center the cloud?"** Not exactly. A data center is the physical building. The *cloud* is the **service model** built on top of one or more data centers (renting out resources from them over the Internet).

---

## 3. Why Is It Called "Cloud" Computing?

### The question
Servers sit in a building on the ground. So why say computing happens "in the cloud"?

### The technical truth
- Saying a data center is "in the cloud" is **technically inaccurate**.
- Servers in a data center are **not part of the Internet itself**. They are computers that **attach to** the Internet.
- In network diagrams, the Internet is drawn as a **cloud shape**. Data centers and servers are drawn **outside** the cloud, with connection lines leading into it.

### Where the term came from
1. Early data centers needed **very fast links to Internet backbones** (the main high-speed highways of the Internet).
2. Placing a data center **near a peering point** (where backbones connect to each other) **cut costs**.
3. Some data centers were literally in **the same building or even the same floor** as the peering equipment.
4. Because they were so close to the "cloud" in the diagrams, engineers started saying the data center was **"in the cloud"**.
5. The phrase **stuck**, even though it is not strictly accurate.

**Analogy:** If your shop is right next to the main highway junction, people might say "your shop is *on the highway*". It isn't, but it's so close that the phrase sticks.

### Scaling a website, the diagram on the slide

The slide asks: *How can a website scale to accommodate thousands of users?*

```
                          +----------+
                     +--> | server 1 |--+
 Internet --> load -------> server 2 |--+--> company
 (incoming    balancer +--> server 3 |--+    database
  traffic)            +--> ...       |--+
                      +--> server N  |--+
```

- **Incoming web traffic** arrives from the Internet.
- A **load balancer** spreads the requests evenly. With N servers, each gets **1/N of the traffic**.
- All servers read from and write to the same **company database**.

**Example:** 10,000 users visit your site at once. With **5 servers**, each handles about **2,000 users** (10,000 ÷ 5). If traffic doubles to 20,000, add 5 more servers (10 total) and each *still* handles 2,000. That is how a site scales: **add more servers behind a load balancer**.

---

## 4. Multi-Tenant Clouds

### The idea
> A **multi-tenant cloud** is a data center that serves customers from **many different organizations at the same time**.

A *tenant* is a customer, like a tenant renting a flat in a big apartment building. Many tenants share one building (the data center), but each has a private, locked flat (their own isolated resources).

### Why it saves money
1. **Consolidating servers** into one data center already saves money:
   - **Quantity discounts** on hardware (buying 10,000 servers is cheaper per unit than buying 10).
   - **Less IT staff overhead** (one team looks after everything).
2. Cloud providers take this to a **much larger scale**. One facility handles computing for **many customers at once**, not just one organization.

**Example:** Org A, Org B, Org C and Org D (the four boxes on the slide) all run inside the same shared data center. Instead of building four small data centers, they share one big one. The provider's cost per customer drops, and so does the price they charge.

### Multi-tenancy inside one company
Multi-tenancy also happens **within a single organization**.

**Example:** A company's finance department keeps its data **fully separate** from the rest of the company. HR and Marketing share the same infrastructure but cannot see finance data. Finance, HR and Marketing are "tenants" of the company's internal system.

### What changes when a data center serves many organizations?
The slide's discussion question. The answer:
- **Isolation/security** becomes critical, because tenants must not see each other's data.
- **Resource sharing** becomes the business model, since costs are spread across many customers.
- **Billing per customer** becomes necessary.
- **Scale** grows, giving better discounts and more expertise.

---

## 5. Elastic Computing

### The idea
> **Elastic computing**: lease only what you need, and **change it at any time**.

Like an elastic band, capacity **stretches** when demand rises and **shrinks** when demand falls.

### The problem it solves
*How can a company avoid paying for servers it isn't using, without running out of capacity during a busy period?*

Without elasticity you have two bad choices:
- **Buy for the peak**: you waste money most of the year (idle servers).
- **Buy for the average**: you crash during busy periods (not enough servers).

### The three points from the slide

**1. Pay for what you use**
Customers lease a few servers or many, and pay only for that number.
*Example:* Lease 3 servers, pay for 3. Lease 50, pay for 50.

**2. Change allocation dynamically**
Add servers during peak times, reduce them when demand drops.
*Example:* A ticket-booking site for a cricket match. Normal day: 4 servers. Hours before the booking opens: scale to 80 servers. Afterward: back to 4. You pay for 80 servers for only a few hours.

**3. Early approach: fixed sizes**
Early clouds offered **rack, half-rack, or quarter-rack** allocations. This was flexible, but **still tied to physical resources** (you got a physical chunk of hardware).
*Example:* Like renting a full shop, half a shop, or a quarter of a shop, but not "just a shelf for one hour". That finer control came later with virtualization (next section).

### Worked example with numbers (seasonal business)
A store needs 100 servers for 5 peak months and 10 servers for the other 7 months.

- **Buying for peak (no elasticity):** 100 servers × 12 months = **1,200 server-months**.
- **Elastic:** (100 × 5) + (10 × 7) = 500 + 70 = **570 server-months**.
- **Saving:** 630 server-months, which is about **52% less** than buying for the peak.

---

## 6. Virtualization

### The question
*If a data center never reboots or reconfigures a physical machine, how can it launch a brand-new server in milliseconds?*

### The answer
Providers **do not hand out physical machines**. They run **software** that creates **virtualized servers**.

> A **virtualized server** is a **software artifact**, a "pretend computer" running inside a real computer.

### The three key properties

#### 6.1 Rapid creation and removal
- Managed **entirely by software**.
- A server can be created or removed **at any time**, with **no reboot** of the physical machines.
- **Example:** Like opening a new tab in your browser instead of buying a new laptop. It takes a click, not a shipment.

#### 6.2 Physical sharing
- Many virtualized servers can **run on one physical server at the same time**.
- **Example:** One physical server with 64 CPU cores and 256 GB RAM could host 16 small virtual servers, each with 4 cores and 16 GB. Like one big apartment building divided into many flats.

#### 6.3 Logical isolation
- Each virtualized server is **fully isolated**.
- **No other tenant can observe or affect its data or computations.**
- **Example:** Company A's virtual server and Company B's virtual server may sit on the *same physical machine*. If Company A's server crashes or gets infected, Company B's is unaffected and cannot read A's files. Like flats in a building: you can hear nothing through the walls and you hold the only key to yours.

### Summary diagram

```
        ONE PHYSICAL SERVER
 +------------------------------------+
 |  Virtualization software           |
 |  +--------+ +--------+ +--------+  |
 |  | VM  A1 | | VM  B1 | | VM  C1 |  |
 |  | Org A  | | Org B  | | Org C  |  |
 |  +--------+ +--------+ +--------+  |
 +------------------------------------+
   (isolated from each other)
```

---

## 7. Who Benefits from Virtualization, and How

### 7.1 Benefits for the Provider

| Benefit | Explanation | Example |
|---|---|---|
| **Creation in milliseconds, not minutes** | Scales to **thousands** of virtualized servers. | A customer clicks "launch server" and it is ready almost immediately. Without virtualization, you'd wait for hardware to be set up. |
| **No need to reconfigure or reboot physical servers** | The physical machine keeps running, and software creates the new server on top of it. | Adding a new customer does not require shutting down hardware that others are using. |
| **Isolation lets tenants mix freely on the same physical server, with zero data-leakage risk** | Because VMs are isolated, the provider can pack any customers together. | A bank's VM and a gaming startup's VM share a machine safely. |
| **Load balancing** | Place new servers on **lightly loaded** machines to avoid overload while others sit idle. | Physical Server 1 is at 90% load and Server 2 is at 20%, so the new VM is placed on Server 2. |

*Note:* "Zero data leakage risk" is how the slide describes the design goal of isolation. In practice, providers invest heavily in security to approach this.

### 7.2 Benefits for the Customer

| Benefit | Explanation | Example |
|---|---|---|
| **Acts just like a physical server, with its own Internet address** | You can use it the same way as a real machine. | You get a virtual server with its own IP address, log in, install software, and host a website. |
| **Ease of creating and deploying new services with provider/vendor tooling** | Dashboards, templates and tools make setup quick. | Pick "Ubuntu + web server" from a menu and it's running in minutes. |
| **Rapid scaling: add copies of an app as requests arrive** | When traffic grows, launch more copies of your app. | During a flash sale, you add 10 more copies of your shopping app to handle the load. |
| **Safe, isolated testing before production** | Test new software in a separate virtual server without risking the live system. | A developer tries a risky update on a test VM. If it breaks, the live website is untouched. |

---

## 8. Service Models: IaaS, PaaS, SaaS (and DaaS)

### The big idea
*If you just want to run a website, do you really need to buy and manage your own servers?* No. Cloud lets you choose **how much you manage yourself**.

> **Rule:** The more you move up the stack (IaaS → PaaS → SaaS), the **more the provider manages** and the **less you manage**.

### The pizza analogy (to remember it forever)

| Model | Pizza analogy | Who does what |
|---|---|---|
| **On-premises** | **Make pizza at home** | You do everything: ingredients, oven, cooking, serving. |
| **IaaS** | **Take-and-bake**. You buy the base, oven provided | Provider gives the kitchen basics. You choose the toppings and cook. |
| **PaaS** | **Pizza delivered, you add your own sides and drinks** | Provider handles most things. You handle the final touches. |
| **SaaS** | **Dining out** | Provider does everything. You just eat. |

### The full stack: "Separation of Responsibilities"

The slide shows 9 layers, from top to bottom:

1. Applications
2. Data
3. Runtime
4. Middleware
5. O/S (Operating System)
6. Virtualization
7. Servers
8. Storage
9. Networking

**Who manages which layers?**

| Layer | On-Premises | IaaS | PaaS | SaaS |
|---|---|---|---|---|
| Applications | **You** | **You** | **You** | Provider |
| Data | **You** | **You** | **You** | Provider |
| Runtime | **You** | **You** | Provider | Provider |
| Middleware | **You** | **You** | Provider | Provider |
| O/S | **You** | **You** | Provider | Provider |
| Virtualization | **You** | Provider | Provider | Provider |
| Servers | **You** | Provider | Provider | Provider |
| Storage | **You** | Provider | Provider | Provider |
| Networking | **You** | Provider | Provider | Provider |

**Reading the table:**
- **On-premises:** you manage all 9 layers.
- **IaaS:** you manage the top 5 (Applications, Data, Runtime, Middleware, OS). The provider manages the bottom 4.
- **PaaS:** you manage only the top 2 (Applications, Data).
- **SaaS:** the provider manages all 9. You just use the software.

### Explaining the layers (so no term is confusing)

| Layer | Plain-English meaning | Example |
|---|---|---|
| **Applications** | The software you actually use or build. | A shopping website, an accounting app. |
| **Data** | The information the application stores and uses. | Customer records, orders, photos. |
| **Runtime** | The environment that runs program code. | Java Runtime, the .NET runtime. |
| **Middleware** | Software that sits between the OS and your application, helping them communicate. | A message queue, a web server, an application server. |
| **O/S** | The base software controlling the computer. | Windows Server, Linux (Ubuntu). |
| **Virtualization** | Software creating virtual servers on physical hardware. | A hypervisor. |
| **Servers** | The physical computers. | Rack-mounted machines. |
| **Storage** | Disks that keep data. | Disk arrays. |
| **Networking** | Cables, switches, routers. | Connecting servers to each other and the Internet. |

---

### 8.1 IaaS: Infrastructure as a Service

**What the provider gives you:**
- Physical building, power, cooling
- Servers, networking, and basic **block storage**
- Extras: **load balancers, backup, network security**
- It **boots physical and virtualized servers** for you

**What you do:**
- **Pick the operating system, applications and firewall rules**
- Manage runtime, middleware, data

**Other notes from the slide:** Most advanced IaaS offerings **auto-scale** (add or remove servers automatically based on demand).

**Examples:**
- Renting a virtual server on **AWS EC2**, **Azure Virtual Machines** or **Google Compute Engine**
- You pick Ubuntu, install a web server, install your app, and configure the firewall yourself

**Best for:** IT teams who need **maximum control**, such as large enterprises that want IaaS plus add-on security and backup.

---

### 8.2 PaaS: Platform as a Service

**What the provider gives you:**
- A ready platform: **compilers, middleware, runtimes** (for example **Java, .NET**)
- It **hosts your applications**

**What you do:**
- Write and deploy your **application and data** only. You don't manage servers or operating systems.

**Other notes from the slide:**
- PaaS was **formerly called "Framework as a Service"**.
- **Some PaaS tools run on-premises**, behind your own firewall.

**Examples:**
- A developer writes a Java web app and uploads it to a PaaS such as **Azure App Service**, **Google App Engine** or **AWS Elastic Beanstalk**. The platform runs it, and the developer never installs an OS or patches a server.

**Best for:** Developers who want to **build** software quickly without worrying about infrastructure.

---

### 8.3 SaaS: Software as a Service

**What it is:** A **subscription** model where apps and files **live in the cloud, not on your device**. The provider manages everything.

**Examples:** Office 365 (the slide's example), Gmail, Google Docs, Dropbox, Zoom, Netflix.

**Three guarantees SaaS gives customers:**

| Guarantee | Meaning | Example |
|---|---|---|
| **Universal Access** | Reach the same apps and files from **any device, any time**, via app or browser login. | Start a Word document on your laptop at home, then open and finish it on your phone in a taxi. |
| **Guaranteed Sync** | **Only one copy** of each file exists. Changes on one device appear everywhere, with **no manual re-sync**. | You edit a spreadsheet on your laptop, and your phone immediately shows the same edit. It isn't a separate copy being updated. It *is* the one copy. |
| **High Availability** | **Backup power and off-site data backups** mean service survives outages, even disasters. | A flood hits one data center, but your email still works because another site has backups. |

**Why does editing on a laptop "just show up" on your phone?** Because both devices are only *windows* into **one master copy** stored in the cloud. Nothing needs to be copied between devices.

**Best for:** End users and small businesses who want finished tools and no IT work.

---

### 8.4 DaaS: Desktop as a Service (a special case of SaaS)

- Instead of delivering **one app**, DaaS delivers an **entire remote desktop**: **OS, apps and files all run in the cloud**.
- All of a user's computing gets **universal access, sync and high availability**, not just a single application.

**Example:** A company gives remote employees a thin laptop. When they log in, they see their normal Windows desktop with all their software and files, but it's actually running in the cloud. If the laptop is lost, nothing is lost and the employee logs in from another device and sees the same desktop.

**SaaS vs DaaS in one line:** SaaS = one application delivered. DaaS = the **whole computer desktop** delivered.

---

### 8.5 "Combining Cloud Delivery Models"
These models can be **combined**.

**Example:** A company uses **SaaS** for email, **PaaS** to build its custom customer portal, and **IaaS** to host a legacy application that needs a specific operating system, all at the same time.

### Quick decision guide

| If you want... | Choose |
|---|---|
| Full control of OS and software, and you have IT staff | **IaaS** |
| To just write and deploy code, with no infrastructure work | **PaaS** |
| Ready-to-use software, no IT work | **SaaS** |
| A complete remote desktop | **DaaS** |

---

## 9. Cloud Types: Private vs Public

### The question
*If public cloud is usually cheaper, why would anyone still build their own data center?* Let's define both first.

### 9.1 Private Cloud
> An **internal cloud, owned and operated by one organization, for its own use.**

- It applies **elastic principles internally**.
- This avoids **underutilized AND oversubscribed** servers by **spreading computing across all physical machines**, even when **different departments' demand varies**.

**Example:** A large bank builds its own cloud. The loan department is busy at month-end, the trading department is busy during market hours. Instead of each department owning separate servers (one idle while the other is overloaded), they all share one internal pool. When trading is quiet, those servers help the loan department.

**Key terms:**
- **Underutilized:** servers sit mostly idle, wasting money.
- **Oversubscribed:** more demand than capacity, so things slow down or fail.

### 9.2 Public Cloud
> A **commercial service, operated by a provider, used by many customer organizations.**

Customers choose a **service tier**:
- **Large enterprises** often want **IaaS** with add-on security and backup.
- **Builders (developers)** want **PaaS**.
- **Smaller customers** may just subscribe to a single **SaaS** offering.

**Example:** A startup uses AWS (public cloud). A large retailer uses AWS too. So does a university. Each is a customer of the same provider, isolated from each other.

### Side-by-side

| Feature | Private Cloud | Public Cloud |
|---|---|---|
| Owned by | One organization | A provider (AWS, Azure, GCP...) |
| Used by | That organization only | Many organizations |
| Who runs it | The organization's own IT team | The provider's staff |
| Typical example | Bank's internal cloud | Renting AWS servers |

---

## 10. Why Choose Public Cloud?

Three main reasons, each explained.

### 10.1 Economic
- **Economy of scale:** bigger discounts on hardware, **shared whitebox switches run by SDN**, and **shared IT staff**.
- The lecture cites savings of **up to ~75%** versus private deployment.

**New terms explained:**
- **Whitebox switches:** cheap, generic network switches without a big-brand price tag. Hyperscalers buy huge numbers of them.
- **SDN (Software Defined Networking):** the network is controlled by **software**, so cheap generic switches can be managed centrally and flexibly.

**Example:** A small company buying 10 branded switches pays a high price. A provider buying 100,000 generic switches and controlling them with software pays far less per unit and passes some of that saving on.

### 10.2 Expertise
- Access to **specialized staff across a far wider range of topics** than any single in-house IT team, including **AI/ML**.

**Example:** A mid-size company can't afford a full-time security expert, a networking expert, a database expert and a machine-learning expert. A cloud provider employs all of them, and you benefit from their knowledge.

### 10.3 Advanced Services
- **AIOps** automates operations and monitors for **anomalies** (unusual behavior).
- **Platforms** let engineers **deploy apps without learning the infrastructure underneath**.

**Example of AIOps:** The system notices a server's disk is filling abnormally fast, automatically raises an alert and may fix the problem before customers notice.

### The trade-off: Lock-in
- Providers **court lock-in**, offering **free convenience services and migration tools** that make it **costly to ever switch providers**.

> **Vendor lock-in:** you become so dependent on one provider's tools that switching becomes very expensive.

**Example:** You build your whole app using a provider's special database service. Moving to another provider means rewriting large parts of your app. You're "locked in".

---

## 11. Why Choose Private Cloud?

Public cloud isn't right for everyone. Private cloud **trades some savings for control**.

### 11.1 Control & Visibility
- **Full root-cause access** to your own switches and servers.
- Helps satisfy **regulatory requirements** on data placement and handling.

**Example:** A government defense agency, or a hospital under health-data laws, must prove exactly where data is stored and who can access it. With its own private cloud, it can inspect every switch and server. In a public cloud, it can't open the provider's hardware to investigate.

**"Root-cause access":** if something breaks, you can dig into any layer to find the **root cause**. In a public cloud, the lower layers belong to the provider.

### 11.2 Reduced Latency
- **On-premises facilities cut the delay** between employees and the data center.
- Especially useful when the **public cloud sites are geographically far away**.

> **Latency:** the delay between sending a request and getting a response.

**Example:** A factory in Gujarat needs instant responses from a control system. If the nearest public cloud region is thousands of kilometers away, the delay may be too high. A private data center in the same building has almost no delay.

### 11.3 Rate-Hike Insurance
- **No provider lock-in risk.**
- As adoption of a provider's services deepens, **switching costs rise**, and the **provider can raise prices**.

**Example:** You run everything on one provider for 5 years. They announce a 30% price increase. Because moving is very costly, you have little choice but to pay. A private cloud protects you from this, because you control your own costs.

### Final answer to "what could justify staying private?"
**Control, regulation/compliance, speed (latency), and protection from price hikes.** Cost is not the only factor.

---

## 12. Hybrid Cloud and Multi-Cloud

Choosing a cloud strategy is **not all-or-nothing**.

### 12.1 Hybrid Cloud
> A **mix**: some computing stays on a **private cloud**, the rest runs on a **public provider**.

**Two main reasons:**

**a) Control when needed**
- Keep **regulated or classified data on the private side**.
- Push the rest to public cloud to **cut cost**.

*Example:* A hospital keeps patient medical records on its private cloud (legal requirement) but hosts its public website and appointment booking on a public cloud.

**b) Computation overflow ("cloud bursting")**
- Handle **peak-season demand** by sending **overflow work** to the public cloud instead of buying idle capacity.

*Example:* A tax-filing company's own data center handles normal load. Around the filing deadline, demand spikes, so extra work overflows into the public cloud. After the deadline, the extra rented servers are released.

### 12.2 Multi-Cloud
> Using **more than one public cloud provider** to avoid dependence on any single one.

**Why do it:**
- **Avoids lock-in risk.**
- Large organizations may **split business units across providers**.

**Trade-offs:**
- **Migrating between providers is hard.**
- **Service parity varies** (the same service may not exist, or work the same way, at every provider).
- May require **custom software to translate or combine outputs** across providers.

**Example:** A multinational runs its e-commerce on AWS, its analytics on Google Cloud, and its Microsoft-based office systems on Azure. If one provider has problems or raises prices, the company is not completely stuck. But its engineers must learn three sets of tools and build software to connect them.

### Hybrid vs Multi-Cloud, the key difference

| | Hybrid Cloud | Multi-Cloud |
|---|---|---|
| What is mixed? | **Private + Public** | **Several Public providers** |
| Main goal | Control + cost + overflow capacity | Avoid lock-in, flexibility |
| Example | Private cloud + AWS | AWS + Azure + Google Cloud |

*An organization can do both at once:* private cloud + AWS + Azure = **hybrid multi-cloud**.

---

## 13. Hyperscalers: The Big Players

> **Hyperscalers** are companies that **own and operate the largest cloud computing facilities**.

The market is **more concentrated than most people expect**: a small number of companies run most of the world's public cloud.

### The big three

| Provider | Full name | Launched / note |
|---|---|---|
| **AWS** | Amazon Web Services | Launched **2006** |
| **Azure** | Microsoft Azure Cloud | Launched **2010** |
| **GCP** | Google Cloud Platform | Google's cloud offering |

### Key numbers (from the lecture, as of 2018)

| Figure | Meaning |
|---|---|
| **$11 billion per year** | Spent on new servers by **Google, Amazon, Microsoft, Facebook and Alibaba combined**. |
| **$32.2 billion** | **Microsoft cloud revenue** by 2018, which was **29%** of Microsoft's total revenue. |
| **$25.7 billion** | **Amazon AWS revenue** by 2018, which was **11%** of Amazon's total revenue. |

**Reading the numbers:**
- For Microsoft, cloud was already **almost 1/3** of the whole company.
- For Amazon, AWS was only **11%** of revenue, though still a huge number in dollars.
- *Quick check:* $32.2B ÷ 0.29 ≈ $111B total Microsoft revenue. $25.7B ÷ 0.11 ≈ $234B total Amazon revenue.

*Note:* These are 2018 figures from the lecture. The cloud market has grown a lot since. Use these numbers as given in your course material.

---

## 14. Master Comparison Cheat Sheet

### 14.1 Service models

| | IaaS | PaaS | SaaS | DaaS |
|---|---|---|---|---|
| **Provides** | Servers, storage, network | Platform to build and run apps | Finished application | Full remote desktop |
| **You manage** | OS, middleware, runtime, apps, data | Apps and data | Just your account and usage | Your files and usage |
| **Provider manages** | Hardware + virtualization | Everything up to the runtime | Everything | Everything (OS, apps, hardware) |
| **Typical user** | IT admins | Developers | End users | Remote and enterprise workforce |
| **Example** | AWS EC2 | Azure App Service | Office 365 | Cloud desktop |

### 14.2 Deployment types

| | Private | Public | Hybrid | Multi-cloud |
|---|---|---|---|---|
| **Who owns** | One org | Provider | Both | Multiple providers |
| **Cost** | Higher | Lower (up to ~75% savings cited) | Balanced | Varies |
| **Control** | Highest | Lowest | Selective | Spread across providers |
| **Lock-in risk** | None | High | Medium | Reduced |
| **Latency** | Low (local) | Depends on distance | Mixed | Mixed |
| **Best for** | Regulated, latency-sensitive | Startups, general workloads | Regulated data + peak demand | Large orgs avoiding dependence |

### 14.3 Benefits vs Trade-offs at a glance

| Option | Biggest benefit | Biggest trade-off |
|---|---|---|
| Public | Cost, expertise, advanced services | Lock-in, less control |
| Private | Control, low latency, no price-hike risk | Higher cost, need own expertise |
| Hybrid | Flexibility: control + overflow | More complexity |
| Multi-cloud | Avoids dependence on one provider | Hard migration, service differences, custom software |

---

## 15. Common Confusions Cleared

**Q1. Is "the cloud" a real place?**
No. It is real data centers (buildings with servers) accessed over the Internet. The word "cloud" came from network diagrams and the closeness of early data centers to Internet peering points.

**Q2. What's the difference between multi-tenancy and virtualization?**
- **Multi-tenancy** = *many customers share the same data center/infrastructure* (the business arrangement).
- **Virtualization** = *the technology that lets one physical machine run many isolated virtual servers* (how isolation and sharing actually work).
Virtualization is a major technology that makes safe multi-tenancy possible.

**Q3. What's the difference between elasticity and just "having lots of servers"?**
Elasticity means capacity **changes with demand**, up and down, and you pay accordingly. Merely owning many servers is not elastic, because you pay for them even when idle.

**Q4. IaaS vs PaaS: how do I tell them apart quickly?**
Ask: *"Do I need to choose and manage the operating system?"*
- Yes → **IaaS**
- No, I only upload my code → **PaaS**

**Q5. PaaS vs SaaS?**
Ask: *"Am I building the software or just using it?"*
- Building → **PaaS**
- Using → **SaaS**

**Q6. SaaS vs DaaS?**
SaaS = one application. DaaS = the **entire desktop** (OS + apps + files).

**Q7. Hybrid vs multi-cloud?**
Hybrid = **private + public**. Multi-cloud = **multiple public providers**.

**Q8. If public cloud is cheaper and has more expertise, why use private?**
Control, regulatory compliance, lower latency, and no exposure to provider price hikes or lock-in.

**Q9. What is lock-in and why is it a problem?**
Becoming so dependent on one provider's tools that leaving is very expensive. The provider can then raise prices, and you have little negotiating power.

**Q10. Does SaaS keep multiple copies of my file on each device?**
No. There is **one master copy** in the cloud. Devices only view and edit it. That's why sync is guaranteed.

---

## 16. Glossary

| Term | Meaning |
|---|---|
| **AIOps** | Using AI to automate IT operations and detect anomalies. |
| **Anomaly** | Unusual behavior that may signal a problem. |
| **Block storage** | Basic raw disk-like storage offered with IaaS. |
| **Chassis** | A box (about 7U) in which multiple servers are stored vertically. |
| **Cloud bursting / overflow** | Sending extra work to a public cloud when private capacity is exceeded. |
| **DaaS** | Desktop as a Service: a full remote desktop from the cloud. |
| **Data center** | A dedicated physical facility housing applications, data and IT infrastructure. |
| **DG** | Diesel Generator, long-duration backup power. |
| **Elastic computing** | Leasing only what you need and changing it at any time. |
| **Hybrid cloud** | A mix of private and public cloud. |
| **Hyperscaler** | A company operating the very largest cloud facilities (AWS, Azure, GCP). |
| **IaaS** | Infrastructure as a Service: servers, storage and networking you configure yourself. |
| **Isolation** | Guarantee that one tenant cannot see or affect another's data or computations. |
| **Latency** | Delay between a request and its response. |
| **Load balancer** | A device or software that spreads incoming traffic across servers. |
| **Lock-in** | Dependence on one provider that makes switching costly. |
| **Middleware** | Software between the OS and your application that helps them work together. |
| **Multi-cloud** | Using more than one public cloud provider. |
| **Multi-tenant** | Serving many customers or organizations from the same infrastructure. |
| **On-premises** | Hardware and software run in your own facility. |
| **Oversubscribed** | Demand exceeds available capacity. |
| **PaaS** | Platform as a Service: build and deploy apps without managing infrastructure. |
| **Peering point** | A place where Internet backbones connect to each other. |
| **Private cloud** | A cloud owned and operated by one organization for itself. |
| **Public cloud** | A commercial cloud service used by many organizations. |
| **Rack / 42U** | A frame holding equipment. 42U means 42 rack units of space. |
| **SaaS** | Software as a Service: subscribe to a finished application. |
| **SDN** | Software Defined Networking: network controlled by software. |
| **Tenant** | A customer or organization using a shared cloud. |
| **U (rack unit)** | Unit of rack height, equal to 1.75 inches. |
| **Underutilized** | Resources sitting idle and wasted. |
| **UPS** | Uninterruptible Power Supply: instant battery backup. |
| **Virtualization** | Software that creates virtual servers on shared physical hardware. |
| **Virtualized server (VM)** | A software-based server running on a physical machine. |
| **Whitebox switch** | A low-cost generic network switch. |

---

## 17. Self-Test (with Answers)

### Short-answer questions

**1. Define a data center and list five of its components.**
*Answer:* A dedicated physical facility housing an organization's critical applications, data and IT infrastructure. Components: server racks, servers, storage, networking devices, cooling, power backup (DG/UPS), fire detection and fighting equipment, fireproof walls, anti-rodent measures, centralized monitoring (any five).

**2. How many 2U servers fit in a 42U rack?**
*Answer:* 42 ÷ 2 = **21**.

**3. Why is the phrase "in the cloud" technically inaccurate?**
*Answer:* Data center servers are not part of the Internet itself. They are computers attached to it, drawn outside the cloud in diagrams. The phrase came from early data centers being physically next to Internet peering points.

**4. What is multi-tenancy?**
*Answer:* A data center serving customers from many organizations at once (or separating different groups inside one organization).

**5. Explain elastic computing with an example.**
*Answer:* Leasing only the capacity you need and changing it at any time. Example: a ticketing site scales from 4 to 80 servers for a booking rush, then back to 4, paying only for what it used.

**6. List the three properties of virtualized servers.**
*Answer:* Rapid creation and removal, physical sharing, logical isolation.

**7. Give two benefits of virtualization to the provider and two to the customer.**
*Answer:* Provider: creation in milliseconds; no reboot or reconfiguration of physical machines; safe mixing of tenants; load balancing (any two). Customer: acts like a physical server with its own Internet address; easy deployment with vendor tooling; rapid scaling; safe isolated testing (any two).

**8. Which layers does the customer manage in IaaS? In PaaS? In SaaS?**
*Answer:* IaaS: applications, data, runtime, middleware, OS. PaaS: applications and data. SaaS: none (just uses the software).

**9. What three guarantees does SaaS give customers?**
*Answer:* Universal access, guaranteed sync, high availability.

**10. How is DaaS different from regular SaaS?**
*Answer:* SaaS delivers a single application. DaaS delivers an entire remote desktop (OS, apps and files).

**11. Give three reasons for choosing public cloud and one trade-off.**
*Answer:* Economic (economy of scale, up to ~75% savings cited), expertise (specialized staff including AI/ML), advanced services (AIOps, platforms). Trade-off: provider lock-in.

**12. Give three reasons for choosing private cloud.**
*Answer:* Control and visibility (including regulatory needs), reduced latency, protection against rate hikes (no lock-in).

**13. What is the difference between hybrid and multi-cloud?**
*Answer:* Hybrid = private + public. Multi-cloud = multiple public providers.

**14. Name the three major hyperscalers and when AWS and Azure launched.**
*Answer:* AWS, Azure, Google Cloud Platform. AWS launched in 2006, Azure in 2010.

### Scenario questions

**S1.** A startup wants to deploy a Java app quickly without managing servers or operating systems. Which model?
*Answer:* **PaaS.**

**S2.** A company needs full control over the operating system and firewall rules for its servers. Which model?
*Answer:* **IaaS.**

**S3.** A hospital must keep patient data on-premises by law but wants to use the cloud for its public website. Which strategy?
*Answer:* **Hybrid cloud**: private for regulated data, public for the website.

**S4.** A large company worries that depending on one provider will let that provider raise prices. What strategy?
*Answer:* **Multi-cloud** (to avoid lock-in) and/or keeping some workloads in a **private cloud**.

**S5.** A retail company is busy only during festival season. How does the cloud help?
*Answer:* **Elastic computing**: lease large capacity during the 5 peak months and pay little or nothing the rest of the year. If it also owns a private data center, it can use **hybrid overflow** to the public cloud during peaks.

**S6.** A manufacturer's control system needs near-instant response times. Which cloud type and why?
*Answer:* **Private (on-premises)**, because of **reduced latency**.

---

## 18. One-Page Revision Summary

1. **Cloud = renting computing over the Internet, paying as you use.** Use cases: startups, smart devices, enterprise software, shared documents, medical alerts, seasonal demand, social media.
2. **Data center** = the physical building (racks, servers, storage, network, cooling, UPS/DG, fire safety, monitoring). **1U = 1.75 in; a 42U rack is standard.**
3. **"Cloud" is a historical name**: data centers sat near Internet peering points, and the phrase stuck, though technically inaccurate.
4. **Multi-tenant** = many organizations share one data center, giving lower cost via quantity discounts and shared staff.
5. **Elastic computing** = lease only what you need, and change it any time; pay for what you use. Early version: rack, half-rack, quarter-rack.
6. **Virtualization** makes elasticity possible with three properties:
   - Rapid creation and removal
   - Physical sharing
   - Logical isolation
7. **Benefits:**
   - Provider: milliseconds creation, no reboot, safe tenant mixing, load balancing.
   - Customer: like a physical server with its own address, easy deployment, rapid scaling, safe testing.
8. **Service models** (more you manage → more provider manages):
   - **IaaS:** you manage OS and up (AWS EC2).
   - **PaaS:** you manage apps and data (build and deploy code).
   - **SaaS:** provider manages everything (Office 365). Guarantees: universal access, guaranteed sync, high availability.
   - **DaaS:** a whole remote desktop.
9. **Private cloud:** one org, internal, avoids idle and overloaded servers. **Public cloud:** a provider serves many orgs.
10. **Why public:** economic (~75% savings cited), expertise, advanced services (AIOps). **Trade-off:** lock-in.
11. **Why private:** control and visibility (regulation), reduced latency, rate-hike insurance.
12. **Hybrid:** private + public (control + overflow). **Multi-cloud:** several public providers (avoid lock-in, but migration is hard and service parity varies).
13. **Hyperscalers:** AWS (2006), Azure (2010), GCP. 2018: $11B/yr on new servers (Google, Amazon, Microsoft, Facebook, Alibaba), Microsoft cloud $32.2B (29% of revenue), AWS $25.7B (11% of revenue).

---

*End of Handbook*
