# Synchronization in Distributed Systems: Complete Handbook
### Clock Synchronization, Logical Clocks, Vector Clocks, Totally Ordered Multicast
*Based on: IT457 Cloud Computing, Week 8 – Lecture 2 (Chapter 6 figures)*

---

## How to Use This Handbook

This handbook is written so that you should not need to ask follow-up questions. For every concept it gives:

1. **The idea** in plain words
2. **A worked example** (with the numbers from your slides wherever possible)
3. **Why it matters / how it connects** to the rest of the lecture
4. **Common confusions** cleared on the spot

At the end you get a **formula sheet, comparison tables, a glossary, practice problems with full solutions, a self-test, and a one-page revision summary**.

> **Marking convention:** Everything not marked is taken from your slides. Material marked ***(beyond the slides)*** is standard textbook knowledge added so that nothing feels like a gap. For exams, always prefer your slide wording, and use the extra material to understand *why*.

---

## Slide-to-Section Map

| Slide (topic) | Where it is explained |
|---|---|
| Clock Synchronization, Fig 6-1 (compiler/editor) | Section 1 |
| Global Time and Global State of Distributed Systems | Section 2 |
| Simulating global time: skew, drift, precision, accuracy, internal vs external | Section 3 |
| Getting time from a time server (T1 to T4, offset θ, delay δ, example numbers) | Section 5 |
| Physical clocks, UTC | Section 4 |
| Cristian's algorithm and the numerical problem | Section 6 |
| Network Time Protocol (NTP) | Section 7 |
| Berkeley algorithm, Fig 6-7 | Section 8 |
| Happened-before relationship | Section 10 |
| Lamport's logical clocks (1) to (5), Figs 6-9 and 6-10 | Section 11 |
| Partial order logical clocks paragraph | Section 12 |
| Totally ordered multicasting, Fig 6-11 | Section 13 |
| Vector clocks (1) to (3), Fig 6-12, the handwritten diagram | Section 14 |
| Global snapshots (mentioned on the "Global Time" slide) | Section 16 |

---

## Table of Contents

