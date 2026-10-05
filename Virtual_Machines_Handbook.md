# Virtual Machines in Cloud Computing Handbook
### Virtualization Mechanisms, Hypervisors, and Processor Privilege Modes
*Based on: IT457 Cloud Computing, Week 5 – Lecture 2 (Chapter 5)*
*Updated edition: now includes the deep-dive explanations from our discussion (Sections 11 to 15)*

---

## How to Use This Handbook

Every section follows the same pattern:

1. **The idea**: what the concept is, in plain words
2. **Example / analogy**: something concrete so it sticks
3. **Common confusion**: the usual mix-ups, cleared

At the end: a **comparison cheat sheet**, a **glossary**, a **self-test with answers**, and a **one-page revision summary**.

> **Where this fits:** Lecture 1 told you *that* cloud providers run virtualized servers. This lecture explains *how* virtualization actually works inside the machine.

---

## Table of Contents

**Part A: Lecture Content**
1. [What Is a Virtual Machine?](#1-what-is-a-virtual-machine)
2. [The Hypervisor](#2-the-hypervisor)
3. [Three Approaches to Virtualization](#3-three-approaches-to-virtualization)
4. [Software Emulation vs Para-Virtualization (Trade-offs)](#4-software-emulation-vs-para-virtualization-trade-offs)
5. [Full Virtualization: Key Properties](#5-full-virtualization-key-properties)
6. [The Big Question: How Can a VM Run at Hardware Speed?](#6-the-big-question-how-can-a-vm-run-at-hardware-speed)
7. [Processor Privilege: Two Modes Became Three](#7-processor-privilege-two-modes-became-three)
8. [Extending to the Hypervisor (The Diagram Explained)](#8-extending-to-the-hypervisor-the-diagram-explained)
9. [Execution and State Transitions (4 Steps)](#9-execution-and-state-transitions-4-steps)
10. [Hierarchy of Trust](#10-hierarchy-of-trust)

**Part B: Deep-Dive Add-ons (from our doubt-clearing discussion)**
11. [Hypervisor from Scratch: Why It Exists and Where It Sits](#11-hypervisor-from-scratch-why-it-exists-and-where-it-sits)
12. [Full vs Para-Virtualization: The Deep Dive](#12-full-vs-para-virtualization-the-deep-dive)
13. [Why Para-Virtualization Gives Faster I/O](#13-why-para-virtualization-gives-faster-io)
14. [CPU Rings and Hardware-Assisted Virtualization](#14-cpu-rings-and-hardware-assisted-virtualization)
15. [Walkthrough: What Happens When a File Upload Arrives in a VM](#15-walkthrough-what-happens-when-a-file-upload-arrives-in-a-vm)

**Part C: Revision Tools**
16. [Master Comparison Cheat Sheet](#16-master-comparison-cheat-sheet)
17. [Common Confusions Cleared](#17-common-confusions-cleared)
18. [Glossary](#18-glossary)
19. [Self-Test (with Answers)](#19-self-test-with-answers)
20. [One-Page Revision Summary](#20-one-page-revision-summary)

> **Note on sources:** Sections 1 to 10 follow your lecture slides. Sections 11 to 15 go deeper to clear the doubts you raised. Anything that goes beyond the slides is marked *(beyond the slides)*. For exams, always prefer your slide wording.

---

## 1. What Is a Virtual Machine?

### Definition (from the slide)
> A **virtual machine (VM)** is a computer system created using **software** on **one physical computer** in order to **emulate the functionality of another separate physical computer**.

In simple words: a VM is a **"computer inside a computer"**. It looks and behaves like a real, separate machine, but it is really just software running on shared hardware.

### The three properties listed on the slide

| Property | Meaning | Example |
|---|---|---|
| **VMs cannot interfere with the host or other VMs** | A VM is sealed off. It cannot damage or spy on the physical machine (the *host*) or on neighboring VMs. | If VM-A is hit by a virus, VM-B on the same server stays safe. |
| **They run their own kernel** | Each VM has its own full operating system core (the *kernel*), independent of other VMs. | One VM runs Linux, the next runs Windows, both on one physical server. |
| **Managed by a hypervisor** | A special piece of software creates, runs and controls VMs. | Covered in the next section. |

### VMs are the primary service of cloud providers
The slide lists the three famous examples:

| Provider | VM service |
|---|---|
| Amazon | **AWS Elastic Compute Cloud (EC2)** |
| Google | **Google Compute Engine** |
| Microsoft | **Azure VMs** |

*Link to Lecture 1:* These are **IaaS** products. When you "rent a server" from AWS, you are actually renting a VM.

### Analogy
Think of a **large apartment building**:
- The **building** = the physical server
- Each **flat** = a VM
- The **building manager** = the hypervisor
- Each tenant can decorate their flat their own way (own OS), but cannot enter a neighbor's flat (isolation)

### Common confusion
**"Is a VM a real computer?"** No. It *emulates* one. To the software running inside, it feels real, but there is no dedicated physical machine behind it.

---

## 2. The Hypervisor

### Definition
> A **hypervisor** is the software used for **creating and managing VMs**. It **controls the underlying hardware**.

### What the slide says, point by point

**Left box: what a hypervisor lets you do**

| Slide point | Meaning | Example |
|---|---|---|
| **Load software onto a server** | You install the hypervisor on a physical server first. | An admin installs hypervisor software on a new rack server. |
| **Allows creating many VMs** | Once running, one server can host many VMs. | One server hosts 20 VMs at once. |
| **Allows the tenant (owner of VM) to boot an OS on the VM** | The customer who owns the VM chooses and starts their own operating system. | A customer picks Ubuntu and boots it in their VM. |
| **Use the OS to launch apps** | The customer's OS then runs their applications. | The customer launches a web server and database in the VM. |

**Right box: what a hypervisor is responsible for**
- It is the software used for creating and managing VMs.
- It **controls the underlying hardware** (CPU, memory, devices).
- **Each VM it creates is independent of the other VMs.**

### The layered diagram on the slide

```
+---------------------------------------------------------+
|  VM 1            |  VM 2            |  ...  |  VM N     |
| +-----+-----+-----+ +-----+-----+-----+       +---+---+---+
| |app 1|app 2|app 3| |app 4|app 5|app 6|       |app7|app8|app9|
| +-----------------+ +-----------------+       +-----------+
| |      OS 1       | |      OS 2       |       |   OS N    |
+---------------------------------------------------------+
|              hypervisor software                        |
+---------------------------------------------------------+
|                   server hardware                       |
+---------------------------------------------------------+
```

**Reading the diagram from the bottom up:**
1. **Server hardware** is the real physical machine at the bottom.
2. **Hypervisor software** sits directly on the hardware and controls it.
3. On top, many **VMs** run side by side. Each has its **own OS** (OS 1, OS 2 ... OS N).
4. Inside each VM, the tenant's **apps** run on that VM's OS.

### Worked example
A cloud provider has one physical server. It installs a hypervisor. Then:
- **Customer A** creates VM 1, boots Linux (OS 1), and runs 3 apps (website, database, cache).
- **Customer B** creates VM 2, boots Windows (OS 2), and runs 3 apps (accounting software etc.).
- **Customer N** creates VM N with another OS.

All three share the same physical hardware, but none can see or affect the others. The hypervisor makes that possible.

### Common confusion
**"Is the hypervisor the same as the OS?"** No. The hypervisor sits *below* the guest operating systems. It is the manager of the VMs. Each VM then has its *own* OS on top.

---

## 3. Three Approaches to Virtualization

There are three ways to build "a computer inside a computer".

### 3.1 Software Emulation
- **Translates hypothetical instructions via byte-code interpreters.**
- **Provides portability across hardware** (the same program can run on very different machines).
- **But introduces high execution overhead** (it is slow).

**How it works, in plain words:** Programs are not written for the real CPU. They are written for an imaginary ("hypothetical") computer. A translator (interpreter) converts each imaginary instruction into real CPU instructions *while the program runs*. That translation costs time.

**Example:** The **Java Virtual Machine (JVM)**. You write Java once, it compiles to byte code, and the JVM interprets it on Windows, Linux or Mac. Same program, many machines. But the translation step makes it slower than code built directly for one CPU.

**Analogy:** A live interpreter at a conference. Anyone who speaks any language can be understood by anyone (portability), but everything takes longer because each sentence is translated.

---

### 3.2 Para-Virtualization
- Allows **multiple OSes to run on a single PC/server**.
- **Replaces privileged OS instructions with hypervisor calls.**
- Offers **near-native execution speed**, **but requires modified operating system code.**

**How it works, in plain words:** Some instructions are *privileged*: only the OS is allowed to run them because they touch hardware directly (for example, controlling devices or memory). In para-virtualization, the guest OS is **rewritten** so that, instead of executing those dangerous instructions itself, it **politely asks the hypervisor** to do it (a *hypervisor call*, also called a *hypercall* or *trap*).

**Example:** A guest OS wants to write to a disk. In a normal OS, it talks to the disk hardware directly. In a para-virtualized OS, its source code was changed so it sends a request to the hypervisor ("please write this to disk for me"), and the hypervisor performs it safely.

**Analogy:** Employees in a company are not allowed to open the safe. Instead of opening it themselves, they must submit a request to the manager, who opens it for them. The company has to retrain all employees (modify the OS) to follow this rule.

---

### 3.3 Full Virtualization
- **Emulates commercial instruction sets seamlessly.**
- **Allows unmodified guest operating systems to execute directly at hardware speed, with isolation.**

**How it works, in plain words:** The VM looks *exactly* like a real physical computer. Any standard operating system (Windows, Linux) can be installed **without changes**, and it runs at nearly full speed because most instructions run directly on the real CPU. The hypervisor only steps in when something privileged happens.

**Example:** You install an ordinary, unmodified copy of Ubuntu in an AWS EC2 VM. Ubuntu has no idea it is virtual. It just works.

**Analogy:** A perfect stage set. The actors (guest OS) behave as if it is a real house and never need special training, but the stage crew (hypervisor) quietly makes sure nobody wanders into areas they shouldn't.

---

### 3.4 Side-by-side summary

| | Software Emulation | Para-virtualization | Full Virtualization |
|---|---|---|---|
| **Core idea** | Interpret instructions of an imaginary machine | Guest OS asks the hypervisor for privileged actions | VM looks exactly like real hardware |
| **Speed** | **Slow** (high overhead) | **Near-native** | **Hardware speed** |
| **Guest OS modified?** | Not applicable (programs are written for a virtual machine) | **Yes**, OS code must be changed | **No**, unmodified OS works |
| **Portability** | **High** (across different CPUs) | Limited to modified OSes | Works with standard commercial OSes |
| **Example** | Java Virtual Machine | Early hypervisor systems | Cloud VMs (EC2, Compute Engine, Azure VMs) |

---

## 4. Software Emulation vs Para-Virtualization (Trade-offs)

### 4.1 Software Emulation Trade-offs

- Uses **byte-code compilers and interpreters** (like the **Java Virtual Machine**) to run software **across heterogeneous CPUs** (different types of CPUs).
- **Major drawback:** a **severe performance penalty** from software translation.
- Because of that, it is **ineligible as a primary virtualization form in cloud data centers.**

**Why this matters:** Cloud providers sell computing power. If every instruction had to be translated, customers would get a fraction of the speed they pay for.

**Example of the penalty:** Imagine a program needing 1 billion instructions. On real hardware, each runs directly. With emulation, *each* of those instructions may require many extra instructions just to translate it. The program takes much longer. A customer paying for a fast server would not accept that.

### 4.2 Para-Virtualization Trade-offs

- **Pioneered in the 1960s:** a hypervisor **moves processors among multiple guest OS instances directly on hardware.**
- **Major drawback:** it **requires altering the guest operating system source code** to replace **I/O and privileged calls with hypervisor traps.**

**Why this matters:** Speed is good, but you can only run operating systems that someone has modified. You cannot simply install an ordinary off-the-shelf Windows. Modifying an OS requires access to its **source code**, which commercial vendors often do not share, and it must be redone for every OS and every new version.

**Example:** A company wants to run a standard commercial OS. It cannot, because the OS has not been rewritten for the hypervisor. That limits customer choice.

### 4.3 Why Full Virtualization won in the cloud

| Requirement of a cloud provider | Emulation | Para-virtualization | Full virtualization |
|---|---|---|---|
| Fast (near hardware speed) | ❌ | ✅ | ✅ |
| Customers can run standard, unmodified OSes | n/a | ❌ | ✅ |
| Strong isolation | ✅ | ✅ | ✅ |

Full virtualization is the only approach that satisfies **both** speed and unmodified operating systems, which is why it is the production standard for cloud VMs.

---

## 5. Full Virtualization: Key Properties

Section 02 of the lecture: *"Key properties and system organization of production cloud virtual machines."*

### Property 1: Commercial Instruction Set Emulation
> To the guest tenant, a VM is **identical to physical hardware**. Standard operating systems like **Linux and Windows boot unaltered.**

- **Instruction set** = the list of commands a CPU understands (for example, the x86 family used in most servers).
- "Commercial" means the standard, widely used CPU instruction sets.
- The VM presents the *same* instruction set as a real machine, so existing software needs no changes.

**Example:** A customer takes a Windows Server installation disk and boots it in a VM. It installs exactly as on a physical machine.

### Property 2: Isolated Facilities & Operation
> **Complete independence between VMs** on the same server. Each guest OS acts as if it has **exclusive control of memory, devices, and CPU cores.**

- Every guest OS *believes* it owns the whole machine.
- In reality, the hypervisor gives each VM its own slice and keeps the slices separate.

**Example:** VM-A and VM-B both think "address 1000 in memory is mine". The hypervisor secretly maps VM-A's "1000" and VM-B's "1000" to completely different real locations, so neither can reach the other's data.

**Analogy:** Hotel rooms. Every guest feels the room is theirs, though the hotel owns the whole building.

### Property 3: Efficient Low-Overhead Execution
> **Almost all non-privileged instructions execute directly on physical CPUs at full native hardware speed.**

- **Non-privileged instructions** = ordinary things like arithmetic, comparisons, moving data inside your own memory.
- These are run **directly** by the real CPU. No translation, no middleman.
- Only the small number of **privileged** instructions need the hypervisor's attention.

**Example:** A VM adding two numbers or sorting a list runs on the real CPU at full speed. Only when it tries to, say, access a physical device does the hypervisor step in.

### Summary

| Property | In one line | Benefit |
|---|---|---|
| Commercial instruction set emulation | Looks like real hardware | Standard OSes run unmodified |
| Isolated facilities & operation | Each VM believes it owns the machine | Security and independence |
| Efficient low-overhead execution | Most instructions run natively | Near-hardware performance |

---

## 6. The Big Question: How Can a VM Run at Hardware Speed?

The slide asks: **"How can an application running in a VM operate at hardware speed?"**

To answer, first understand how a *normal* computer (without a hypervisor) runs code.

### The traditional two-mode picture (from the diagram)

```
 OS starts first
      |
      v
 +----------------------+---------------------+
 |  operating system    |      app code       |
 |  code                |                     |
 +----------------------+---------------------+
   executed in KERNEL mode   executed in USER mode
```

**Reading the diagram:**
1. The **OS starts first** (when the computer boots).
2. The **OS starts an app**, and control moves to the app code.
3. The **app invokes an OS service** (for example, "save this file"), and control goes back to the OS code.

### Memory holds two kinds of code
| Code | Runs in | Power level |
|---|---|---|
| **Operating system code** | **Kernel mode** | High: can do anything on the hardware |
| **App code** | **User mode** | Low: restricted, cannot touch hardware directly |

### Why this gives speed
The CPU itself knows which mode it is in. In **user mode**, ordinary instructions simply run **directly on the CPU at full speed**. The CPU only interrupts when the app tries something it isn't allowed to do (like touching a device or another program's memory).

**Example:** Your calculator app adds numbers in user mode at full CPU speed. When you press "Save", the app asks the OS to write to disk. That request (a *system call*) switches the CPU to kernel mode, the OS does the job, then control returns to the app.

### The key insight for VMs
The **same hardware mechanism** is extended to cover the hypervisor. That way, apps inside a VM still run directly on the CPU in user mode. No translation is needed. That is the secret of hardware speed, explained in the next sections.

---

## 7. Processor Privilege: Two Modes Became Three

Section 03 of the lecture: *"How hardware execution modes maintain speed and system protection."*

### Traditional vs virtualization-ready CPUs

| | Traditional OS | Cloud virtualization |
|---|---|---|
| Number of hardware modes | **2** (Kernel mode, User mode) | **3** (Hypervisor mode, Kernel mode, User mode) |

Cloud virtualization **extends CPU hardware support to 3 distinct privilege levels**.

### The three privilege levels

```
 HIGHEST privilege    ┌────────────────────────┐
                      │ 1. HYPERVISOR MODE     │  runs the hypervisor
                      ├────────────────────────┤
                      │ 2. KERNEL MODE         │  runs each guest OS
                      ├────────────────────────┤
 LOWEST privilege     │ 3. USER MODE           │  runs tenant applications
                      └────────────────────────┘
```

### 7.1 Hypervisor Mode (highest privilege)
- **Manages host resources and creates VMs.**
- The hypervisor can **create a VM and allocate memory to the VM.**

**Example:** A customer clicks "launch VM with 8 GB RAM". The hypervisor, running in hypervisor mode, carves out 8 GB of real memory and assigns it to that new VM.

### 7.2 Kernel Mode (restricted privilege)
- The **guest OS manages its own memory and apps.**
- The OS is **restricted to the memory allocated to its VM.**
- The OS **can run apps in the memory it has been allocated.**

**Why "restricted"?** In a normal computer, kernel mode is the top level. Inside a VM, kernel mode is now *second* level. The guest OS still feels powerful, but its power is limited to its own VM's boundaries.

**Example:** The guest Linux in VM-A can freely manage the 8 GB it was given, starting programs, switching between them and so on. But it cannot touch even one byte of VM-B's memory.

### 7.3 User Mode (basic privilege)
- **Tenant applications execute natively.**

"Natively" means directly on the real CPU, at full hardware speed, which answers the "hardware speed" question.

**Example:** The customer's website code, database and so on, all run in user mode at full speed.

### The privilege ladder in one table

| Level | Who runs here | Power | Can it touch other VMs? |
|---|---|---|---|
| **Hypervisor mode** | Hypervisor | Highest, controls all hardware and creates VMs | Yes, it manages all of them |
| **Kernel mode** | Guest OS | Restricted to its own VM's memory | **No** |
| **User mode** | Tenant apps | Basic, runs natively | **No** |

### Analogy: a company
- **Hypervisor mode = the building owner.** Decides which floor each company gets.
- **Kernel mode = a company's manager.** Runs everything within their own floor but cannot enter other floors.
- **User mode = an employee.** Does daily work inside the rules set by the manager.

---

## 8. Extending to the Hypervisor (The Diagram Explained)

This is the earlier two-mode diagram, now with the hypervisor added.

```
 hypervisor starts first
        |
        v
 +--------------------+----------------------+---------------+
 |  hypervisor code   | operating system code|   app code    |
 +--------------------+----------------------+---------------+
   hypervisor mode         kernel mode           user mode
```

### Following the arrows, step by step

| Arrow in the diagram | What happens | Mode change |
|---|---|---|
| **"hypervisor starts first"** | When the server boots, the hypervisor starts before anything else. | Begins in hypervisor mode |
| **"hypervisor creates a VM and starts an OS"** | The hypervisor sets up a VM and boots a guest OS inside it. | Hypervisor mode → kernel mode |
| **"OS starts an app"** | The guest OS launches a tenant application. | Kernel mode → user mode |
| **"app invokes an OS service"** | The app asks the OS to do something privileged (like saving a file). | User mode → kernel mode |
| **"OS exits"** | The guest OS finishes (shut down or needs hypervisor help), so control returns to the hypervisor. | Kernel mode → hypervisor mode |

### Compare with the two-mode diagram
- **Before:** OS (kernel mode) ↔ App (user mode)
- **After:** Hypervisor (hypervisor mode) ↔ OS (kernel mode) ↔ App (user mode)

One extra layer was added **on the same hardware mechanism**, so speed is preserved.

### Worked example: a full journey
1. The server powers on. The **hypervisor** starts (hypervisor mode).
2. A customer requests a VM. The hypervisor creates it and **boots Linux** in it (switch to kernel mode).
3. Linux **starts a web server app** (switch to user mode). The web server now runs at full speed.
4. A visitor uploads a file. The web server **asks Linux to save it** (back to kernel mode).
5. Linux does the work. If it needs something only the hypervisor may do (like talking to the real physical disk), control passes to the **hypervisor** (hypervisor mode).
6. When done, control flows back down to the OS and the app.

---

## 9. Execution and State Transitions (4 Steps)

The slide gives a 4-step timeline. Here is each step explained.

### Step 1: Hypervisor Boot
> Hypervisor runs first in **hypervisor mode** to **allocate memory and configure host hardware.**

- It is the first software to run on the server.
- It sets up the hardware and prepares memory that will later be handed out to VMs.

**Example:** Like a building owner marking out the floors and offices before any tenant moves in.

### Step 2: Guest OS Start
> Hypervisor boots the VM's guest OS **into Kernel Mode** within **isolated memory boundaries.**

- The guest OS starts in kernel mode, but only inside its own sealed region of memory.
- It cannot see beyond its boundaries.

**Example:** A tenant moves into floor 3 and starts running their office. They are in charge of floor 3 only.

### Step 3: App Launch
> Guest OS launches app code in **User Mode.** The application executes at **full hardware speed.**

- Apps run directly on the real CPU with no translation, so there is no performance penalty.

**Example:** An employee on floor 3 does normal desk work without needing anyone's permission for every action.

### Step 4: System Call Trap
> An app service call triggers a **hardware trap**, switching the CPU **back to Kernel or Hypervisor mode.**

- When the app needs a privileged action, the **CPU hardware itself** detects it and automatically switches to a higher-privilege mode.
- The switch goes to **kernel mode** (to the guest OS) or **hypervisor mode** (if the hypervisor must handle it).

**Example:** The employee needs to enter the server room (a privileged area). A built-in lock stops them and automatically calls the manager (kernel) or building owner (hypervisor) to decide what to do.

### What is a "trap"?
A **trap** is an automatic, hardware-triggered switch of control to a more privileged layer. It happens when:
- an app makes a **system call** (a polite request for an OS service), or
- software attempts something it is **not allowed** to do (an illegal request).

### Timeline in one view

```
 1. Hypervisor      2. Guest OS       3. App          4. System call
    boot               start             launch          trap
 [hypervisor mode]  [kernel mode]     [user mode]     [back to kernel
  sets up memory     OS boots in      app runs at      or hypervisor
  and hardware       its own memory   full speed       mode]
```

---

## 10. Hierarchy of Trust

Not all software is equally trusted. The CPU hardware enforces who is allowed to do what.

### 10.1 Hypervisor Trust Level
> **Trusted completely.** Manages host CPU cores, memory boundaries, and physical hardware interfaces.
> Must **correctly configure CPU hardware isolation mechanisms** so **no guest OS can breach its VM container or inspect adjacent memory.**

- The hypervisor is the **most trusted** software on the server.
- If the hypervisor makes a mistake in configuring boundaries, a VM could escape and read another customer's data. So it must be **correct and secure**.

**Example:** A building owner has the master key. If they accidentally leave a door between two companies unlocked, one company could walk into the other. That is why the owner must be trustworthy and careful.

### 10.2 Guest OS & App Sandboxing
> Operating systems and user applications require **lower trust levels**, **enforceably restricted by CPU hardware modes.**
> If an app attempts **unauthorized memory access or privileged commands**, **hardware automatically traps execution back to the governing layer.**

- **Sandboxing** = confining software into a restricted area where it cannot harm anything outside.
- The restriction is enforced by **hardware**, not just by good behavior, so even a malicious app cannot simply ignore it.
- The "governing layer" is the guest OS (for user apps) or the hypervisor (for guest OS misbehavior).

**Example:** A malicious app in VM-A tries to read memory belonging to VM-B. The CPU hardware detects the illegal access instantly and traps control to the hypervisor, which blocks it. The attack fails.

### Trust ladder

| Layer | Trust level | Enforced by |
|---|---|---|
| Hypervisor | **Completely trusted** | Runs in the most privileged mode |
| Guest OS | Lower, restricted to its VM | CPU hardware modes plus hypervisor configuration |
| User apps | Lowest | CPU hardware modes plus guest OS |

### Why it matters
This is the reason multi-tenancy (from Lecture 1) is safe. Customers from different organizations can share one server **because hardware-enforced trust levels keep them apart.** Notice that the overall security of every VM depends heavily on the hypervisor being correct.

---

## 11. Hypervisor from Scratch: Why It Exists and Where It Sits

### 11.1 Why a hypervisor is needed at all

A normal operating system (Windows or Linux) is built on one assumption: **"I am the only boss of this machine."** It expects to control all the memory, the CPU, the disk and the network card.

Now put two operating systems on one server. Both believe they own everything, so they clash:

```
OS 1: "Memory 0-8 GB is mine!"          OS 2: "No, memory 0-8 GB is mine!"
OS 1: "I'll write to disk block 5."     OS 2: "I'll also write to disk block 5."
```

Someone must stand in the middle and keep order. That someone is the **hypervisor**.

### 11.2 The three jobs of a hypervisor

| Job | Meaning | Example |
|---|---|---|
| **Divide** the hardware | Give each VM its own share of CPU and memory | VM 1 gets 8 GB RAM and 2 CPUs. VM 2 gets 16 GB and 4 CPUs. |
| **Isolate** the VMs | One VM can never touch another VM's memory or data | VM 1 cannot read VM 2's files. |
| **Control the real hardware** | Only the hypervisor talks to the physical disk, network card and so on | Every disk write from a VM is carried out by the hypervisor. |

### 11.3 Exact location: below all operating systems

In the cloud, the hypervisor sits **directly on the server hardware and below every operating system**. There is **no "main OS" underneath it**.

```
  VM 1 (guest OS + apps)    VM 2 (guest OS + apps)    VM N
  ───────────────────────────────────────────────────────────
                      HYPERVISOR   <-- the real boss
  ───────────────────────────────────────────────────────────
                      SERVER HARDWARE
```

- The hypervisor **boots first** (your slide: "hypervisor starts first").
- Every OS you see (Linux, Windows) is a **guest OS** running **above** it, inside a VM.
- In CPU terms, the hypervisor runs in **hypervisor mode**, the highest privilege level, above kernel mode.

### 11.4 Type 1 and Type 2 hypervisors *(beyond the slides)*

| | Type 1 (bare-metal) | Type 2 (hosted) |
|---|---|---|
| **Runs where?** | Directly on hardware, **below** all OSes | **On top of** a normal host OS, like an ordinary program |
| **Where used?** | **Cloud data centers** (this is what your lecture describes) | Laptops and desktops for learning and testing |
| **Examples** | VMware ESXi, Xen, Microsoft Hyper-V | Oracle VirtualBox, VMware Workstation |
| **Performance** | Better (no extra OS in between) | Lower (host OS sits in the middle) |

**Example:** When you install VirtualBox on your Windows laptop and run Ubuntu inside it, VirtualBox sits **above** Windows. That is a Type 2 hypervisor. AWS does not do that. It runs the hypervisor straight on the server, with no Windows or Linux underneath.

### 11.5 The real difficulty that leads to full and para virtualization

Guest operating systems **still think they are the boss**. They will try to run **privileged instructions** (commands that control hardware, such as "write to disk" or "change memory mapping").

If a guest OS ran those directly on real hardware, it could break isolation. So the hypervisor must make sure **every privileged action goes through it**.

There are two ways to solve this:

- **Full virtualization:** catch the guest when it tries something privileged.
- **Para-virtualization:** teach the guest to ask the hypervisor instead.

The next section explains both properly.

---

## 12. Full vs Para-Virtualization: The Deep Dive

### 12.1 The core difference in one line

Both let several operating systems share one machine. The difference is **whether the guest OS knows it is virtual**.

- **Full = "You don't know you're in a VM, and I will handle it."**
- **Para = "You know you're in a VM, so ask me."**

### 12.2 Full virtualization = "fool the guest OS"

**Idea:** Do not change the OS. Make the VM look **exactly like a real computer**. The guest OS keeps believing it is the boss and behaves normally. When it tries a privileged action, the hypervisor **catches it** and handles it quietly.

**Example: a guest OS saves a file to disk**

1. The guest OS thinks: "I'll send a command to the disk controller" (it believes the disk is real).
2. The CPU notices a privileged action and automatically **traps**: the VM pauses and control jumps to the hypervisor.
3. The hypervisor works out what was intended: "This is a disk write from VM 1. I'll do it safely in VM 1's own area."
4. The hypervisor writes to the real disk.
5. The hypervisor gives control back. The guest OS never knew anything happened.

**Analogy:** Actors on a stage set. They believe it is a real house and need no special instructions. The stage crew (hypervisor) quietly steps in whenever someone is about to do something dangerous.

**Result:**
- The guest OS is **completely unmodified**, so Windows, Linux and others just work.
- Normal instructions (arithmetic and so on) run **directly** on the real CPU at full speed.
- Only privileged actions cause a switch to the hypervisor.

### 12.3 Para-virtualization = "tell the guest OS the truth"

**Idea:** Modify the guest OS so it **knows it is virtual**. Instead of attempting privileged actions and getting caught, it **asks the hypervisor directly** using special calls (hypervisor calls, or *hypercalls*).

**Example: the same file save**

1. The guest OS knows: "I'm in a VM. I must not touch the disk myself."
2. It sends a direct request: **"Hypervisor, please write this to disk."**
3. The hypervisor writes to the real disk.
4. The hypervisor replies "done."

**Analogy:** Actors who know it is a stage and have been trained: "If you need something dangerous, ask the director." Nobody needs to catch them, because they cooperate.

**Result:**
- Less catching and guessing, so it is **efficient**, especially for disk and network work.
- But the OS's **source code must be changed**, so only OSes that someone has modified can run.

### 12.4 Same task, side by side ("save a file")

```
FULL VIRTUALIZATION                     PARA-VIRTUALIZATION
Guest OS (unmodified)                   Guest OS (modified)
   | tries to write to the disk            | knows it is virtual
   | (thinks it is real hardware)          |
   v                                       v
CPU catches it (trap)                   Calls the hypervisor directly
   |                                       |
   v                                       v
Hypervisor handles it                   Hypervisor handles it
   |                                       |
   v                                       v
Real disk                               Real disk
```

### 12.5 Comparison table

| | Full virtualization | Para-virtualization |
|---|---|---|
| **Does the guest OS know it is virtual?** | **No** | **Yes** |
| **Guest OS modified?** | **No** | **Yes** |
| **How does the hypervisor get involved?** | It **catches** (traps) the guest | The guest **asks** (hypercall) |
| **Compatibility** | Any standard OS, including closed ones like Windows | Only OSes someone has modified (for example open-source Linux) |
| **Speed** | Hardware speed for normal instructions | Hardware speed, with an efficiency edge on I/O |
| **Main limitation** | Catching and emulating privileged actions can cost time | Needs source code changes |

### 12.6 Where to use which

**Full virtualization is used when you must run any standard OS unchanged.**
- Customers bring their own Windows or Linux, and you cannot modify closed-source Windows.
- This is why it is the **default in the cloud**.

**Para-virtualization is useful for: *(beyond the slides)***
- **I/O speed (disk and network).** This is the biggest real-world use today.
- **Older CPUs** with no hardware virtualization support, where full virtualization was slow or impossible.
- **Open-source guests like Linux**, which can be modified easily.
- Early Amazon EC2 ran on the **Xen** hypervisor with para-virtualized Linux guests, a well-known example.

### 12.7 Modern practice: both together *(beyond the slides)*

Providers today usually **combine** the two:
- The VM runs an **unmodified OS** (full virtualization), so any customer OS works.
- Inside it, the OS installs **para-virtualized drivers** for disk and network (VirtIO is a well-known example) for speed.

So para-virtualization lives on mostly as a **performance trick for I/O**, inside fully virtualized VMs.

---

## 13. Why Para-Virtualization Gives Faster I/O

### 13.1 The answer in one paragraph

In para-virtualization, I/O is fast because the guest OS knows it is virtual, so it skips the slow step of pretending to use real hardware. Instead of writing to many small device registers (each one causing a trap into the hypervisor), it places its data in a shared memory queue and sends one direct request to the hypervisor, often handling many packets or disk blocks at once. That means far fewer switches between the VM and the hypervisor, and no need for the hypervisor to imitate a real device in software, which cuts the overhead per I/O operation.

### 13.2 Full virtualization I/O (the slow path)

The guest OS thinks it has a real network card or disk, so it talks to it like real hardware: by writing to many small hardware registers, one step at a time.

Sending **one network packet** might look like this:

1. Guest writes to register 1 (trap into the hypervisor).
2. Guest writes to register 2 (trap again).
3. Guest writes to register 3 (trap again).
4. ... and so on, then waits for an interrupt that the hypervisor must fake.

Each **trap** means:
- The CPU leaves the VM and enters the hypervisor (a **VM exit**).
- The hypervisor works out what the guest meant and **emulates the fake device** in software.
- The CPU switches back into the VM.

Each switch is expensive, and one packet can cause many of them. Multiply by thousands of packets per second and the overhead is large.

### 13.3 Para-virtualization I/O (the fast path)

The guest OS skips the pretend hardware and uses a driver written for the hypervisor:

1. The guest puts the packet in a **shared memory area** (a queue both the guest and hypervisor can read).
2. It sends **one notification**: "new data is ready."
3. The hypervisor picks up the data and sends it through the real network card.

### 13.4 Reasons it is faster

| Reason | Effect |
|---|---|
| **Fewer traps** | One notification instead of many register writes |
| **No fake hardware emulation** | The hypervisor does not have to imitate a specific real card in software |
| **Batching** | Many packets or disk blocks can sit in the queue and be handled together |
| **Shared memory** | Data is not copied back and forth many times |

### 13.5 Analogy

- **Full virtualization:** you want to order food, but the restaurant pretends to be an old-style phone system. You press a long series of buttons, and a staff member interprets each press.
- **Para-virtualization:** you drop a complete order form in a box and ring one bell. The kitchen collects it all at once.

### 13.6 Illustrative numbers (made up to show the idea)

- Emulated device: 1 packet is about 10 traps.
- Para-virtualized: 1 packet is about 1 notification, and with batching, 1 notification can cover 50 packets.

Even if each switch costs the same, the second approach does a small fraction of the switching.

### 13.7 Link to your slide

Your slide says para-virtualization **replaces I/O and privileged calls with hypervisor traps/calls**. That is this idea: instead of risky hardware-style operations that must be intercepted one by one, the guest makes a **deliberate, direct request**.

*Exam tip:* the mechanics (VirtIO, shared queues, VM exits) go beyond the slides. For exams, say: **"fewer traps, and direct calls to the hypervisor instead of emulating hardware."**

---

## 14. CPU Rings and Hardware-Assisted Virtualization

*(beyond the slides, but it explains your slide on three modes)*

### 14.1 What are CPU rings?

**Rings** are privilege levels on x86 CPUs, numbered 0 to 3. A **lower number means more power**.

| Ring | Who runs here | Power |
|---|---|---|
| **Ring 0** | OS kernel | Full hardware access |
| Ring 1 and 2 | Rarely used | n/a |
| **Ring 3** | User applications | Restricted |

This is the same idea as your lecture's **kernel mode (ring 0)** and **user mode (ring 3)**.

### 14.2 The problem with a hypervisor

Both the **hypervisor** and every **guest kernel** want ring 0, but only one can have it. The hypervisor must win, because it controls everything.

### 14.3 Three solutions

| Approach | How it handles the ring problem |
|---|---|
| **1. Software tricks (early full virtualization)** | The guest kernel is pushed to a weaker ring (for example ring 1). The hypervisor traps or rewrites its privileged instructions on the fly. This is slow and complex, because some older x86 instructions misbehave instead of trapping. |
| **2. Para-virtualization** | The guest kernel is also moved to a weaker ring, but because it was **modified**, it calls the hypervisor instead of executing risky instructions. The problem is avoided entirely. |
| **3. Hardware-assisted virtualization (Intel VT-x, AMD-V)** | The CPU adds a **new privilege level below ring 0**, informally called **"ring -1"**. The hypervisor lives there, and the guest kernel stays in **ring 0**, believing it is in full control. |

Option 3 is why **full virtualization became fast** and is the cloud standard.

### 14.4 Connecting to your lecture's three modes

```
Ring -1  ->  Hypervisor mode   (hypervisor)
Ring  0  ->  Kernel mode       (guest OS)
Ring  3  ->  User mode         (applications)
```

Your slide's **"three hardware privilege levels"** are exactly this hardware-assisted model.

*Note:* "ring -1" is an informal name. Technically, Intel and AMD describe it as separate hypervisor ("root") and guest ("non-root") modes of the CPU.

---

## 15. Walkthrough: What Happens When a File Upload Arrives in a VM

### 15.1 The question
When an upload request arrives at an app in a VM, control goes from the app to the kernel. Does the kernel save the file itself, or does it tell the hypervisor? When does the kernel need the hypervisor?

### 15.2 Short answer
The guest kernel handles **most of the work itself**. It involves the hypervisor only when it needs something **outside its own VM's boundaries**, such as **real hardware**.

### 15.3 Step by step

| Step | Who | What happens | Mode |
|---|---|---|---|
| 1 | **App** | Receives the upload and makes a system call: "save this file". | User mode, then switches to kernel mode |
| 2 | **Guest kernel** | Does the OS work inside the VM: checks permissions, decides file name and location, holds the data in the VM's **RAM** (often as a cache). | Kernel mode |
| 3 | **Guest kernel** | Needs to write to the "disk". The VM only has a **virtual disk**, so this write is **trapped** (full virtualization) or sent as a **hypervisor call** (para-virtualization). | Switch toward hypervisor mode |
| 4 | **Hypervisor** | Stores the data on the **real physical storage**, in the area belonging to this VM. | Hypervisor mode |
| 5 | **Hypervisor, kernel, app** | Control returns to the kernel, then to the app: "saved". | Back down to kernel, then user mode |

### 15.4 What the kernel can do alone

- Manage its own memory (inside the amount allocated to the VM)
- Start, stop and schedule apps
- Handle file system logic and permissions
- Run ordinary system calls

### 15.5 When the kernel needs the hypervisor

- **Real hardware I/O:** disk, network card and other physical devices
- **More memory** than the VM was given, or anything touching memory outside its boundaries
- **Anything that could affect other VMs:** CPU cores, shared devices, hardware configuration
- **Privileged actions** that only hypervisor mode may perform

### 15.6 Rule of thumb

- **Inside my VM's walls** -> the kernel does it alone.
- **Outside my VM's walls, or real hardware** -> the hypervisor does it.

### 15.7 Virtual disk vs virtual memory (do not mix these up)

| | Virtual disk | Virtual memory |
|---|---|---|
| **What it is** | The **fake disk** the VM sees. To the guest OS it looks like a real hard drive. | A technique for managing **RAM** (the VM's allocated memory). |
| **Holds** | Permanent files | Data currently in use |
| **Where it really lives** | Typically a **file or reserved area** on the real disk, managed by the hypervisor | In the VM's allocated real RAM |

So the kernel first holds the upload in the VM's **RAM**, then "writes it to the **virtual disk**". That write goes to the hypervisor, which stores it on the **real physical disk**.

### 15.8 One-line flow

```
App  ->  Kernel (VM's RAM, virtual disk)  ->  Hypervisor  ->  Real physical disk
```

### 15.9 Analogy
You write a letter and put it in your office **outbox** (virtual disk). You think it is delivered. Really, the building's **mailroom** (hypervisor) collects it and sends it through the real postal system (physical disk). The floor manager (kernel) can reorganize papers inside their own floor, but cannot use the mailroom directly.

### 15.10 Why route everything through the hypervisor?
If VM-A and VM-B both wrote straight to the real disk, they could overwrite each other's data. Routing every write through the hypervisor lets it keep each VM's data in a **separate area**. This is the **isolation** from your lecture.

*Note:* your slide simplifies this by saying a system call traps "back to Kernel or Hypervisor mode". The exact details depend on the hypervisor, but the rule of thumb above is the idea to remember.

---

## 16. Master Comparison Cheat Sheet

### 16.1 Three virtualization approaches

| | Software Emulation | Para-virtualization | Full Virtualization |
|---|---|---|---|
| Mechanism | Byte-code interpreters translate instructions | Hypervisor calls replace privileged OS instructions | VM presents real-hardware instruction set |
| Speed | Slow | Near-native | Hardware speed |
| Modified guest OS needed? | n/a | **Yes** | **No** |
| Main advantage | Portability across CPUs | Speed | Speed plus unmodified OSes |
| Main drawback | Severe performance penalty | Must alter OS source code | (None highlighted; the production standard) |
| Used in cloud data centers? | No, ineligible as primary form | Limited | **Yes** |
| Example | Java Virtual Machine | Early (1960s) hypervisor systems | EC2, Compute Engine, Azure VMs |

### 16.2 The three privilege modes

| Mode | Runs | Privilege | Key limit |
|---|---|---|---|
| Hypervisor mode | Hypervisor | Highest | Must configure isolation correctly |
| Kernel mode | Guest OS | Restricted | Only its own VM's memory |
| User mode | Tenant apps | Basic | Runs natively, traps on privileged actions |

### 16.3 Two-mode vs three-mode systems

| | Traditional computer | Virtualized (cloud) computer |
|---|---|---|
| Modes | Kernel, User | Hypervisor, Kernel, User |
| First software to start | Operating system | Hypervisor |
| OS power | Full control of hardware | Limited to its VM |

### 16.4 Full vs para-virtualization: exam-style quick table

| Question | Full virtualization | Para-virtualization |
|---|---|---|
| Guest knows it is virtual? | No | Yes |
| Guest OS modified? | No | Yes |
| Privileged action handled by | **Trap** (hypervisor catches it) | **Hypercall** (guest asks) |
| Works with Windows? | Yes | Only if Windows is modified (practically no) |
| Typical use today | Main cloud approach | Fast I/O drivers (VirtIO) inside fully virtualized VMs |
| One-line memory trick | "You don't know, I'll handle it" | "You know, so ask me" |

### 16.5 Hypervisor types

| | Type 1 (bare-metal) | Type 2 (hosted) |
|---|---|---|
| Location | On hardware, below all OSes | On top of a host OS |
| Where used | Cloud data centers | Laptops and desktops |
| Examples | ESXi, Xen, Hyper-V | VirtualBox, VMware Workstation |

### 16.6 Rings and modes mapped together

| x86 ring | Lecture's mode | Runs |
|---|---|---|
| Ring -1 (informal) | Hypervisor mode | Hypervisor |
| Ring 0 | Kernel mode | Guest OS |
| Ring 3 | User mode | Applications |

---

## 17. Common Confusions Cleared

**Q1. What is the difference between a VM and a hypervisor?**
The VM is the *virtual computer* (guest). The hypervisor is the *software that creates and manages* the VMs.

**Q2. Does the hypervisor run inside a VM?**
No. The hypervisor runs *beneath* the VMs, directly on the server hardware.

**Q3. If VMs share one CPU, how are they fast?**
Because most instructions (the non-privileged ones) execute **directly on the real CPU**. The hypervisor only gets involved for the occasional privileged operation.

**Q4. Why isn't software emulation used in cloud data centers?**
Its translation causes a **severe performance penalty**. Cloud customers pay for speed.

**Q5. What is the difference between para-virtualization and full virtualization?**
Para-virtualization needs the **guest OS to be modified** to ask the hypervisor for privileged actions. Full virtualization runs **unmodified** operating systems.

**Q6. What does "native" or "hardware speed" mean?**
The code runs directly on the physical CPU, with no translation layer in between.

**Q7. Is kernel mode still the "highest" mode inside a VM?**
It is the highest mode *that the guest OS can use*, but in a virtualized system it is below hypervisor mode. The guest OS is restricted to its own VM's memory.

**Q8. What is a "trap"?**
A hardware-triggered, automatic switch to a more privileged layer, caused by a system call or by an attempted illegal action.

**Q9. Who starts first on a virtualized server?**
The **hypervisor**. On a normal, non-virtualized computer, the OS starts first.

**Q10. Why must the hypervisor be trusted completely?**
It configures the hardware boundaries between VMs. If it makes a mistake, one VM could read another's memory.

**Q11. Is the Java Virtual Machine the same as a cloud VM?**
No. The JVM is **software emulation** (byte-code interpretation, used for portability). A cloud VM from EC2 uses **full virtualization** (a whole computer with its own OS). The names are similar, but the technologies differ.

**Q12. Is the hypervisor above or below the "main OS"?**
In the cloud, it is **below all operating systems**, directly on the hardware. There is no main OS beneath it. (Only a Type 2 hypervisor, like VirtualBox on a laptop, sits above a host OS.)

**Q13. Does the guest kernel save the uploaded file directly to the real disk?**
No. It handles the OS work inside its VM, then writes to a **virtual disk**. That write is passed to the hypervisor, which stores it on the real disk.

**Q14. Is "virtual disk" the same as "virtual memory"?**
No. Virtual disk is the fake disk the VM sees (permanent storage). Virtual memory is a RAM-management technique.

**Q15. In full virtualization, does the guest OS ever interact with the hypervisor?**
Yes, but without knowing it. When it attempts a privileged action, the hardware traps it to the hypervisor automatically.

**Q16. If para-virtualization needs a modified OS, is it useless today?**
No. It survives mainly as **para-virtualized drivers** (for example VirtIO) for disk and network inside fully virtualized VMs, giving faster I/O.

**Q17. Why is para-virtualization faster for I/O?**
The guest makes direct requests through a shared queue instead of the hypervisor emulating a fake hardware device through many traps.

**Q18. What are CPU rings?**
Privilege levels on x86 CPUs: ring 0 (kernel, most power) to ring 3 (user apps, least). Hardware-assisted virtualization adds a level below ring 0 (informally "ring -1") for the hypervisor.

**Q19. Why did full virtualization become fast?**
Because CPUs gained hardware support (Intel VT-x, AMD-V) that gives the hypervisor its own privilege level, so guest kernels can stay in ring 0 without clashing.

**Q20. When does the kernel need the hypervisor?**
For real hardware I/O, memory outside its VM, and anything that could affect other VMs. Inside its own VM's walls, the kernel works alone.

---

## 18. Glossary

| Term | Meaning |
|---|---|
| **Byte code** | Intermediate code written for a hypothetical machine and interpreted at run time (for example, Java byte code). |
| **Commercial instruction set** | Standard CPU command set used by real, widely sold hardware. |
| **Emulation** | Making one system imitate another, often by translating instructions in software. |
| **Full virtualization** | Virtualization in which unmodified guest OSes run at near-hardware speed with isolation. |
| **Guest OS** | The operating system running inside a VM. |
| **Hardware trap** | An automatic CPU-triggered switch to a more privileged mode, caused by a system call or a forbidden action. |
| **Heterogeneous CPUs** | Different types of CPUs. |
| **Host** | The physical computer on which VMs run. |
| **Hypervisor** | Software that creates and manages VMs and controls the underlying hardware. |
| **Hypervisor call** | A request from a modified guest OS asking the hypervisor to perform a privileged action. |
| **Hypervisor mode** | The highest CPU privilege level, used by the hypervisor. |
| **Isolation** | Guarantee that one VM cannot see or affect another. |
| **JVM** | Java Virtual Machine, an example of software emulation using byte code. |
| **Kernel** | The core of an operating system. |
| **Kernel mode** | Restricted privilege level in which the guest OS runs. |
| **Native execution** | Running directly on the physical CPU at full hardware speed. |
| **Non-privileged instruction** | An ordinary instruction (like arithmetic) that is safe to run directly on the CPU. |
| **Overhead** | Extra time or resources spent on something other than the real work. |
| **Para-virtualization** | Virtualization in which the guest OS is modified to use hypervisor calls instead of privileged instructions. |
| **Privileged instruction** | An instruction that controls hardware or memory and may only be run by trusted software. |
| **Sandboxing** | Confining software to a restricted area so it cannot harm anything outside. |
| **System call** | A request from an app to the OS for a service. |
| **Tenant** | The customer who owns a VM. |
| **User mode** | Basic privilege level in which tenant applications run. |
| **Virtual machine (VM)** | A software-created computer that emulates a separate physical computer. |

### Additional Terms (from the deep-dive sections)

| Term | Meaning |
|---|---|
| **AMD-V / Intel VT-x** | CPU hardware features that support virtualization by giving the hypervisor its own privilege level. |
| **Batching** | Handling many I/O operations together in one go. |
| **Hardware-assisted virtualization** | Virtualization made fast by special CPU support (VT-x, AMD-V). |
| **Hypercall** | A direct request from a modified guest OS to the hypervisor. |
| **Ring (CPU ring)** | An x86 privilege level, 0 (most privileged) to 3 (least). |
| **Ring -1** | Informal name for the hypervisor's privilege level below ring 0. |
| **Shared memory queue** | A memory area both guest and hypervisor can read, used for fast para-virtualized I/O. |
| **Type 1 hypervisor** | Bare-metal hypervisor running directly on hardware (cloud). |
| **Type 2 hypervisor** | Hosted hypervisor running on top of a normal OS (laptops). |
| **VirtIO** | A well-known set of para-virtualized drivers for disk and network. |
| **Virtual disk** | The fake disk a VM sees, backed by real storage managed by the hypervisor. |
| **VM exit** | The CPU leaving a VM to run the hypervisor, usually after a trap. |
| **Xen** | A well-known hypervisor, used in early AWS EC2. |

---

## 19. Self-Test (with Answers)

### Short-answer questions

**1. Define a virtual machine and list three of its properties.**
*Answer:* A computer system created using software on one physical computer to emulate another separate physical computer. Properties: it cannot interfere with the host or other VMs; it runs its own kernel; it is managed by a hypervisor.

**2. Name the three major cloud VM services.**
*Answer:* AWS Elastic Compute Cloud (EC2), Google Compute Engine, Azure VMs.

**3. What is a hypervisor and what does it do?**
*Answer:* Software that creates and manages VMs and controls the underlying hardware. It lets many VMs run on one server, lets each tenant boot an OS and launch apps, and keeps each VM independent.

**4. Name the three approaches to virtualization.**
*Answer:* Software emulation, para-virtualization, full virtualization.

**5. Why is software emulation unsuitable as the primary form of virtualization in cloud data centers?**
*Answer:* Translating instructions in software (byte-code interpretation) causes a severe performance penalty.

**6. What is the major drawback of para-virtualization?**
*Answer:* It requires altering the guest operating system's source code to replace I/O and privileged calls with hypervisor traps.

**7. List the three key properties of full virtualization.**
*Answer:* Commercial instruction set emulation (unmodified Linux and Windows boot); isolated facilities and operation; efficient low-overhead execution (non-privileged instructions run directly at native speed).

**8. How many hardware modes do traditional OSes use, and how many does cloud virtualization use? Name them.**
*Answer:* Traditional: 2 (kernel mode, user mode). Cloud virtualization: 3 (hypervisor mode, kernel mode, user mode).

**9. What can the hypervisor do that the guest OS cannot?**
*Answer:* Create VMs and allocate memory to them, and manage host resources. The guest OS is restricted to the memory allocated to its VM.

**10. Describe the four steps of execution and state transitions.**
*Answer:* (1) Hypervisor boots in hypervisor mode and allocates memory and configures hardware. (2) The guest OS is started in kernel mode within isolated memory. (3) The guest OS launches apps in user mode at full hardware speed. (4) An app's service call triggers a hardware trap, switching the CPU back to kernel or hypervisor mode.

**11. What happens if an app attempts unauthorized memory access?**
*Answer:* The hardware automatically traps execution back to the governing layer (guest OS or hypervisor), which blocks it.

**12. Why must the hypervisor be trusted completely?**
*Answer:* It manages CPU cores, memory boundaries and hardware interfaces, and must configure CPU isolation correctly so no guest OS can breach its VM or inspect adjacent memory.

**13. How can an application in a VM run at hardware speed?**
*Answer:* Applications run in user mode, and non-privileged instructions execute directly on the physical CPU. The hypervisor and OS are involved only when a privileged operation causes a trap.

**14. Where does the hypervisor sit relative to operating systems in a cloud server?**
*Answer:* Directly on the hardware, **below** all guest operating systems. There is no main OS beneath it.

**15. What are the three jobs of a hypervisor?**
*Answer:* Divide the hardware among VMs, isolate the VMs from each other, and control the real hardware.

**16. What is the core difference between full and para-virtualization?**
*Answer:* In full virtualization the guest OS does not know it is virtual (unmodified) and the hypervisor catches its privileged actions through traps. In para-virtualization the guest is modified, knows it is virtual, and asks the hypervisor directly through hypercalls.

**17. Why is para-virtualization faster for I/O?**
*Answer:* The guest skips pretending to use real hardware. It uses a shared memory queue and one direct request, often for many operations at once, which means far fewer traps and no software emulation of a device.

**18. Where is para-virtualization useful today?**
*Answer:* Fast disk and network I/O through para-virtualized drivers (for example VirtIO) inside fully virtualized VMs; older CPUs without hardware support; and open-source guests like Linux that are easy to modify.

**19. What are CPU rings, and how do they relate to the three modes in the lecture?**
*Answer:* Privilege levels on x86 CPUs (0 to 3). Ring 0 is kernel mode and ring 3 is user mode. Hardware-assisted virtualization adds a level below ring 0 (informally "ring -1") for the hypervisor mode.

**20. In the file-upload example, what does the guest kernel do alone, and what goes to the hypervisor?**
*Answer:* The kernel does permission checks, file system logic and holds the data in the VM's RAM. The write to the virtual disk goes to the hypervisor, which stores it on the real physical disk.

**21. Difference between a virtual disk and virtual memory?**
*Answer:* A virtual disk is the fake permanent-storage device a VM sees, backed by real storage through the hypervisor. Virtual memory is a technique for managing RAM.

**22. Type 1 vs Type 2 hypervisor?**
*Answer:* Type 1 runs directly on hardware, below all OSes, and is used in cloud data centers (Xen, ESXi, Hyper-V). Type 2 runs on top of a host OS, for example VirtualBox on a laptop.

### Scenario questions

**S1.** A company wants to run an unmodified, standard commercial operating system at near-native speed on a cloud server. Which virtualization approach makes this possible?
*Answer:* **Full virtualization.**

**S2.** A developer writes a program once and wants it to run on many types of CPUs, accepting slower speed. Which approach is that?
*Answer:* **Software emulation** (for example, Java byte code on the JVM).

**S3.** A research group modifies the source code of an OS so it asks the hypervisor for privileged actions, getting near-native speed. Which approach?
*Answer:* **Para-virtualization.**

**S4.** Malicious code inside VM-A tries to read VM-B's memory. What stops it?
*Answer:* The CPU's hardware privilege enforcement and the hypervisor's isolation configuration. The illegal access triggers a hardware trap back to the governing layer.

**S5.** A guest OS finishes and exits. Which mode does control return to?
*Answer:* **Hypervisor mode.**

**S6.** In which mode does a customer's web application normally run, and why is it fast?
*Answer:* **User mode.** It executes natively on the physical CPU with no translation.

**S7.** You install VirtualBox on your Windows laptop and run Ubuntu in it. Which hypervisor type is VirtualBox, and why?
*Answer:* **Type 2**, because it runs on top of a host OS (Windows) rather than directly on the hardware.

**S8.** A guest OS tries to write to a disk controller without knowing it is virtual. What happens?
*Answer:* The CPU traps the privileged action to the hypervisor, which performs the write safely on the real disk. This is **full virtualization**.

**S9.** A modified Linux guest wants to write to disk and sends "hypervisor, please write this" directly. Which approach?
*Answer:* **Para-virtualization** (a hypercall).

**S10.** A provider wants customers to run any OS, but also wants fast network I/O. What combination does it use?
*Answer:* **Full virtualization** for the unmodified OS, plus **para-virtualized drivers** (for example VirtIO) for I/O.

---

## 20. One-Page Revision Summary

1. **VM** = a software-created computer emulating a separate physical one. It cannot interfere with the host or other VMs, runs its own kernel, and is managed by a hypervisor. **Examples:** AWS EC2, Google Compute Engine, Azure VMs.
2. **Hypervisor** = software that creates and manages VMs and controls hardware. Stack from bottom: **server hardware → hypervisor → VMs (each with own OS) → apps.** The tenant boots an OS on the VM and launches apps.
3. **Three approaches:**
   - **Software emulation:** byte-code interpreters (JVM); portable but severe overhead, so not used as the primary cloud form.
   - **Para-virtualization:** pioneered in the 1960s; hypervisor calls replace privileged instructions; near-native speed but the **guest OS source must be modified**.
   - **Full virtualization:** unmodified OSes at hardware speed with isolation. **The cloud standard.**
4. **Full virtualization properties:** (a) commercial instruction set emulation (Linux and Windows boot unaltered); (b) isolated facilities and operation (each OS thinks it owns memory, devices and CPU cores); (c) efficient low-overhead execution (non-privileged instructions run natively).
5. **Hardware speed comes from CPU modes.** Traditionally: OS in **kernel mode**, apps in **user mode**.
6. **Cloud adds a third mode:**
   - **Hypervisor mode:** highest; creates VMs and allocates memory.
   - **Kernel mode:** guest OS; restricted to its VM's memory.
   - **User mode:** tenant apps; run natively.
7. **Flow:** hypervisor starts first → creates VM and starts OS → OS starts app → app invokes OS service → OS exits back to hypervisor.
8. **Four-step timeline:** Hypervisor boot → Guest OS start (kernel mode) → App launch (user mode, full speed) → System call trap (back to kernel or hypervisor mode).
9. **Hierarchy of trust:** the hypervisor is completely trusted and must configure isolation correctly. Guest OSes and apps are lower trust and hardware-restricted. Illegal actions are **trapped by hardware** back to the governing layer.
10. **Big picture:** hardware-enforced privilege levels are what make **fast, isolated, multi-tenant** cloud VMs possible.
11. **Hypervisor location:** directly on the hardware, **below all guest OSes** (Type 1, used in the cloud). Type 2 (VirtualBox) runs on top of a host OS. Hypervisor jobs: divide, isolate, control real hardware.
12. **Full vs para (core difference):** full = guest does not know it is virtual (unmodified, privileged actions **trapped**); para = guest knows (modified, **asks** via hypercalls). *Full: "you don't know, I'll handle it." Para: "you know, so ask me."*
13. **Para's usefulness:** fast I/O (via drivers like VirtIO), older CPUs, easy-to-modify open-source guests. Modern clouds use **full virtualization + para-virtualized I/O drivers**.
14. **Why para I/O is fast:** the guest skips pretending to use real hardware, uses a shared memory queue and one direct request (often batched), so far fewer traps and no device emulation.
15. **CPU rings:** ring 0 = kernel, ring 3 = user. Hardware-assisted virtualization (VT-x, AMD-V) adds a level below ring 0 ("ring -1") for the hypervisor, which makes full virtualization fast. This equals the lecture's three modes.
16. **File upload flow:** App -> kernel (checks, RAM, virtual disk) -> hypervisor -> real disk. Kernel works alone **inside its VM's walls**; the hypervisor handles **real hardware and anything outside the VM**. Virtual disk is not virtual memory.

---

*End of Handbook*