**Part A: The Problem and the Vocabulary**
1. [Why Clock Synchronization Is Needed (Fig 6-1)](#1-why-clock-synchronization-is-needed-fig-6-1)
2. [Global Time and Global State in Distributed Systems](#2-global-time-and-global-state-in-distributed-systems)
3. [Simulating Global Time: Skew, Drift, Precision, Accuracy](#3-simulating-global-time-skew-drift-precision-accuracy)

**Part B: Physical Clock Synchronization**
4. [Physical Clocks and UTC](#4-physical-clocks-and-utc)
5. [Getting the Time from a Time Server (T1 to T4, θ and δ)](#5-getting-the-time-from-a-time-server-t1-to-t4-θ-and-δ)
6. [Cristian's Algorithm](#6-cristians-algorithm)
7. [Network Time Protocol (NTP)](#7-network-time-protocol-ntp)
8. [The Berkeley Algorithm](#8-the-berkeley-algorithm)
9. [Physical Clock Algorithms Compared](#9-physical-clock-algorithms-compared)

**Part C: Logical Time**
10. [Order Matters More Than Time: Happened-Before](#10-order-matters-more-than-time-happened-before)
11. [Lamport's Logical Clocks](#11-lamports-logical-clocks)
12. [Partial Order vs Total Order](#12-partial-order-vs-total-order)
13. [Totally Ordered Multicasting](#13-totally-ordered-multicasting)
14. [Vector Clocks](#14-vector-clocks)
15. [Lamport Clocks vs Vector Clocks](#15-lamport-clocks-vs-vector-clocks)
16. [Global Snapshots (Brief)](#16-global-snapshots-brief)

**Part D: Revision Tools**
17. [Where This Shows Up in Real Cloud Systems](#17-where-this-shows-up-in-real-cloud-systems)
18. [Formula Sheet](#18-formula-sheet)
19. [Common Confusions Cleared](#19-common-confusions-cleared)
20. [Glossary](#20-glossary)
21. [Practice Problems with Full Solutions](#21-practice-problems-with-full-solutions)
22. [Self-Test (Short Questions with Answers)](#22-self-test-short-questions-with-answers)
23. [One-Page Revision Summary](#23-one-page-revision-summary)

---

## The Big Picture (Read This First)

A distributed system is many computers cooperating over a network. They have **no shared memory and no shared clock**. That creates one central question:

> **How can computers that cannot see each other's clocks agree on time, or at least on the order in which things happened?**

There are two families of answers:

```
                 SYNCHRONIZATION IN DISTRIBUTED SYSTEMS
                                  |
          +-----------------------+------------------------+
          |                                                |
   PHYSICAL CLOCKS                                   LOGICAL CLOCKS
   "What time is it, really?"                        "What happened before what?"
   (need real time, e.g., UTC)                       (need only the ORDER of events)
          |                                                |
   +------+---------+---------+                     +------+-----------+
   |                |         |                     |                  |
 Cristian's        NTP     Berkeley             Lamport clocks     Vector clocks
 algorithm                 algorithm            (scalar number)    (vector of numbers)
 (external)     (external) (internal)           -> consistent      -> detects
                                                   ordering           concurrency
                                                       |
                                              Totally ordered multicast
                                              (replicated database example)
```

| Question | Use |
|---|---|
| "I need the actual wall-clock time (for example, to stamp a bank transaction in UTC)." | **Physical clocks**: Cristian, NTP |
| "I need all machines to agree with each other, even if not with real time." | **Physical, internal sync**: Berkeley |
| "I only care that cause comes before effect." | **Logical clocks**: Lamport |
| "I must also know when two events are unrelated (concurrent)." | **Vector clocks** |

---

# PART A: THE PROBLEM AND THE VOCABULARY

## 1. Why Clock Synchronization Is Needed (Fig 6-1)

### What the slide shows
Two computers, each with its **own local clock**:

```
Computer running the compiler :   2144     2145     2146     2147
                                    *
                              output.o created

Computer running the editor   :   2142     2143     2144     2145
                                             *
                                       output.c created
```

> **Caption:** *When each machine has its own clock, an event that occurred after another event may nevertheless be assigned an earlier time.*

### The story behind the figure (the `make` program)
This is the classic example. A build tool such as `make` decides whether to recompile a program by comparing **file timestamps**:

- If the source file (`output.c`) is **newer** than the compiled file (`output.o`), then the source has changed, so **recompile**.
- If `output.o` is **newer or equal**, nothing changed, so **skip compiling**.

Now imagine this sequence in **real time**:

1. The compiler machine creates `output.o`. Its clock says **2144**.
2. *Later*, the programmer edits the source on the editor machine and saves `output.c`. The editor machine's clock is a little behind, so it says **2143**.

In reality `output.c` is **newer** (it was changed after `output.o` was built). But the timestamps say `output.c` = 2143 and `output.o` = 2144, so `output.c` *looks older*. `make` wrongly concludes nothing changed and **does not recompile**. The programmer's latest change is silently ignored.

### Why this happens
Each machine's clock is slightly different (and runs at a slightly different speed). There is **no single, shared clock**, so timestamps from different machines **cannot be compared safely**.

### Everyday analogy
Two friends each have a wristwatch, but the watches are not set to the same time. Friend A says "I posted the letter at 2:44". Friend B says "I received a reply at 2:43". Looking only at the numbers, the reply seems to arrive *before* the letter was posted, which is impossible.

### Why it matters
Any system that uses timestamps to decide "which update is newer" (databases, file sync, caches, logs, auctions, banking) can give **wrong answers** if clocks disagree.

---

## 2. Global Time and Global State in Distributed Systems

### 2.1 The setting (from the slide)
> **Asynchronous distributed systems** consist of several processes **without common memory** which communicate **(solely) via messages** with **unpredictable transmission delays**.

Breaking this down:

| Term | Meaning | Example |
|---|---|---|
| **Process** | A running program on one machine | A web server on machine 1, a database on machine 2 |
| **Without common memory** | Processes cannot read each other's variables directly | Machine 1 cannot just look at machine 2's RAM |
| **Communicate solely via messages** | The only way to share information is to send a network message | A request over HTTP |
| **Unpredictable transmission delays** | A message may take 5 ms or 500 ms, and you cannot know in advance | Network congestion |
| **Asynchronous** | There is **no upper bound** on message delay or on how fast each process runs | The Internet |

*(beyond the slides)* The opposite is a **synchronous** system, in which message delay and clock speed have known upper bounds. Synchronous systems are much easier to reason about. The goal in this lecture is to **make an asynchronous system behave like a synchronous one as far as possible**.

### 2.2 Why global time and global state are hard (from the slide)

| Reason | Explanation |
|---|---|
| **Processes are distributed geographically** | Mumbai and London cannot look at the same clock instantly. Even light takes time to travel. |
| **Rate of event occurrence can be high (unpredictable)** | Thousands of events per second, at random moments. |
| **Event execution times can be small** | Events happen faster than the precision you could use to compare them. |

**Global time** means one clock that all processes agree on.
**Global state** means a consistent picture of *all* processes and messages at one instant (like a group photo).

### 2.3 What we do instead: approximate (from the slide)

> We can only approximate the global view.

| Goal | Approximation technique | Section |
|---|---|---|
| Simulate a **synchronous system** on top of an asynchronous one | Clock synchronization algorithms | Sections 5 to 8 |
| Simulate **global time** | **Logical clocks** | Sections 10 to 15 |
| Simulate **global state** | **Global snapshots** | Section 16 |

---

## 3. Simulating Global Time: Skew, Drift, Precision, Accuracy

### 3.1 Clocks drift (from the slide)
Clocks in a distributed system drift:
- **Relative to each other**, and
- **Relative to a real-world clock**.

Why? Every computer has a **quartz crystal oscillator** that ticks at a nearly (but not exactly) fixed rate. Temperature, age, manufacturing differences and voltage make each crystal slightly different. *(beyond the slides)*

### 3.2 Clock skew vs clock drift (very important, often confused)

| Term | Definition (slide) | Think of it as | Unit |
|---|---|---|---|
| **Clock skew** | Relative difference in clock **values** of two processes | "How far apart do the clocks read **right now**?" | seconds |
| **Clock drift** | Relative difference in clock **frequencies (rates)** of two processes | "How fast is the gap **growing**?" | seconds per second (or ppm) |

**Worked example:**
- Clock A reads 10:00:05 and clock B reads 10:00:00 at the same real instant. **Skew = 5 seconds.**
- Clock A ticks 1.001 s for every real second. Clock B ticks exactly 1.000 s. **Relative drift = 0.001 s per second** (1 ms every second).
- So the skew grows: after 1,000 seconds, A has gained an extra 1 second on B, and skew becomes 6 s.

**Analogy:** Two runners. *Skew* is the distance between them right now. *Drift* is the difference in their speeds, which determines how fast that distance changes.

**Typical numbers** *(beyond the slides)*: ordinary quartz clocks drift about 10⁻⁶ to 10⁻⁵ (1 to 10 parts per million), i.e., roughly **0.09 s to 0.86 s per day**. That is why clocks must be **re-synchronized periodically**.

### 3.3 How often must we resync? *(beyond the slides)*
If every clock's maximum drift rate is ρ, two clocks can move apart by up to 2ρ per second. To keep their difference within δ, resynchronize at least every

```
        δ
 t  =  ───
       2ρ
```

**Example:** ρ = 10⁻⁵, required δ = 1 ms = 0.001 s → t = 0.001 / (2 × 10⁻⁵) = **50 seconds**.

### 3.4 Why synchronization is needed (from the slide)
Clock synchronization is needed to simulate global time, for:
- **Correctness**: consistency of data (the `make` example)
- **Fairness**: for example, the first request should be served first

### 3.5 Precision and accuracy (slide terms)

| Term | Meaning | Written as |
|---|---|---|
| **Precision** | Deviation between **any two clocks on any two machines** is within a specified bound | clocks agree **with each other** |
| **Accuracy** | The clock is kept within a bound **α** of the **real time** (reference such as UTC) | clocks agree **with real time** |

*(beyond the slides)* In symbols: precision means |Cᵢ(t) − Cⱼ(t)| ≤ π for all machines i, j. Accuracy means |Cᵢ(t) − t| ≤ α, where t is true time.

**Analogy (archery):**
- **Precise but not accurate:** all arrows land close together, but far from the bullseye.
- **Accurate:** arrows land near the bullseye (and therefore also near each other).

*(beyond the slides)* If every clock is accurate within α, then any two clocks are within 2α of each other (so accuracy implies precision of 2α). The reverse is not true: clocks can agree with each other and all be wrong together.

### 3.6 Internal vs external synchronization (from the slide)

| | Internal synchronization | External synchronization |
|---|---|---|
| **Keeps clocks...** | **Precise** (agreeing with each other) | **Accurate** (agreeing with real time/UTC) |
| **Needs an outside reference?** | **No** | **Yes** (UTC source, such as an atomic clock or GPS) |
| **Algorithm examples** | **Berkeley algorithm** | **Cristian's algorithm, NTP** |
| **Example use** | A cluster of servers that must agree with each other | A server that must show the correct legal time |

---

# PART B: PHYSICAL CLOCK SYNCHRONIZATION

## 4. Physical Clocks and UTC

### The problem (slide)
Sometimes we simply need the **exact time**, not just an ordering. Examples: legal time-stamping of a contract, scheduling a task at 9:00 AM, expiry of a certificate, stamping a stock trade.

### The solution: UTC (Universal Coordinated Time)

| Point on the slide | Explained |
|---|---|
| Based on the number of **transitions per second of the cesium-133 atom** | An atomic clock counts the extremely regular oscillations of cesium-133 atoms. *(beyond the slides)* The official second is defined as 9,192,631,770 such oscillations. |
| Real time is taken as the **average of about 50 cesium clocks** around the world | No single clock is trusted. Averaging many reduces error. |
| **Leap second** introduced from time to time | Earth's rotation is slowly getting slower, so **days are getting longer** compared with the atomic clock. A leap second is added to keep atomic time in step with the day/night cycle. |
| UTC is **broadcast** through **short-wave radio and satellite** | Machines with a receiver can get the time directly. |
| Satellites give accuracy of about **±0.5 ms** | A very accurate source for the reference. |

### Why not just give every computer a receiver?
Receivers cost money and need antennas or a sky view. So only a few machines get UTC directly, and the rest **ask them over the network**. That is exactly what Cristian's algorithm and NTP do.

---

## 5. Getting the Time from a Time Server (T1 to T4, θ and δ)

This slide is the **foundation** of Cristian's algorithm and NTP. Read it carefully.

### 5.1 The four timestamps

A client sends a request to a time server and gets a reply. Four moments are recorded:

| Symbol | Meaning | Measured on |
|---|---|---|
| **T1** | Client **sends** the request | Client's clock |
| **T2** | Server **receives** the request | Server's clock |
| **T3** | Server **sends** the response | Server's clock |
| **T4** | Client **receives** the response | Client's clock |

```
 Client :   T1 ───────────────────────────────────── T4
              \                                       ^
               \  request (delay ΔTreq)               |  response (delay ΔTres)
                v                                     |
 Server :        T2 ─── server processing time ─── T3
```

- **ΔTreq** = how long the request spent travelling = about T2 − T1 (plus clock difference).
- **ΔTres** = how long the response spent travelling = about T4 − T3 (plus clock difference).

### 5.2 The key assumption
> **ΔTreq ≈ ΔTres**, i.e., the request and response take about the same time (T2 − T1 ≈ T4 − T3 in terms of travel delay).

That is, the network is roughly **symmetric**. We cannot measure each direction separately because the clocks differ, so we assume they are equal. This is the main source of error.

### 5.3 The two formulas

```
 Offset θ  =  ( (T2 − T1) + (T3 − T4) ) / 2

 Delay  δ  =  ( (T4 − T1) − (T3 − T2) ) / 2
```

| Quantity | Meaning |
|---|---|
| **θ (offset)** | How far the **server's clock is ahead of the client's** (positive means server ahead). The client **adds θ** to its clock to correct it. |
| **δ (delay)** | The **one-way network delay** (round-trip time minus server processing, divided by 2). |

### 5.4 Where the formulas come from (derivation)
Let θ = server clock − client clock, and let the one-way delays be δreq and δres.

- Request arrives at the server: **T2 = T1 + δreq + θ** → T2 − T1 = δreq + θ
- Reply arrives at the client: **T4 = T3 − θ + δres** → T3 − T4 = θ − δres

Add the two lines:

```
(T2 − T1) + (T3 − T4)  =  2θ + (δreq − δres)  ≈  2θ        (since δreq ≈ δres)

        θ  =  ( (T2 − T1) + (T3 − T4) ) / 2
```

For the delay: total time at the client is T4 − T1. Subtract the time spent inside the server (T3 − T2). What remains is the time on the network in both directions, and half of it is the one-way delay:

```
        δ  =  ( (T4 − T1) − (T3 − T2) ) / 2
```

### 5.5 The slide's numerical example, step by step

| Event | Time |
|---|---|
| Client sends request (T1) | 10:00:00.000 |
| Server receives it (T2) | 10:00:00.080 |
| Server sends response (T3) | 10:00:00.081 |
| Client receives response (T4) | 10:00:00.161 |

(Read ".080" as 80 milliseconds after 10:00:00.)

**Step 1: Round-trip time (RTT) seen by the client**
RTT = T4 − T1 = 161 − 0 = **161 ms** ✔ (matches the slide)

**Step 2: One-way delay the simple way (Cristian-style)**
One-way delay ≈ RTT / 2 = 161 / 2 = **80.5 ms** ✔ (matches the slide)

**Step 3: Estimate the time**
The server's timestamp in its reply is **10:00:00.081**. That message travelled for about 80.5 ms, so the client estimates the true current time as

```
 10:00:00.081 + 80.5 ms = 10:00:00.1615
```

The client's own clock reads 10:00:00.161, so the client is about **0.5 ms behind**.

**What the last slide line means:** "Server timestamp = 10:00:00.161 − 80.5". The client received the reply at 161 ms on its own clock. The reply left the server about 80.5 ms earlier, i.e., at **about 80.5 ms on the client's clock**. But the server stamped it **81 ms**. So the server's clock is about 0.5 ms ahead, which gives the same conclusion (client about 0.5 ms behind).

**Step 4: Now the NTP formulas (using all four timestamps)**

```
 θ = ((T2 − T1) + (T3 − T4)) / 2
   = ((80 − 0) + (81 − 161)) / 2
   = (80 + (−80)) / 2
   = 0 ms

 δ = ((T4 − T1) − (T3 − T2)) / 2
   = (161 − 1) / 2
   = 80 ms
```

**Why do the two methods differ by 0.5 ms?**
The simple RTT/2 method ignores the **1 ms the server spent processing** (T3 − T2 = 81 − 80). It treats the entire 161 ms as network time (80.5 ms each way). The NTP formula **subtracts** the processing time, giving a one-way delay of exactly 80 ms and an offset of **0**. So in this example the two clocks are actually **perfectly in sync**; the 0.5 ms "error" in the simple method came only from ignoring server processing time.

| Method | One-way delay | Offset result |
|---|---|---|
| Simple (Cristian, RTT/2) | 80.5 ms | Client about 0.5 ms behind |
| NTP formula (θ, δ) | 80 ms | θ = 0 (clocks in sync) |

---

## 6. Cristian's Algorithm

### 6.1 The idea (slide)
Use a **time server** to synchronize clocks.

- The **time server** keeps the reference time (say, UTC).
- A **client** asks the time server for the time.
- The server **responds with its current time T**.
- The client uses the received value **T** to set its clock.

### 6.2 The problem: network delay (slide)
By the time T reaches the client, it is **already old**. The message spent time on the network. So simply setting the clock to T makes the client **late** by the transit time.

### 6.3 The improvement using RTT (slide)

- **RTT** = response-received-time − request-sent-time (the client can measure this on its own clock)
- Suppose we know **min** = the **minimum** one-way transmission time between client and server.
- Suppose the server timestamped the message at the **last possible instant** before sending it.

Then the reply travelled for **at least min** and **at most RTT − min**. (The request itself needed at least min, so at most RTT − min is left for the reply.) Therefore:

> **The actual time is in the range [ T + min , T + RTT − min ].**

### 6.4 Setting the clock and the error

- Best single estimate (midpoint): **client time = T + RTT/2**
- Width of the uncertainty range: (T + RTT − min) − (T + min) = RTT − 2·min
- So the **maximum error** is about **± (RTT/2 − min)**

```
 T+min                  T+RTT/2                  T+RTT−min
   |-----------------------|-------------------------|
   earliest possible      best guess            latest possible
                     (error at most RTT/2 − min)
```

**Key insight:** The **smaller the RTT, the better the accuracy**. A short round trip leaves little room for uncertainty.

### 6.5 Complete worked example *(beyond the slides: extra practice)*
Client sends a request and receives the reply 200 ms later. The reply carries T = 12:00:05.000. The minimum one-way time is 40 ms.

- RTT = 200 ms
- Best estimate: T + RTT/2 = 5.000 + 0.100 = **12:00:05.100**
- Range: [T + min, T + RTT − min] = [5.040, 5.160]
- Maximum error: RTT/2 − min = 100 − 40 = **±60 ms**

### 6.6 The slide's numerical problem, solved in full

> **Problem.** Consider a time server A. Server B requests the time from A at 10:37:640 (hr, min, 1/100 sec). At 10:37:710, server B receives a reply from time server A with a timestamp of 10:37:375. Find the drift of B's clock with respect to A's clock (assume no processing time at the time-server).

**How to read the times:** the last field counts hundredths of a second, so 640 means 6.40 s, 710 means 7.10 s and 375 means 3.75 s into the minute. We just work in units of 1/100 s.

| Item | Value |
|---|---|
| B sends request (local clock) | 640 |
| B receives reply (local clock) | 710 |
| Timestamp from A in the reply | 375 |

**Step 1: RTT** = reply received − request sent = 710 − 640 = **70** (1/100 s) = 0.70 s

> **Note on the slide's written solution:** it writes "610 − 540 = 70". Those are leftover numbers from an earlier version of the question. The difference is **also 70**, so the answer is unchanged. With the numbers printed in the question, use 710 − 640 = 70.

**Step 2: Adjusted local time** = server time + RTT/2 = 375 + 35 = **410** (1/100 s) = 4.10 s

**Step 3: Drift (offset)** = adjusted time − B's actual local time = 410 − 710 = **−300** (1/100 s) = **−3 seconds**

**Interpretation:**
- B's clock reads 7.10 s, but the time should be about 4.10 s.
- So **B's clock is 3 seconds ahead (fast)** of the server's clock.
- B must **set its clock back by 3 seconds** (or, better, slow it down until it matches; see the note below).

> **Terminology warning:** the slide calls the result "drift", but strictly it is the **offset (skew)** between the clocks at that moment. *Drift* is a rate (seconds per second). In exams, if the question uses the word "drift" the way this slide does, answer: **−3 s** (B is 3 s ahead).

### 6.7 Practical issues *(beyond the slides)*
- **Time must never run backwards.** If B's clock is ahead, setting it back could make a later event get an *earlier* timestamp (the original Fig 6-1 problem!). The fix is to **slow the clock down gradually** until it matches.
- **Single point of failure:** if the time server crashes, no one can sync.
- **Trust:** a faulty or malicious time server can give wrong time.
- **Improve accuracy:** make several requests and use the one with the **smallest RTT** (least uncertainty).

---

## 7. Network Time Protocol (NTP)

### 7.1 What it is (slide)
- The **most widely used physical clock synchronization protocol on the Internet**
- **10 to 20 million** NTP servers and clients on the Internet
- **Claimed accuracy (varies):** **milliseconds on WANs, sub-milliseconds on LANs**

### 7.2 Hierarchical tree of time servers (slide)

```
              Level 1   (most accurate, connected to UTC)
               /   \
         Level 2     Level 2
          /   \          \
   Level 3  Level 3    Level 3        (less accurate)
        ...clients at the lowest level...
```

| Level | Role (from the slide) |
|---|---|
| **Primary server at the root** | **Synchronizes with UTC** (directly connected to an accurate source) |
| **Secondary servers** | **Backup to the primary server**; they synchronize with the primary |
| **Lowest level: synchronization subnet with clients** | End machines (your laptops, servers) synchronize from the servers above |

**Rule:** accuracy gets **slightly worse at each step down** the tree, because every hop adds some network uncertainty. *(beyond the slides)* These levels are called **strata**: stratum 0 is the reference clock itself (atomic clock or GPS receiver), stratum 1 servers are directly attached to it, stratum 2 servers sync from stratum 1, and so on.

### 7.3 How NTP actually synchronizes
NTP uses the **four-timestamp exchange** from Section 5. A client exchanges messages with a server, computes the **offset θ** and **delay δ** with the two formulas, and adjusts its clock by θ (usually **gradually**, never jumping backwards). *(beyond the slides)* NTP also asks **several servers** and uses statistical filtering to ignore bad ones, which is why it is more accurate and more robust than a single Cristian request.

### 7.4 Why a tree?
- **Scalability:** millions of clients cannot all query a handful of atomic clocks. The tree spreads the load.
- **Fault tolerance:** secondary servers back up primaries.
- **Cost:** only the few top servers need expensive hardware.

### 7.5 Example
Your laptop asks `pool.ntp.org` for the time. That server is a stratum-2 (or stratum-3) machine that got its time from a stratum-1 server, which gets it from GPS or an atomic clock. Your laptop performs the T1 to T4 exchange and corrects its clock by θ.

---

## 8. The Berkeley Algorithm

### 8.1 The idea (slide)
- **One daemon (time daemon) without UTC.** It has no access to the real time.
- **Periodically**, this daemon **polls** all the machines and asks for their time.
- The machines **respond**.
- The daemon **computes an average time** and then **broadcasts** this average.

**Goal:** all machines agree **with each other** (internal synchronization). It does *not* try to match real time.

### 8.2 The slide's figure (Fig 6-7)

**(a) The daemon asks all other machines for their clock values.**

```
          Time daemon
             3:00
           /   |   \
        3:00  3:00  3:00        (daemon polls everyone)
        /      |       \
     2:50     3:00     3:25
   Machine A  (daemon)  Machine B
```

**(b) The machines answer** (each tells how far it is from the daemon):
- The daemon itself: **0**
- Machine A (2:50): **−10** (10 minutes behind the daemon's 3:00)
- Machine B (3:25): **+25** (25 minutes ahead)

### 8.3 Step by step calculation

**Step 1: Compute the average difference**

```
 average = (0 + (−10) + (+25)) / 3 = 15 / 3 = +5
```

**Step 2: Agreed time** = daemon's time + 5 = **3:05**

**Step 3: The daemon tells each machine how to adjust** (a correction, not a time value):

| Machine | Reads | Difference from daemon | Correction = average − difference | New time |
|---|---|---|---|---|
| Daemon | 3:00 | 0 | +5 − 0 = **+5** | 3:05 |
| Machine A | 2:50 | −10 | +5 − (−10) = **+15** | 3:05 |
| Machine B | 3:25 | +25 | +5 − 25 = **−20** | 3:05 |

All three machines now read **3:05**. (This is Fig 6-7 part (c) in the textbook; your slides show parts (a) and (b).)

### 8.4 Why send corrections instead of the new time?
A message containing "the time is 3:05" would be **out of date by the time it arrives** (network delay). A **relative correction** ("add 15 minutes") is **not damaged by delay** in the same way. *(beyond the slides)*

### 8.5 Details *(beyond the slides)*
- The daemon **estimates each machine's RTT** (like Cristian) so the readings are as accurate as possible.
- **Faulty clocks are ignored.** If a machine's value is wildly different, the daemon excludes it from the average so one broken clock cannot ruin everyone.
- A machine that must go **backward** (like Machine B, −20) should **slow its clock** gradually rather than jump back.
- If the daemon crashes, another machine can be **elected** as the new daemon.

### 8.6 Why it is "internal"
Notice the agreed time (3:05) is **not UTC**. Nobody checked the real time. The machines simply agree **with each other**. That is **precision**, not accuracy.

---

## 9. Physical Clock Algorithms Compared

| | Cristian's algorithm | NTP | Berkeley algorithm |
|---|---|---|---|
| **Type** | External | External | **Internal** |
| **Needs UTC source?** | Yes (the server has it) | Yes (root of the tree has it) | **No** |
| **Who initiates?** | **Client** asks server | Client asks servers (hierarchy) | **Daemon polls** the machines |
| **Structure** | One time server | **Hierarchical tree** of servers | One master (daemon) and followers |
| **How time is computed** | Server time + RTT/2 | θ and δ from 4 timestamps, filtered over servers | **Average** of all clocks |
| **What is sent back** | Server timestamp | Timestamps | **Correction** values |
| **Main weakness** | Single server; RTT error | Complex; accuracy decreases with depth | Daemon is a single point; no real-time reference |
| **Scale** | Small | **Internet-wide (millions)** | Small cluster |
| **Typical use** | Simple LAN setups | The Internet | A group of machines needing mutual agreement |

---

# PART C: LOGICAL TIME

## 10. Order Matters More Than Time: Happened-Before

### 10.1 The issue (slide)
> What usually matters is **not** that all processes agree on exactly **what time it is**, but that they agree on the **order in which events occur**. This requires a **notion of ordering**.

**Example:** In a chat app, you do not care whether a message was sent at 10:03:02.411 or 10:03:02.412. You care that the **reply appears after the question**.

This idea (from Leslie Lamport, 1978) lets us avoid physical clocks altogether when only order matters.

### 10.2 The "happens-before" relation (→)

The slide gives two situations in which a → b ("a happens before b") can be **observed directly**:

| Rule | Statement | Example |
|---|---|---|
| **Rule 1: same process** | If a and b are events in the **same process** and a occurs before b, then **a → b** | In P1: "read the file" then "send the result" |
| **Rule 2: message** | If a is the event of a message being **sent** by one process and b is the event of that message being **received** by another process, then **a → b** | P1 sends m (event a), P2 receives m (event b), so a → b |

**Rule 3: transitivity** *(beyond the slides, but always part of the definition)*
If a → b and b → c, then **a → c**.

### 10.3 Why a message send must come before its receive
A message cannot be received before it is sent. That is a **physical fact that needs no clock**. So we can use it as a trustworthy ordering link.

### 10.4 Concurrent events
If **neither** a → b **nor** b → a, then a and b are **concurrent** (written a ∥ b). This does **not** mean they happened at the same instant. It means **there is no chain of causality** connecting them, so nobody can say which came first.

**Example:** P1 and P3 each do something on their own and never exchange messages. Those events are concurrent.

### 10.5 Worked example with a diagram

```
 P1:   a ───── b (send m1) ─────────── c
                    \
                     v
 P2:   d ───────── e (receive m1) ─── f (send m2)
                                          \
                                           v
 P3:   g ─────────────────────────────── h (receive m2)
```

| Relation | True? | Reason |
|---|---|---|
| a → b | Yes | Rule 1 (same process P1) |
| b → e | Yes | Rule 2 (send m1, receive m1) |
| e → f | Yes | Rule 1 (same process P2) |
| f → h | Yes | Rule 2 (send m2, receive m2) |
| b → h | Yes | b → e → f → h (transitivity) |
| a → h | Yes | a → b → e → f → h |
| c and e | **Concurrent** | No chain links them: c is after b in P1, e is on P2, and no message goes from c to e |
| d and b | **Concurrent** | d (P2) and b (P1) are unrelated |
| g and a | **Concurrent** | No messages between P3's g and P1's a |

---

## 11. Lamport's Logical Clocks

### 11.1 The goal
Give every event a **number** (timestamp) so that:

> **If a → b, then the timestamp of a is smaller than the timestamp of b.**

No physical clock is needed. Each process keeps a simple **counter**.

### 11.2 The algorithm (slide: "Lamport's Logical Clocks (5)")
Each process **Pᵢ** keeps a counter **Cᵢ**.

| Step | Rule | In plain words |
|---|---|---|
| **1** | **Before executing an event**, Pᵢ executes **Cᵢ ← Cᵢ + 1** | Every event (including send and receive) bumps the counter by 1 |
| **2** | When Pᵢ **sends** message m to Pⱼ, it sets **ts(m) = Cᵢ** (after step 1) | The message carries the sender's counter value as its timestamp |
| **3** | On **receipt** of m, Pⱼ sets **Cⱼ ← max{Cⱼ, ts(m)}**, then executes step 1 (Cⱼ ← Cⱼ + 1), then **delivers the message to the application** | The receiver jumps ahead if it was behind, then counts the receive event |

### 11.3 Why "max" and then "+1"?
- **max** guarantees the receiver's clock is **at least** the sender's timestamp (never behind it).
- **+1** guarantees the receive event's timestamp is **strictly greater** than the send timestamp.
- Together: **receive timestamp > send timestamp** always. That is exactly the requirement for rule 2 of happened-before.

### 11.4 A small example with the simplest ticks (+1 per event)

```
 P1:  a(1) ── b(2) send m1 ──────────── c(3)
                     \
                      v   ts(m1)=2
 P2:  d(1) ────────── e(?)  ──── f(?) send m2
                                    \
                                     v   ts(m2)=?
 P3:  g(1) ────────────────────────── h(?) ── i(?)
```

Compute step by step:

| Event | Calculation | Counter |
|---|---|---|
| a (P1) | 0 + 1 | **1** |
| b (P1, send m1) | 1 + 1 | **2**, so ts(m1) = 2 |
| c (P1) | 2 + 1 | **3** |
| d (P2) | 0 + 1 | **1** |
| e (P2, receive m1) | max(1, 2) = 2, then +1 | **3** |
| f (P2, send m2) | 3 + 1 | **4**, so ts(m2) = 4 |
| g (P3) | 0 + 1 | **1** |
| h (P3, receive m2) | max(1, 4) = 4, then +1 | **5** |
| i (P3) | 5 + 1 | **6** |

Check: b(2) → e(3) ✔, f(4) → h(5) ✔. Every cause has a smaller timestamp than its effect.

### 11.5 The slide's figure: Fig 6-9 (a), three processes with clocks at different rates

Each process has its **own clock ticking at a different speed**:

| Process | Ticks by |
|---|---|
| P1 | 6 each step (0, 6, 12, 18, ...) |
| P2 | 8 each step (0, 8, 16, 24, ...) |
| P3 | 10 each step (0, 10, 20, 30, ...) |

Four messages (arrows):

| Message | From → To | Sent when sender's clock reads | Arrives when receiver's clock reads | Problem? |
|---|---|---|---|---|
| m1 | P1 → P2 | 6 | 16 | No (16 > 6) |
| m2 | P2 → P3 | 24 | 40 | No (40 > 24) |
| **m3** | P3 → P2 | **60** | **56** | **Yes! Arrives "before" it was sent (56 < 60)** |
| **m4** | P2 → P1 | **64** | **54** | **Yes! Arrives "before" it was sent (54 < 64)** |

This is impossible in reality, so the clocks are inconsistent. The caption: *"The clocks run at different rates."*

### 11.6 The slide's figure: Fig 6-9 (b), Lamport's algorithm corrects the clocks

Apply the rules to fix the bad arrivals.

**When P2 receives m3 (ts = 60):**
- P2's counter was 56.
- C₂ ← max(56, 60) = 60, then +1 → **61** (circled "61" on the slide; "P2 adjusts its clock").
- P2's later ticks continue at +8 each: 61, **69**, 77, 85.

**When P1 receives m4:**
- P2 sent m4 at 69 (the *corrected* clock), so ts(m4) = **69**.
- P1's counter was 54 (it had advanced 48 → 54).
- C₁ ← max(54, 69) = 69, then +1 → **70** (circled "70"; "P1 adjusts its clock").
- P1's later ticks continue at +6 each: 70, 76, ...

**After correction:**

| Message | Sent at | Received at | OK? |
|---|---|---|---|
| m3 | 60 | 61 | ✔ |
| m4 | 69 | 70 | ✔ |

The key message: **Lamport's algorithm only ever moves a clock forward, and only when needed, to restore the rule "receive > send".** It does not need clocks to run at the same speed (6, 8 and 10 per tick is fine).

### 11.7 Where the algorithm lives: Fig 6-10 (positioning in the system)
The Lamport logic is **not** in the application. It sits in a **middleware layer** between the application and the network:

```
          APPLICATION LAYER
   Application sends message      Message is delivered to application
              |                                 ^
              v                                 |
   +---------------------------+   +---------------------------+
   | Adjust local clock        |   | Adjust local clock        |    MIDDLEWARE
   | and timestamp message     |   |                           |    LAYER
   +---------------------------+   +---------------------------+
              |                                 ^
   Middleware sends message          Message is received
              v                                 |
                      NETWORK LAYER
```

| Direction | What the middleware does |
|---|---|
| **Sending** | Takes the message from the application, **increments the clock and stamps the message** with it, then hands it to the network |
| **Receiving** | Takes the message from the network, **adjusts the local clock** (max, then +1), then delivers it to the application |

**Benefit:** the application does not need to know anything about logical clocks. It is handled transparently.

### 11.8 Properties and limits of Lamport clocks

| Property | True? |
|---|---|
| If a → b then C(a) < C(b) | **Yes (guaranteed)** |
| If C(a) < C(b) then a → b | **No (not guaranteed)** |
| Can tell when two events are concurrent | **No** |

**Why the converse fails (example):** In Section 11.4, event d on P2 has timestamp 1, and event b on P1 has timestamp 2. So C(d) < C(b), yet d and b are **concurrent** (no causal path). A smaller number does **not** prove that "it happened first" or that "it caused the other". It just means "it is *consistent* with some valid ordering".

**Remember:** Lamport timestamps tell you "if a caused b, then C(a) < C(b)". They **cannot** tell you whether two events with different timestamps are related or concurrent. For that you need **vector clocks**.

---

## 12. Partial Order vs Total Order

### 12.1 The slide's statement
> Partial order logical clocks assign timestamps to distributed events to create a **causal, but not necessarily total, ordering**, capturing the "happened-before" relationship without relying on physical clocks. This establishes that one event **could have caused** another, but **not all events can be compared** (some events are **concurrent**). **Lamport's algorithm uses simple scalar timestamps for a partial ordering, while vector clocks provide a more complete ordering, enabling detection of concurrent events.**

### 12.2 Plain-language meaning

| Term | Meaning | Analogy |
|---|---|---|
| **Total order** | **Every** pair of events can be put in an order | A queue of people numbered 1, 2, 3, ... Everyone has a place relative to everyone else |
| **Partial order** | **Some** pairs can be ordered, others cannot (they are concurrent) | A family tree. A grandfather is "before" his grandchild, but two cousins are not before or after each other |

Happened-before (→) is a **partial order**: if two events have no causal chain, they are simply **unordered (concurrent)**.

### 12.3 How to read the slide's last sentence correctly
- **Lamport (scalar) timestamps** are **consistent with** the partial order of causality (if a → b then C(a) < C(b)), but a single number **cannot reveal** which pairs are concurrent.
- **Vector clocks** **fully capture** the partial order: from two vector timestamps you can tell **before, after, or concurrent**.

*(beyond the slides)* A subtle point: if two events have the **same** Lamport timestamp from different processes, you can **break the tie with the process ID**, giving every event a unique (timestamp, process ID) pair. That **extends** the partial order into an (artificial) **total order**, which is exactly what totally ordered multicast needs (next section). The extra order for concurrent events is arbitrary but **identical on every machine**, which is what matters.

For exams, use the slide's wording: *Lamport = scalar, partial ordering; vector clocks = detect concurrency.*

---

## 13. Totally Ordered Multicasting

### 13.1 The problem: replicated database (Fig 6-11)
A database is **replicated** on two (or more) servers for speed and fault tolerance. Two users send updates at the same time from different places.

```
   User 1 ── Update 1 ───────────────┐
        \                            v
         \──────► Replica 1 (left)    Replica 2 (right)
                       ^                     ^
   User 2 ── Update 2 ─┘ (dashed)  ──────────┘

   Left replica:  Update 1 performed BEFORE Update 2
   Right replica: Update 2 performed BEFORE Update 1
```

The network delivers updates in different orders at different replicas. Caption: *"Updating a replicated database and leaving it in an inconsistent state."*

### 13.2 Why the order matters: a concrete example (textbook example)
A bank account holds **$1,000**.
- **Update 1:** deposit **$100**.
- **Update 2:** add **1% interest**.

| Replica | Order applied | Calculation | Final balance |
|---|---|---|---|
| Left | Update 1, then Update 2 | (1000 + 100) × 1.01 | **$1,111** |
| Right | Update 2, then Update 1 | (1000 × 1.01) + 100 | **$1,110** |

The replicas now **disagree**, which is **inconsistent**. The same two updates produced different data because they ran in a different order.

### 13.3 The goal: totally ordered multicast
> **All replicas must apply all updates in exactly the same order.**

It does not matter whether the order is "Update 1 first" or "Update 2 first", as long as **everyone uses the same one**.

### 13.4 Solution using Lamport clocks *(beyond the slide text; this is the standard method that the figure is introducing)*

1. Each update message is **timestamped with the sender's Lamport clock** (use (timestamp, process ID) to break ties).
2. The message is **multicast to everyone**, including the sender itself.
3. Every process puts each received message in a **local queue sorted by timestamp**.
4. On receiving a message, a process **multicasts an acknowledgement** to everyone.
5. A message is **delivered to the application** only when it is **at the head of the queue** and has been **acknowledged by all processes**.

**Why this works:** because every process eventually sees the same set of messages with the same timestamps, every queue ends up sorted the same way, so every replica applies updates in the same order.

*(Assumptions: messages from one sender arrive in the order sent, and no message is lost.)*

### 13.5 Example with the numbers
- P1 sends Update 1 (+$100) with timestamp **(1, P1)**.
- P2 sends Update 2 (+1% interest) with timestamp **(1, P2)**. These are concurrent, with equal counters.
- The tie-break uses the process ID, so **(1, P1) < (1, P2)**.
- Every replica orders them: **Update 1, then Update 2**.
- Both replicas compute (1000 + 100) × 1.01 = **$1,111**. ✔ Consistent.

---

## 14. Vector Clocks

### 14.1 Why Lamport clocks are not enough (Fig 6-12)
Fig 6-12 shows five messages using **ordinary (scalar) logical clocks**:

| Message | From → To | Timestamp (sender's clock at send) |
|---|---|---|
| m1 | P1 → P2 | 6 |
| m2 | P3 → P2 | 20 |
| m3 | P2 → P3 | 32 |
| m4 | P3 → P2 | 60 |
| m5 | P2 → P1 | 69 |

Look at m1 (timestamp 6) and m2 (timestamp 20). Since 6 < 20, you might think "m1 was sent before m2, and may have caused it". But **P1 and P3 never communicated before sending them**. m1 and m2 are actually **concurrent** (unrelated).

Now m3 is sent by P2 *after* receiving both m1 and m2, so m3 **may depend on both**. With scalar clocks you **cannot tell** which of these relationships are real and which are accidental. The caption: *"Concurrent message transmission using logical clocks."*

**Conclusion:** one number per process cannot capture "who has seen what". We need each process to keep **knowledge about every other process**: a **vector**.

### 14.2 The structure (slide: "Vector Clocks (2)")
Each process **Pᵢ** maintains a **vector VCᵢ** with one slot per process:

| Property | Meaning | Example (3 processes) |
|---|---|---|
| **1.** VCᵢ[i] = number of events that have occurred so far at **Pᵢ** | The **local logical clock** of Pᵢ | In P1's vector, slot 1 counts P1's own events |
| **2.** If VCᵢ[j] = k, then **Pᵢ knows that k events have occurred at Pⱼ** | Pᵢ's **knowledge of the local time at Pⱼ** | If VC₃ = (2, 3, 1), P3 knows P1 has had at least 2 events, P2 at least 3, and P3 itself 1 |

**Analogy:** Each process keeps a **notebook** with a line for every process: "Here is what I know about how far each person has got."

### 14.3 The three rules (slide: "Vector Clocks (3)")

| Step | Rule | In plain words |
|---|---|---|
| **1** | Before executing an event, **Pᵢ executes VCᵢ[i] ← VCᵢ[i] + 1** | Count my own event (only **my own** slot goes up) |
| **2** | When **Pᵢ sends** message m to Pⱼ, it sets **ts(m) = VCᵢ** (after step 1) | Attach the **whole vector** to the message |
| **3** | On **receipt** of m, **Pⱼ sets VCⱼ[k] ← max{ VCⱼ[k], ts(m)[k] } for every k**, then executes step 1, then delivers the message | Merge: for each slot keep the **larger** value, then count the receive event in my own slot |

**Compare with Lamport:** same ideas (increment, stamp, max on receive), but applied **slot by slot** to a vector instead of to a single number.

### 14.4 The slide's handwritten diagram: complete trace

The slide shows three processes P1, P2, P3 with vectors written as (P1's count, P2's count, P3's count), and four messages m1 to m4. The handwritten notes at the top show P1 starting at (0,0,0). Here is the **complete, verified trace**. Every vector below matches the printed values on the slide.

```
 P1:   (1,1,0)   (2,1,0)   (3,1,0)   (4,1,0)
          ^           \         •         \
          |            \m2               \m3
          |m1           \                  \
 P2:   (0,1,0)           \              (4,2,0)   (4,3,0)
                           \                          \
                            \                          \m4
 P3:                      (2,1,1)                     (4,3,2)
```

Start: all vectors are (0,0,0).

| # | Process | Event | Calculation | New vector |
|---|---|---|---|---|
| 1 | P2 | **Send m1** | Own slot (2nd): 0 → 1 | **(0,1,0)**, so ts(m1) = (0,1,0) |
| 2 | P1 | **Receive m1** | Merge: max((0,0,0),(0,1,0)) = (0,1,0); then own slot (1st) +1 | **(1,1,0)** ✔ |
| 3 | P1 | **Send m2** | Own slot +1 → 2 | **(2,1,0)**, so ts(m2) = (2,1,0) |
| 4 | P3 | **Receive m2** | Merge: max((0,0,0),(2,1,0)) = (2,1,0); then own slot (3rd) +1 | **(2,1,1)** ✔ |
| 5 | P1 | **Internal event** (the black dot) | Own slot +1 → 3 | **(3,1,0)** ✔ |
| 6 | P1 | **Send m3** | Own slot +1 → 4 | **(4,1,0)**, so ts(m3) = (4,1,0) |
| 7 | P2 | **Receive m3** | Merge: max((0,1,0),(4,1,0)) = (4,1,0); then own slot (2nd) +1 | **(4,2,0)** ✔ |
| 8 | P2 | **Send m4** | Own slot +1 → 3 | **(4,3,0)**, so ts(m4) = (4,3,0) |
| 9 | P3 | **Receive m4** | Merge: max((2,1,1),(4,3,0)) = (4,3,1); then own slot +1 | **(4,3,2)** ✔ |

The handwritten arrows "(0,0,0) → (1,1,0) → (2,1,0)" on the slide are the instructor tracing **P1's own progression**: the start, then the receive of m1, then the send of m2. The other handwritten fragments (such as "VC₂[...]" and "VC₁[...] = (2,1,1)") are partial workings of the **max step**, which is the merge computed in rows 4, 7 and 9 above.

### 14.5 Comparing two vectors (how to detect order and concurrency) *(beyond the slides, but this is the reason vector clocks exist)*

Let V and W be the vectors of two events.

| Relation | Condition | Meaning |
|---|---|---|
| **V ≤ W** | **Every** component of V ≤ the same component of W | V is not ahead of W anywhere |
| **V < W** | V ≤ W **and** V ≠ W | **Event V happened before event W** (V → W) |
| **Concurrent** | **Neither** V ≤ W **nor** W ≤ V (some component bigger in V, some bigger in W) | The events are **independent** |

**This is the big advantage: V < W holds if and only if V → W.** (For Lamport clocks only one direction holds.)

### 14.6 Applying it to the slide's diagram

| Compare | Vectors | Result | Reason |
|---|---|---|---|
| send(m1) at P2 vs send(m2) at P1 | (0,1,0) vs (2,1,0) | **send(m1) → send(m2)** | (0,1,0) ≤ (2,1,0) in every slot. P1 received m1 before sending m2. |
| receive(m2) at P3 vs receive(m4) at P3 | (2,1,1) vs (4,3,2) | **before** | Every slot ≤, same process |
| send(m3) at P1 vs receive(m2) at P3 | (4,1,0) vs (2,1,1) | **Concurrent** | 4 > 2 in slot 1 but 0 < 1 in slot 3. Neither is ≤ the other |
| internal dot at P1 vs receive(m2) at P3 | (3,1,0) vs (2,1,1) | **Concurrent** | 3 > 2 but 0 < 1 |
| receive(m1) at P1 vs receive(m3) at P2 | (1,1,0) vs (4,2,0) | **before** | (1,1,0) ≤ (4,2,0) everywhere |

### 14.6b Intuition for why concurrency shows up
Each slot of the vector says "how many events of that process have influenced me". If my vector is larger in slot 1 but yours is larger in slot 3, then I have seen things you have not, and you have seen things I have not. **Neither of us knows everything the other knows**, so neither event can be the cause of the other. That is concurrency.

### 14.7 Cost of vector clocks *(beyond the slides)*
- A vector has **one entry per process**, so messages become bigger as the system grows (N processes means N numbers).
- Lamport clocks add only **one number** per message. This is the trade-off: **more information versus more space**.

---

## 15. Lamport Clocks vs Vector Clocks

| Feature | Lamport (scalar) clock | Vector clock |
|---|---|---|
| **Timestamp** | One integer per process | A vector of N integers |
| **Update on send** | Counter +1, attach it | Own slot +1, attach whole vector |
| **Update on receive** | max(own, ts), then +1 | Slot-by-slot max, then own slot +1 |
| **a → b implies ordering of timestamps?** | **Yes**: C(a) < C(b) | **Yes**: VC(a) < VC(b) |
| **Timestamp order implies a → b?** | **No** | **Yes** (V < W ⇔ a → b) |
| **Can detect concurrent events?** | **No** | **Yes** |
| **Message size** | Small | Grows with number of processes |
| **Typical use** | Totally ordered multicast, simple ordering | Conflict detection in replicated data, causal ordering |
| **Slide wording** | "Simple scalar timestamps for a partial ordering" | "More complete ordering, enabling detection of concurrent events" |

**Quick decision rule:**
- Only need **a consistent order** that respects causality, and cheap messages → **Lamport**.
- Need to know whether two events are **related or concurrent** → **Vector clocks**.

---

## 16. Global Snapshots (Brief)

*(The "Global Time and Global State" slide lists "Simulate a global state – Global Snapshots", but the slides do not go further. This section is **beyond the slides** so that the term is not a loose end.)*

### 16.1 The problem
Can we capture a **consistent picture of the entire distributed system** (every process's state **plus** all messages currently in transit) without stopping it?

**Why hard:** you cannot photograph all machines at the same instant (no global clock). If you record each machine at a slightly different moment, the picture can be **inconsistent**.

**Example of an inconsistent snapshot:** a bank transfer of $100 from account A (on machine 1) to account B (on machine 2). If machine 1 records its state **after** debiting A, but machine 2 records **before** the money arrives, the snapshot shows $100 has **vanished**.

### 16.2 The Chandy-Lamport snapshot idea (marker algorithm)
1. The **initiator** records its own local state and sends a special **marker** message on all its outgoing channels.
2. When a process receives a marker for the **first time**, it:
   - records its own state,
   - records that channel as empty,
   - sends markers on all its outgoing channels.
3. After that, it **records any normal messages** arriving on each incoming channel **until a marker arrives on that channel**. Those recorded messages are the ones **in transit**.
4. The snapshot = **all recorded process states + all recorded in-transit messages**.

**Result:** a **consistent** global state (the $100 would be counted either in A's state, in B's state, or as an in-transit message, but never lost or counted twice).

**Uses:** detecting deadlocks, checkpointing/rollback recovery, garbage collection, debugging distributed programs.

---

# PART D: REVISION TOOLS

## 17. Where This Shows Up in Real Cloud Systems

*(beyond the slides)*

| Concept | Where you meet it |
|---|---|
| **NTP** | Almost every server, VM and phone uses NTP to keep time. Cloud providers run their own NTP services for VMs. |
| **Clock skew problems** | Log analysis across servers (events appear out of order), certificate validation errors ("certificate not yet valid"), cache expiry, scheduled-job misfires. |
| **UTC / GPS / atomic clocks** | Large providers run their own time infrastructure. Google Spanner's **TrueTime** exposes time as an **interval** [earliest, latest] (the same idea as Cristian's uncertainty range) and waits out the uncertainty to order transactions. |
| **Lamport clocks / total ordering** | Replicated state machines, distributed databases, and consensus protocols all need all replicas to agree on a single order of operations. |
| **Vector clocks / version vectors** | Used in some replicated key-value stores (for example Amazon's Dynamo-style systems) to detect **conflicting concurrent updates** to the same item. |
| **Snapshots** | Checkpointing of large distributed jobs, consistent backups of distributed databases. |

---

## 18. Formula Sheet

| Item | Formula |
|---|---|
| **RTT** | T4 − T1 (client receive − client send) |
| **Offset θ** (server ahead of client) | θ = ((T2 − T1) + (T3 − T4)) / 2 |
| **Delay δ** (one-way) | δ = ((T4 − T1) − (T3 − T2)) / 2 |
| **Client correction** | New client time = old client time + θ |
| **Cristian's estimate** | Client time = T + RTT/2 |
| **Cristian's range** | [ T + min , T + RTT − min ] |
| **Cristian's max error** | ± ( RTT/2 − min ) |
| **Slide example result** | If adjusted local = server + RTT/2, then offset = adjusted − local |
| **Berkeley average** | avg = (sum of differences from daemon, including 0 for itself) / number of machines |
| **Berkeley correction for machine i** | correction = avg − difference_i |
| **Resync interval** | t = δ / (2ρ) |
| **Lamport step 1** | Cᵢ ← Cᵢ + 1 (before every event) |
| **Lamport send** | ts(m) = Cᵢ |
| **Lamport receive** | Cⱼ ← max{ Cⱼ , ts(m) }, then Cⱼ ← Cⱼ + 1 |
| **Vector step 1** | VCᵢ[i] ← VCᵢ[i] + 1 |
| **Vector send** | ts(m) = VCᵢ |
| **Vector receive** | VCⱼ[k] ← max{ VCⱼ[k] , ts(m)[k] } for all k, then VCⱼ[j] ← VCⱼ[j] + 1 |
| **Vector compare** | V < W iff all V[k] ≤ W[k] and V ≠ W (this means V → W). If neither V ≤ W nor W ≤ V, they are concurrent |

---

## 19. Common Confusions Cleared

**Q1. What is the difference between clock skew and clock drift?**
**Skew** = difference in the clocks' **readings** at an instant (a value in seconds). **Drift** = difference in the clocks' **speeds** (a rate, seconds per second). Drift makes skew grow over time.

**Q2. What is the difference between precision and accuracy?**
**Precision** = clocks agree **with each other**. **Accuracy** = clocks agree **with real time (UTC)**. Internal synchronization targets precision; external synchronization targets accuracy.

**Q3. Which algorithms are internal and which are external?**
**Berkeley = internal** (no UTC). **Cristian and NTP = external** (they use a UTC source).

**Q4. Why does Cristian's algorithm divide RTT by 2?**
Because we assume the request and response took equal time, so the reply's journey was about half the round trip.

**Q5. Why is the NTP delay formula different from RTT/2?**
NTP **subtracts the server's processing time** (T3 − T2) before halving, so it is more accurate when the server takes time to respond.

**Q6. In the slide's example the offset θ = 0, but the simple method said 0.5 ms behind. Which is right?**
The NTP result (θ = 0) is more precise because it accounts for the 1 ms server processing time. The 0.5 ms difference came from lumping processing time into network delay.

**Q7. In the slide problem (10:37:640 ...), is the answer +3 s or −3 s?**
**−3 seconds** (adjusted 410 − local 710 = −300 hundredths). It means B's clock is **3 seconds ahead**, so B must go back by 3 s.

**Q8. Why does the Berkeley algorithm send corrections and not the new time?**
A correction is not made stale by message delay, while an absolute time would be.

**Q9. In Berkeley, why is the daemon's own clock counted in the average?**
The daemon is one of the machines being synchronized, so its difference (0) is part of the average. That is why the slide's answer divides by 3.

**Q10. Is a Lamport timestamp a real time?**
**No.** It is just a counter that respects causality. It has no relation to seconds or the wall clock.

**Q11. If C(a) < C(b) does it mean a happened before b?**
**Not necessarily.** Only the reverse is guaranteed: a → b implies C(a) < C(b).

**Q12. Why add 1 after the max on receive?**
So that the receive event gets a timestamp **strictly larger** than the send timestamp. Otherwise they could be equal.

**Q13. Do clocks need to tick at the same speed for Lamport's algorithm?**
**No.** Fig 6-9 uses ticks of 6, 8 and 10. The algorithm adjusts forward whenever the rule "receive > send" is violated.

**Q14. Can a Lamport clock ever go backwards?**
**No.** It only increases (by 1 per event, or jumps forward by max).

**Q15. What does "concurrent" mean?**
Not "at the same time". It means **no causal chain** connects the events, so neither could have influenced the other.

**Q16. Why does totally ordered multicast need the tie-break by process ID?**
Two concurrent events from different processes can have the **same** Lamport timestamp. Using (timestamp, process ID) makes every message's position unique and identical on every replica.

**Q17. Why do the replicas end up different in Fig 6-11?**
Because operations like "+$100" and "+1%" are **not commutative** (order changes the result), and the network delivered them in different orders to the two replicas.

**Q18. How does a vector clock differ in the "receive" rule from a Lamport clock?**
Lamport takes max of **one** number. A vector clock takes the max **slot by slot** across all N entries, and increments **only the receiver's own slot**.

**Q19. What does a vector entry VCᵢ[j] = k mean?**
Process Pᵢ knows that **k events have occurred at Pⱼ**.

**Q20. Why can't we just use NTP to get perfect global time and skip logical clocks?**
NTP is accurate only to milliseconds, not perfect. Events can happen faster than that, and message delays are unpredictable, so physical timestamps can still misorder events. Logical clocks give **exact ordering by causality** without depending on clock accuracy.

**Q21. Is the Berkeley algorithm the same as NTP?**
No. Berkeley **averages** the machines' own clocks with no UTC (internal). NTP **pulls time from UTC-connected servers** in a hierarchy (external).

**Q22. Why is the root of the NTP tree "more accurate"?**
It is directly connected to a UTC source. Each step down the tree adds network uncertainty, so lower levels are slightly less accurate.

---

## 20. Glossary

| Term | Meaning |
|---|---|
| **Accuracy** | How close a clock is to real time (UTC), within bound α. |
| **Asynchronous system** | A system with no known bound on message delay or process speed. |
| **Berkeley algorithm** | Internal synchronization where a daemon polls machines and broadcasts the average-based correction. |
| **Causal ordering** | Ordering of events that respects cause and effect. |
| **Cesium-133** | The atom whose oscillations define the atomic second. |
| **Clock drift** | Difference in the **rates** at which two clocks run. |
| **Clock skew** | Difference in the **readings** of two clocks at an instant. |
| **Concurrent events** | Events with no happened-before relation in either direction. |
| **Cristian's algorithm** | External synchronization using a time server and RTT/2. |
| **Daemon** | A background program (here, the Berkeley time coordinator). |
| **Delay δ** | One-way network delay computed from the four timestamps. |
| **External synchronization** | Keeping clocks accurate against a real-time reference. |
| **Global snapshot** | A consistent record of all process states and in-transit messages. |
| **Global state** | The combined state of all processes and channels at one logical instant. |
| **Happened-before (→)** | Lamport's partial order: same-process order plus send-before-receive plus transitivity. |
| **Internal synchronization** | Keeping clocks mutually consistent (precise), without real time. |
| **Lamport clock** | A scalar logical clock that respects happened-before. |
| **Leap second** | An extra second added to UTC to keep it aligned with Earth's rotation. |
| **Logical clock** | A counter that orders events without measuring real time. |
| **Middleware** | Software layer between application and network where logical clocks are managed. |
| **min (in Cristian)** | The minimum one-way message transmission time. |
| **NTP** | Network Time Protocol, the Internet's hierarchical time synchronization system. |
| **Offset θ** | Difference between the server's and the client's clocks. |
| **Partial order** | An ordering in which some pairs of elements are incomparable. |
| **Physical clock** | A clock that measures real time. |
| **Precision** | How closely clocks agree with each other. |
| **Replicated database** | A database with copies on several machines. |
| **RTT** | Round-trip time: time from sending a request to receiving the reply. |
| **Scalar timestamp** | A single-number timestamp (Lamport). |
| **Stratum** | A level in the NTP hierarchy. |
| **Time server** | A machine that provides a reference time to others. |
| **Total order** | An ordering in which every pair of events is comparable. |
| **Totally ordered multicast** | Multicast in which all processes deliver all messages in the same order. |
| **Transitivity** | If a → b and b → c then a → c. |
| **UTC** | Universal Coordinated Time, the world's reference time based on atomic clocks. |
| **Vector clock** | A logical clock with a vector of counters, one per process; detects concurrency. |
| **WAN / LAN** | Wide area network / local area network. |

---

## 21. Practice Problems with Full Solutions

### Problem 1: Cristian's algorithm
A client sends a request at its local time 12:00:00.000 and receives the reply at its local time 12:00:00.200. The reply carries the server time 12:00:05.000. The minimum one-way delay is 40 ms. (a) What time should the client set? (b) By how much must it adjust? (c) What is the range of the true time and the maximum error?

**Solution**
- RTT = 200 ms.
- (a) Set the clock to T + RTT/2 = 5.000 + 0.100 = **12:00:05.100**.
- (b) The client's own clock reads 12:00:00.200 at that moment, so adjust by 5.100 − 0.200 = **+4.900 s (forward)**.
- (c) Range: [T + min, T + RTT − min] = [5.040, 5.160] s. Maximum error = RTT/2 − min = 100 − 40 = **±60 ms**.

---

### Problem 2: Offset and delay with four timestamps
T1 = 100 ms, T2 = 160 ms, T3 = 162 ms, T4 = 122 ms (T1 and T4 on the client clock, T2 and T3 on the server clock). Find θ and δ, and say which clock is ahead.

**Solution**
- θ = ((T2 − T1) + (T3 − T4)) / 2 = ((160 − 100) + (162 − 122)) / 2 = (60 + 40) / 2 = **50 ms**
- δ = ((T4 − T1) − (T3 − T2)) / 2 = ((122 − 100) − (162 − 160)) / 2 = (22 − 2) / 2 = **10 ms**
- θ is positive, so the **server is 50 ms ahead**. The client should **add 50 ms** to its clock.
- Check: T2 = T1 + δ + θ = 100 + 10 + 50 = 160 ✔; T3 = T4 − δ + θ = 122 − 10 + 50 = 162 ✔.

---

### Problem 3: Berkeley algorithm
The time daemon reads 3:00. Machine A reads 3:10, Machine B reads 2:48, Machine C reads 3:06. Find the corrections.

**Solution (in minutes)**
- Differences from the daemon: daemon 0, A **+10**, B **−12**, C **+6**.
- Average = (0 + 10 − 12 + 6) / 4 = 4 / 4 = **+1**. Agreed time = **3:01**.
- Corrections (average − difference): daemon **+1**, A **1 − 10 = −9**, B **1 − (−12) = +13**, C **1 − 6 = −5**.
- Check: A 3:10 − 9 = 3:01 ✔; B 2:48 + 13 = 3:01 ✔; C 3:06 − 5 = 3:01 ✔.

---

### Problem 4: The slide's problem (a variation to practice)
Server B sends a request at local time 500 (1/100 s). The reply arrives at B's local time 540 with the server timestamp 900. Find B's offset from the time server.

**Solution**
- RTT = 540 − 500 = 40. RTT/2 = 20.
- Adjusted time = 900 + 20 = 920.
- Offset = adjusted − local = 920 − 540 = **+380** (1/100 s) = **+3.8 s**.
- B's clock is **3.8 s behind**; it must move forward by 3.8 s.

---

### Problem 5: Lamport clocks
Use +1 per event.
P1: event a, send m1 to P2 (event b), event c.
P2: event d, receive m1 (event e), send m2 to P3 (event f).
P3: event g, receive m2 (event h), event i.
Find all timestamps. Is d → b? Is b → h?

**Solution**
| Event | Timestamp |
|---|---|
| a, b (send m1, ts(m1) = 2), c | 1, 2, 3 |
| d | 1 |
| e (receive m1): max(1, 2) + 1 | 3 |
| f (send m2, ts(m2) = 4) | 4 |
| g | 1 |
| h (receive m2): max(1, 4) + 1 | 5 |
| i | 6 |

- d → b? **No.** C(d) = 1 < C(b) = 2, but there is no causal path; d and b are **concurrent**. This shows that a smaller timestamp does not imply happened-before.
- b → h? **Yes.** b → e (message m1), e → f (same process), f → h (message m2). Check: C(b) = 2 < C(h) = 5 ✔.

---

### Problem 6: Lamport receive rule
P1 sends a message with timestamp 40. At that moment P2's counter is 25. What is P2's counter after receipt? What if P2's counter had been 55?

**Solution**
- Counter 25: max(25, 40) = 40, then +1 → **41**.
- Counter 55: max(55, 40) = 55, then +1 → **56** (no big jump; the clock was already ahead).

---

### Problem 7: Vector clocks
Three processes start at (0,0,0).
- P1: event a, then sends m1 to P2 (event b).
- P2: event c, receives m1 (event d), sends m2 to P3 (event e).
- P3: event f, receives m2 (event g).

Compute all vectors and decide which pairs are concurrent.

**Solution**

| Event | Calculation | Vector |
|---|---|---|
| a (P1) | own slot +1 | (1,0,0) |
| b (P1, send m1) | own slot +1 | **(2,0,0)**, ts(m1) |
| c (P2) | own slot +1 | (0,1,0) |
| d (P2, receive m1) | max((0,1,0),(2,0,0)) = (2,1,0); own slot +1 | **(2,2,0)** |
| e (P2, send m2) | own slot +1 | **(2,3,0)**, ts(m2) |
| f (P3) | own slot +1 | (0,0,1) |
| g (P3, receive m2) | max((0,0,1),(2,3,0)) = (2,3,1); own slot +1 | **(2,3,2)** |

Comparisons:
- a (1,0,0) vs c (0,1,0): **concurrent** (1>0 in slot 1, 0<1 in slot 2).
- b (2,0,0) vs g (2,3,2): b < g, so **b → g** ✔ (b → d → e → g).
- f (0,0,1) vs e (2,3,0): **concurrent** (0<2 but 1>0).
- f (0,0,1) vs g (2,3,2): f < g, so **f → g** (same process).
- c (0,1,0) vs d (2,2,0): c < d, so **c → d** (same process).

---

### Problem 8: Totally ordered multicast
Initial balance is 500. P1 multicasts "add 50" with timestamp (3, P1). P2 multicasts "multiply by 2" with timestamp (3, P2). What final balance do all replicas reach?

**Solution**
- Timestamps tie at 3, so the tie-break uses process ID: (3, P1) < (3, P2).
- Every replica applies **"add 50" first**: 500 + 50 = 550, then **"multiply by 2"**: 550 × 2 = **1100**.
- All replicas agree on **1100**. (If one replica had done the doubling first it would reach 1050, which is the inconsistency totally ordered multicast prevents.)

---

### Problem 9: Fig 6-9 fix-up
In the Fig 6-9 pattern, P3 sends a message at clock 90 and P1's clock reads 70 when it arrives. What is P1's clock after receipt, if P1 ticks by 6?

**Solution**
- max(70, 90) = 90, then +1 → **91**.
- Subsequent ticks continue by 6: 97, 103, ...

---

### Problem 10: Which synchronization method?
(a) A cluster of 10 machines with no Internet access must agree with each other. (b) A web server must show correct legal time. (c) A chat system only needs replies after questions.

**Solution**
- (a) **Berkeley algorithm** (internal, no UTC needed).
- (b) **NTP** (or Cristian) with a UTC source (external).
- (c) **Lamport logical clocks** (only the causal order matters).

---

## 22. Self-Test (Short Questions with Answers)

**1. Define an asynchronous distributed system.**
*Answer:* Processes without common memory that communicate only via messages with unpredictable transmission delays.

**2. Why are global time and global state hard to realize?**
*Answer:* Processes are geographically distributed; events can occur at a high and unpredictable rate; event execution times can be very small.

**3. How do we approximate the global view?**
*Answer:* Simulate a synchronous system on an asynchronous one; simulate global time with logical clocks; simulate global state with global snapshots.

**4. Define clock skew and clock drift.**
*Answer:* Skew is the relative difference in clock values; drift is the relative difference in clock frequencies (rates).

**5. Define precision and accuracy, and say which kind of synchronization targets each.**
*Answer:* Precision: clocks agree with each other within a bound (internal synchronization). Accuracy: clocks stay within α of real time (external synchronization).

**6. What is UTC based on, and how is it obtained?**
*Answer:* Transitions of the cesium-133 atom; real time is the average of about 50 cesium clocks; leap seconds are added because days are getting longer; broadcast by short-wave radio and satellite (about ±0.5 ms from satellites).

**7. Write the offset and delay formulas.**
*Answer:* θ = ((T2 − T1) + (T3 − T4)) / 2 and δ = ((T4 − T1) − (T3 − T2)) / 2, assuming request delay ≈ response delay.

**8. Describe Cristian's algorithm and its error bound.**
*Answer:* Client asks the time server, which replies with T. The client sets its time to T + RTT/2. The true time lies in [T + min, T + RTT − min], so the error is at most RTT/2 − min.

**9. Describe NTP's structure and accuracy.**
*Answer:* A hierarchical tree of servers. The primary server at the root synchronizes with UTC, secondary servers back it up, and the lowest level is a synchronization subnet with clients. Accuracy: milliseconds on WANs and sub-milliseconds on LANs. It is the most widely used physical clock protocol, with 10 to 20 million servers and clients.

**10. Describe the Berkeley algorithm.**
*Answer:* A daemon without UTC polls all machines, receives their times, computes the average, and broadcasts the adjustment so that all machines agree (internal synchronization).

**11. State the two situations where happened-before can be observed directly.**
*Answer:* (1) Two events in the same process, a before b, gives a → b. (2) a is a message send and b is the receipt of that message, gives a → b.

**12. State Lamport's three update rules.**
*Answer:* (1) Cᵢ ← Cᵢ + 1 before an event. (2) A sent message carries ts(m) = Cᵢ. (3) On receipt, Cⱼ ← max{Cⱼ, ts(m)}, then step 1, then deliver.

**13. Where is the Lamport logic implemented?**
*Answer:* In a middleware layer between application and network. It adjusts the clock and timestamps outgoing messages, and adjusts the clock on incoming messages before delivery.

**14. What does Fig 6-9 (a) show and how does (b) fix it?**
*Answer:* (a) Clocks tick at different rates (6, 8, 10), and messages m3 and m4 arrive with timestamps lower than their sending times. (b) The receiver adjusts its clock forward (P2 to 61, P1 to 70) so receive > send.

**15. What is the problem in Fig 6-11?**
*Answer:* A replicated database receives two updates in different orders at its replicas, leaving it inconsistent. Totally ordered multicast (Lamport timestamps and acknowledgements) makes all replicas apply the same order.

**16. State the two properties of a vector clock.**
*Answer:* (1) VCᵢ[i] is the number of events at Pᵢ (its local logical clock). (2) If VCᵢ[j] = k then Pᵢ knows k events have occurred at Pⱼ.

**17. State the vector clock update rules.**
*Answer:* (1) VCᵢ[i] ← VCᵢ[i] + 1 before an event. (2) A sent message carries ts(m) = VCᵢ. (3) On receipt, VCⱼ[k] ← max{VCⱼ[k], ts(m)[k]} for each k, then step 1, then deliver.

**18. How do you test whether two events are concurrent using vector clocks?**
*Answer:* Compare slot by slot. If neither vector is ≤ the other in all slots, the events are concurrent.

**19. Why do vector clocks detect concurrency while Lamport clocks do not?**
*Answer:* A vector records what each process knows about every other process, so differing knowledge shows that neither event could have caused the other. A single number cannot carry that information.

**20. Differentiate partial order from total order.**
*Answer:* In a total order, every pair of events is comparable. In a partial order, some pairs (concurrent events) are incomparable. Happened-before is a partial order.

---

## 23. One-Page Revision Summary

1. **Problem:** distributed machines have **no shared memory and no shared clock**, and messages have unpredictable delays (**asynchronous**). Timestamps from different machines can be wrongly ordered (Fig 6-1, the `make` example).
2. **Global time and state are hard** (geography, high event rate, tiny event times), so we **approximate**: clock synchronization, **logical clocks** (for time), **global snapshots** (for state).
3. **Skew** = difference in clock **values**. **Drift** = difference in clock **rates**. **Precision** = clocks agree with each other (**internal sync**). **Accuracy** = clocks agree with real time (**external sync**).
4. **UTC:** cesium-133 atomic clocks (average of about 50), leap seconds, broadcast by radio and satellite (about ±0.5 ms).
5. **Four timestamps:** T1 (client sends), T2 (server receives), T3 (server sends), T4 (client receives). **θ = ((T2−T1)+(T3−T4))/2**, **δ = ((T4−T1)−(T3−T2))/2**, assuming symmetric delays. Slide example: RTT = 161 ms, one-way about 80.5 ms; NTP formulas give θ = 0, δ = 80 ms.
6. **Cristian:** ask the time server, set time = T + RTT/2; true time in [T+min, T+RTT−min]; error ±(RTT/2 − min). Slide problem: RTT = 70, adjusted = 375 + 35 = 410, offset = 410 − 710 = **−300 = −3 s** (B is 3 s ahead).
7. **NTP:** Internet-wide, **hierarchical tree** (root synchronized with UTC, secondaries as backup, clients at the bottom), ms on WANs and sub-ms on LANs.
8. **Berkeley:** internal. A **daemon without UTC** polls, **averages** (including itself), and sends **corrections**. Slide: daemon 3:00, A 2:50 (−10), B 3:25 (+25); average +5, so all go to 3:05 (corrections +5, +15, −20).
9. **Order beats time:** use **happened-before (→)**: same process order, **send → receive**, transitivity. No chain means **concurrent**.
10. **Lamport clocks:** C ← C+1 before each event; message carries C; on receipt **C ← max(C, ts)**, then +1. Guarantees **a → b ⇒ C(a) < C(b)**, but **not the converse**, and cannot detect concurrency. Implemented in **middleware**. Fig 6-9: ticks of 6, 8, 10; P2 adjusts to **61**, P1 to **70**.
11. **Partial vs total order:** causality is a **partial order**. Lamport (scalar) gives a partial ordering; vector clocks give a **more complete ordering that detects concurrency**. Break ties with process ID to get a total order.
12. **Totally ordered multicast (Fig 6-11):** replicas must apply updates in the **same order**. Example: $1,000, +$100 and +1% give **$1,111 vs $1,110** if ordered differently. Fix: Lamport timestamps, ordered queues, acknowledgements, deliver when at the head and acked by all.
13. **Vector clocks:** each process keeps a vector. **VCᵢ[i]** = my own events, **VCᵢ[j]=k** = I know k events happened at Pⱼ. Rules: increment own slot; send the whole vector; on receipt take **slot-wise max**, then increment own slot. **V < W ⇔ a → b**; if neither is ≤ the other, the events are **concurrent**.
14. **Slide trace (verified):** (0,1,0) → (1,1,0) → (2,1,0) → (2,1,1) → (3,1,0) → (4,1,0) → (4,2,0) → (4,3,0) → (4,3,2).
15. **Which to use:** need real time → **NTP/Cristian**; agree among ourselves without UTC → **Berkeley**; need causal order cheaply → **Lamport**; need to detect concurrent updates → **vector clocks**.

---

*End of Handbook*
