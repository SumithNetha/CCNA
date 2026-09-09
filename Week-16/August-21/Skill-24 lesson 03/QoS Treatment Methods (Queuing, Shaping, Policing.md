# Week 16 — August 21

# QoS Treatment Methods: Queuing, Shaping & Policing

This lesson completes the **QoS traffic-treatment stage**. The previous lesson covered **classification and marking**; now we determine **what the network actually does with that marked traffic when congestion occurs**.

Your study plan places this lesson on **Friday, August 21**, together with the **“QoS Treatment Methods in Action” lab**, quiz, and the Skill 24 wrap-up lab. 

---

# 1. The Big QoS Picture

QoS can be understood as a pipeline:

```text
                 TRAFFIC
                    │
                    ▼
             CLASSIFICATION
              "What is it?"
                    │
                    ▼
                 MARKING
              "How important?"
                    │
                    ▼
             TRUST BOUNDARY
          "Do I trust this mark?"
                    │
                    ▼
              TREATMENT
                    │
       ┌────────────┼────────────┐
       ▼            ▼            ▼
    QUEUING       SHAPING      POLICING
       │            │            │
       └────────────┼────────────┘
                    ▼
              TRANSMISSION
```

### Classification

Identifies the traffic.

Example:

```text
Source → Voice traffic
```

### Marking

Adds a label indicating how the traffic should be treated.

Example:

```text
Voice → DSCP EF
```

### Treatment

Actually determines what happens to the traffic during congestion.

Examples:

* Put voice in a priority queue.
* Give business traffic guaranteed bandwidth.
* Delay excess traffic with shaping.
* Drop excess traffic with policing.
* Drop lower-priority traffic early using WRED.

---

# 2. Trust Boundary

Before discussing treatment, the lesson introduces an important concept:

## Trust boundary

A **trust boundary** is the point in the network where you decide whether to trust the QoS markings arriving with packets.

Imagine a user device sends:

```text
"I'm extremely important!"

DSCP = EF
```

Should the network automatically believe it?

**No.**

A normal endpoint could potentially mark its own traffic as high priority.

For example:

```text
Gaming PC
     │
     │ "I'm priority traffic!"
     ▼
Switch
```

If the switch blindly trusts the marking:

```text
Gaming traffic
     ↓
High priority
     ↓
Gets preferential treatment
```

That could allow users or applications to abuse QoS.

---

## Trust boundary example

```text
                     TRUSTED
                       │
Voice Phone ──► Switch ──► Network
                  ▲
                  │
           Trust boundary
```

You decide:

> "From this point onward, I trust these QoS markings."

But traffic entering from an untrusted endpoint may be:

```text
Classify
   ↓
Remark
   ↓
Forward
```

---

# 3. Why Trust Boundaries Matter

QoS resources are limited.

Suppose a link has:

```text
100 Mbps
```

and everyone marks their traffic as high priority:

```text
PC 1 → EF
PC 2 → EF
PC 3 → EF
PC 4 → EF
PC 5 → EF
```

Then QoS loses much of its usefulness.

The entire point is to differentiate traffic.

Therefore:

> **Don't trust QoS markings unless you have a reason to trust the device generating them.**

---

# 4. QoS Becomes Important During Congestion

This is one of the most important ideas in the lesson.

If you have:

```text
1 Gbps available

Traffic:
200 Mbps
```

there isn't much competition.

QoS treatment doesn't have much work to do.

But:

```text
100 Mbps available

Traffic:
150 Mbps
```

creates a problem.

```text
150 Mbps demand
       │
       ▼
 ┌────────────┐
 │ 100 Mbps   │
 │   LINK     │
 └────────────┘
       │
       ▼
     50 Mbps
     excess
```

Something has to happen.

QoS decides **which traffic gets preferential treatment and how excess traffic is handled**.

---

# 5. QoS Does NOT Create Bandwidth

Very important:

```text
100 Mbps link
     +
QoS
     ≠
200 Mbps link
```

QoS doesn't increase physical capacity.

Instead:

```text
Limited bandwidth
       +
Traffic prioritization
       +
Traffic management
       ↓
Better behavior during congestion
```

Think of QoS as **managing scarcity**.

---

# 6. Queuing

## What is queuing?

When packets cannot immediately leave an interface, they wait in a queue.

```text
Packets
 ↓↓↓↓↓↓↓↓↓

┌──────────────┐
│ P1           │
│ P2           │
│ P3           │
│ P4           │
│ P5           │
└──────────────┘
       │
       ▼
   Interface
```

The queuing mechanism determines:

> **Which packet should be transmitted next?**

---

# 7. FIFO

The basic queuing behavior is:

## FIFO — First In, First Out

The packet that enters first leaves first.

```text
Arrival:

P1 → P2 → P3 → P4

Transmission:

P1 → P2 → P3 → P4
```

No special consideration is given to:

* voice
* video
* database traffic
* backups
* web traffic

It is simply:

> First packet in → first packet out.

---

# 8. Tail Drop

What happens when the queue becomes full?

Eventually, there is no more room.

Packets arriving after the queue is full are dropped.

This is called:

## Tail drop

Example:

```text
Queue capacity = 5 packets

[P1]
[P2]
[P3]
[P4]
[P5]  ← FULL

P6 arrives
 ↓
NO SPACE
 ↓
DROP
```

The problem is that tail drop doesn't inherently care what the packet is.

It could be:

```text
Voice packet     → DROP
Database packet  → DROP
Backup packet    → DROP
Web packet       → DROP
```

Everything is treated according to the queue's behavior.

---

# 9. The Problem With Tail Drop

Imagine many TCP connections are using the same congested link.

The queue fills:

```text
Queue full
   ↓
Multiple packets dropped
   ↓
Multiple TCP senders detect loss
   ↓
Multiple senders reduce transmission
```

Then traffic decreases.

After some time:

```text
Traffic increases
      ↓
Queue fills
      ↓
Packets drop
      ↓
Traffic decreases
```

This cycle can repeat.

---

# 10. Global Synchronization

The lesson calls this:

## Global synchronization

Multiple TCP flows respond to congestion around the same time.

Conceptually:

```text
Traffic
  ▲
  │       /\        /\
  │      /  \      /  \
  │     /    \    /    \
  │____/      \__/      \__
  │
  └────────────────────────► Time
```

This creates a **sawtooth-like traffic pattern**.

The network repeatedly:

```text
builds traffic
     ↓
hits congestion
     ↓
drops packets
     ↓
TCP backs off
     ↓
traffic decreases
     ↓
traffic builds again
```

The result can be inefficient utilization.

---

# 11. WRED

## Weighted Random Early Detection

**WRED** attempts to deal with congestion **before the queue becomes completely full**.

Instead of:

```text
Queue
████████████████
FULL
 ↓
TAIL DROP
```

WRED can begin dropping selected packets earlier.

```text
Queue filling
██████████░░░░░
      ↓
   WRED begins
   early drops
      ↓
TCP senders back off
```

---

# 12. Why "Early" Detection?

The objective is to avoid waiting until the queue is completely full.

Instead:

```text
Normal
  ↓
Queue increasing
  ↓
Congestion approaching
  ↓
WRED starts dropping
  ↓
TCP responds
  ↓
Traffic reduces
```

This is more controlled than waiting until:

```text
Queue = 100% full
```

and then suddenly dropping packets at the tail.

---

# 13. Why "Weighted"?

The **weighted** part allows WRED to treat traffic differently according to its characteristics, including the packet's marking/drop preference.

The lesson's idea is:

```text
Higher-priority traffic
        ↓
Protect more

Lower-priority traffic
        ↓
Drop earlier
```

For example:

```text
              WRED
                │
       ┌────────┴────────┐
       ▼                 ▼
 High-priority       Low-priority
 traffic             traffic
       │                 │
       ▼                 ▼
 Protect more        Drop sooner
```

This makes previously applied QoS markings useful during congestion.

---

# 14. WRED and TCP

WRED is particularly useful because many network applications use TCP.

When TCP detects packet loss, it can reduce its sending rate.

Therefore:

```text
WRED
  ↓
Early packet loss
  ↓
TCP detects congestion
  ↓
TCP reduces sending rate
  ↓
Congestion decreases
```

Instead of waiting for severe queue overflow.

---

# 15. Shaping

Now we get to one of the most important QoS concepts.

## Traffic shaping

Shaping controls the rate at which traffic is transmitted by **buffering excess traffic**.

The key word is:

> **BUFFER**

Suppose:

```text
Incoming traffic = 100 Mbps
Desired rate     = 50 Mbps
```

A shaper can do:

```text
100 Mbps
    │
    ▼
┌───────────┐
│  SHAPER   │
└───────────┘
    │
    ▼
 Buffer excess
    │
    ▼
50 Mbps
```

The excess traffic waits.

---

# 16. Shaping = Delay

A useful mental model:

> **Shaping says: "Not now. I'll send it later."**

Example:

```text
Traffic arrives too quickly
           ↓
        SHAPER
           ↓
    ┌─────────────┐
    │   BUFFER    │
    │ P4          │
    │ P5          │
    │ P6          │
    └─────────────┘
           ↓
       Send later
```

So shaping can introduce **delay**, but it can avoid immediately dropping packets.

---

# 17. Why Use Shaping?

Suppose a WAN service allows:

```text
20 Mbps
```

but your local router could transmit:

```text
100 Mbps
```

If traffic is sent too quickly toward the provider, the provider may discard excess traffic.

You can shape your outbound traffic:

```text
100 Mbps capable interface
          ↓
       SHAPER
          ↓
        20 Mbps
          ↓
        ISP
```

The router controls the rate before traffic reaches the bottleneck.

---

# 18. Shaping Smooths Bursts

Applications don't always generate perfectly steady traffic.

You might see:

```text
Normal
  ↓
Burst
  ↓
Normal
  ↓
Large burst
```

Shaping can smooth these bursts:

```text
BURSTY TRAFFIC
████████████████
       ↓
    SHAPING
       ↓
████████
████████
████████
```

Instead of allowing the burst to immediately overwhelm the downstream link.

---

# 19. Policing

Now compare that with:

## Traffic policing

Policing enforces a configured traffic-rate limit.

Conceptually:

```text
Traffic
   ↓
POLICER
   ↓
Is it within allowed rate?
   │
 ┌─┴─────────────┐
 │               │
YES             NO
 │               │
 ▼               ▼
Forward      Drop/Remark
```

---

# 20. Policing = Enforcement

A good memory technique:

> **Policing says: "You're over the limit. Stop."**

Unlike shaping, the excess traffic isn't simply stored so that it can be transmitted later.

The lesson describes policing as potentially:

* **dropping** excess traffic
* **remarking** excess traffic

---

# 21. Shaping vs Policing

This is one of the most important comparison tables from this lesson.

| Characteristic         | Shaping                              | Policing                |
| ---------------------- | ------------------------------------ | ----------------------- |
| Controls traffic rate  | ✅                                    | ✅                       |
| Buffers excess traffic | **Yes**                              | **No**                  |
| Delays packets         | **Yes**                              | Generally no            |
| Drops excess traffic   | Can occur if buffers eventually fill | **Yes, commonly**       |
| Can remark traffic     | Possible depending on configuration  | **Yes**                 |
| Smooths traffic        | **Yes**                              | No                      |
| Main concept           | Delay/buffer                         | Enforce/drop            |
| Mental model           | "Send it later"                      | "You're over the limit" |

### Remember:

```text
SHAPING
Excess
  ↓
BUFFER
  ↓
SEND LATER


POLICING
Excess
  ↓
DROP / REMARK
```

---

# 22. When Should You Shape?

Use shaping when:

> **You would rather delay traffic than lose it.**

Example:

```text
Business application
       ↓
Traffic burst
       ↓
SHAPER
       ↓
Buffer
       ↓
Send later
```

The application may experience additional delay, but the packet has a chance to reach its destination.

---

# 23. When Should You Police?

Use policing when:

> **You need a hard traffic ceiling.**

For example, you may want to prevent lower-priority traffic from consuming an excessive amount of bandwidth.

The lesson gives examples such as:

* guest traffic
* peer-to-peer traffic
* traffic that can become excessively greedy

Conceptually:

```text
Guest traffic
     ↓
POLICER
     ↓
Maximum allowed rate
     ↓
Excess → DROP / REMARK
```

---

# 24. The Queuing Toolbox

The lesson then discusses several queuing approaches.

The important ones are:

1. FIFO
2. WFQ
3. Custom Queuing
4. Priority Queuing
5. LLQ

---

# 25. FIFO

Already covered:

```text
First in
   ↓
First out
```

No traffic prioritization.

---

# 26. WFQ

## Weighted Fair Queuing

WFQ attempts to provide fair treatment between traffic flows while giving some preference to **lower-bandwidth conversations**.

The lesson highlights its usefulness for:

* small interactive traffic
* remote sessions

Conceptually:

```text
Small interactive flow
        ↓
      WFQ
        ↓
Gets fair opportunity

Large flow
        ↓
      WFQ
        ↓
Doesn't automatically dominate
```

This can be useful for interactive traffic because small flows can otherwise get buried behind large transfers.

---

# 27. Why WFQ Isn't Ideal for Everything

Consider a continuous voice or video stream.

Voice/video can require consistent, predictable treatment.

WFQ isn't designed to provide the same strict low-latency behavior as a dedicated priority mechanism.

Therefore:

```text
Interactive small flows
       ↓
       WFQ
```

may be useful, while:

```text
Real-time voice
       ↓
Priority treatment
```

is often more appropriate.

---

# 28. Custom Queuing

Custom Queuing allows bandwidth to be assigned to different traffic classes.

Conceptually:

```text
              Custom Queuing
                    │
       ┌────────────┼────────────┐
       ▼            ▼            ▼
    Queue 1      Queue 2      Queue 3
      40%          30%          30%
```

The router services queues in a **round-robin style**.

For example:

```text
Queue 1
   ↓
Queue 2
   ↓
Queue 3
   ↓
Queue 1
   ↓
Queue 2
   ↓
...
```

---

# 29. Problem With Custom Queuing

Suppose voice traffic is assigned to one queue.

It still has to wait for its turn.

```text
Queue 1 → Data
Queue 2 → Backup
Queue 3 → Voice
```

If the router is servicing the other queues:

```text
Data
 ↓
Backup
 ↓
Voice
```

voice experiences additional delay.

For real-time voice, this isn't ideal.

---

# 30. Priority Queuing

Priority Queuing solves the previous problem by creating priority levels.

Example:

```text
             PRIORITY QUEUING

          ┌─────────────────┐
          │ HIGH PRIORITY   │
          │ Voice           │
          └────────┬────────┘
                   │
                   ▼
          ┌─────────────────┐
          │ MEDIUM          │
          │ Business        │
          └────────┬────────┘
                   │
                   ▼
          ┌─────────────────┐
          │ LOW             │
          │ Backup          │
          └─────────────────┘
```

High-priority traffic gets serviced first.

---

# 31. The Problem: Starvation

Priority queuing sounds perfect:

> "Just put voice first!"

But what if the priority queue is constantly full?

```text
HIGH PRIORITY
████████████████████
████████████████████
████████████████████

LOW PRIORITY
      ↓
     WAIT
      ↓
     WAIT
      ↓
     WAIT
```

Lower-priority queues can potentially receive little or no service.

This is:

## Traffic starvation

One class consumes so much attention that other traffic struggles to get served.

---

# 32. LLQ

## Low Latency Queuing

The lesson describes **LLQ** as combining several useful QoS concepts.

The key idea:

> **LLQ provides a strict priority queue for delay-sensitive traffic while also providing class-based treatment for other traffic.**

Conceptually:

```text
                       LLQ
                        │
          ┌─────────────┴──────────────┐
          │                            │
          ▼                            ▼
   PRIORITY QUEUE                CLASS QUEUES
          │                            │
       Voice                       Video
                                    │
                                 Business
                                    │
                                  Other
```

---

# 33. Why LLQ Is So Important

Voice traffic needs low:

* delay
* jitter

Therefore, voice can be placed into a priority queue.

```text
Voice
  ↓
Priority Queue
  ↓
Transmit quickly
```

Meanwhile:

```text
Video
  ↓
Class-based queue

Business
  ↓
Class-based queue

Other
  ↓
Fair/default queue
```

This gives the network administrator considerably more control.

---

# 34. LLQ and Protection Against Starvation

The lesson specifically points out that the priority queue is **policed**, preventing it from consuming the entire link.

Conceptually:

```text
             LLQ
              │
              ▼
       Priority Queue
              │
              ▼
       Priority traffic
              │
       ┌──────┴──────┐
       │             │
       ▼             ▼
 within limit     exceeds limit
       │             │
       ▼             ▼
    transmit      enforce limit
```

So instead of:

```text
Voice → unlimited bandwidth
```

you establish controlled priority treatment.

---

# 35. LLQ Example

Suppose:

```text
WAN = 100 Mbps
```

You have:

```text
Voice       → priority
Video       → important
Database    → important
Web         → normal
Backup      → low priority
```

A conceptual QoS policy might look like:

```text
100 Mbps
   │
   ├── Voice
   │     └── Priority
   │
   ├── Video
   │     └── Class queue
   │
   ├── Database
   │     └── Class queue
   │
   └── Other
         └── Fair/default queue
```

During congestion:

```text
Voice gets priority
       ↓
Video gets controlled treatment
       ↓
Database gets controlled treatment
       ↓
Other traffic gets remaining treatment
```

---

# 36. Why LLQ Is Excellent for VoIP

Consider a coffee shop.

There is a WAN link connecting the shop to headquarters.

At the same time:

```text
VoIP call
+
Plex traffic
+
Cloud applications
+
File upload
+
Guest browsing
```

Suppose a large file upload begins.

Without appropriate QoS:

```text
Large file transfer
████████████████████

Voice
  ↓
competes
  ↓
delay
  ↓
jitter
  ↓
choppy call
```

With LLQ:

```text
Voice
  ↓
Priority Queue
  ↓
Fast treatment

Other traffic
  ↓
Class queues
  ↓
Controlled treatment
```

This protects the real-time application.

---

# 37. Castle Rysen Example

The Castle Rysen RFP specifically requires district shops to support **Plex-based video streaming and video surveillance**, while separating administrative and guest traffic. 

It also calls for QoS as part of the network's monitoring and management capabilities. 

So imagine:

```text
                    District Shop
                         │
                       Router
                         │
                     WAN Link
                         │
       ┌─────────────────┼─────────────────┐
       │                 │                 │
      Voice             Plex          Business
       │                 │                 │
   Priority          Important        Important
```

During congestion, QoS can ensure that critical applications aren't treated identically to less important traffic.

---

# 38. Complete QoS Treatment Model

Here's the mental model I recommend remembering:

```text
                         QoS
                          │
          ┌───────────────┴───────────────┐
          │                               │
   Classification                      Marking
   "What is it?"                  "How should it be
                                      treated?"
          │                               │
          └───────────────┬───────────────┘
                          │
                    TRUST BOUNDARY
                          │
                          ▼
                      CONGESTION
                          │
             ┌────────────┼────────────┐
             │            │            │
             ▼            ▼            ▼
          QUEUING       SHAPING      POLICING
             │            │            │
             │            │            │
       Decide order    Buffer       Enforce rate
             │         excess            │
             │            │          Drop/Remark
             ▼            ▼            ▼
                     TRANSMISSION
```

And WRED operates as an additional congestion-management mechanism:

```text
Queue filling
     ↓
   WRED
     ↓
Early selective drops
     ↓
TCP backs off
```

---

# 39. The Most Important Comparisons

## FIFO vs Priority Queuing

| FIFO                       | Priority Queuing                |
| -------------------------- | ------------------------------- |
| First in, first out        | Priority determines service     |
| No priority                | High priority gets served first |
| Simple                     | More complex                    |
| Can hurt important traffic | Can starve lower classes        |

---

## WFQ vs Priority Queuing

| WFQ                                          | Priority Queuing                        |
| -------------------------------------------- | --------------------------------------- |
| Fair treatment between flows                 | Strict priority                         |
| Helps smaller flows                          | High priority gets immediate preference |
| Good for interactive traffic                 | Good for delay-sensitive traffic        |
| Doesn't inherently guarantee strict priority | Can cause starvation                    |

---

## Shaping vs Policing

| Shaping         | Policing                       |
| --------------- | ------------------------------ |
| Buffers         | Doesn't normally buffer excess |
| Delays          | Drops/remarks excess           |
| Smooths traffic | Enforces a ceiling             |
| "Send later"    | "Too much → enforce"           |

---

## Tail Drop vs WRED

| Tail Drop                                           | WRED                             |
| --------------------------------------------------- | -------------------------------- |
| Waits until queue is full                           | Starts dropping before full      |
| Drops arriving packets at queue limit               | Early selective drops            |
| Can contribute to global synchronization            | Helps reduce synchronization     |
| Doesn't intelligently prefer lower-priority traffic | Can use traffic/drop preferences |

---

# 40. Exam Memory Tricks

### QoS pipeline

> **Classify → Mark → Treat**

---

### FIFO

> **First In, First Out**

---

### Tail Drop

> **Queue full → packet gets dropped**

---

### WRED

> **Drop early, especially less-preferred traffic**

---

### Shaping

> **Buffer and delay**

Think:

> **SHAPING = SEND LATER**

---

### Policing

> **Enforce the limit**

Think:

> **POLICING = DROP/REMARK**

---

### Priority Queuing

> **High priority goes first**

Problem:

> **Starvation**

---

### LLQ

> **Priority treatment + class-based queuing**

Best mental association:

> **LLQ → Low-latency voice/video**

---

# 41. One Scenario to Remember Everything

Imagine a 50 Mbps WAN link:

```text
              50 Mbps
                │
        ┌───────┴────────┐
        │                │
       Voice            Data
        │                │
      DSCP EF          Other DSCP
        │                │
        └───────┬────────┘
                │
             QoS Policy
                │
       ┌────────┼─────────┐
       │        │         │
      LLQ      WFQ      WRED
       │
     Voice
       │
   Priority
```

If traffic becomes excessive:

### Queuing

Determines **who goes first**.

### LLQ

Gives voice **priority treatment**.

### WRED

Can begin **dropping less-preferred traffic early**.

### Shaping

Can **buffer excess traffic and send it later**.

### Policing

Can **drop or remark traffic exceeding the configured rate**.

---

# 42. Final Mental Model

Don't memorize QoS as a collection of unrelated acronyms.

Think about the problem sequentially:

```text
1. WHAT IS THIS TRAFFIC?
        ↓
   Classification

2. HOW SHOULD IT BE IDENTIFIED?
        ↓
      Marking

3. DO I TRUST THAT MARK?
        ↓
   Trust Boundary

4. THE LINK IS CONGESTED.
   WHO GETS SERVED FIRST?
        ↓
      Queuing

5. TRAFFIC IS TOO BURSTY.
   CAN I DELAY IT?
        ↓
     Shaping

6. TRAFFIC EXCEEDS A HARD LIMIT.
        ↓
     Policing

7. QUEUE IS GETTING FULL.
   CAN I REDUCE CONGESTION EARLY?
        ↓
       WRED
```

### The four words to lock in:

> **Queuing = ORDER**
> **WRED = DROP EARLY**
> **Shaping = BUFFER**
> **Policing = ENFORCE**

And the overall lesson:

> **QoS doesn't make the network faster. It makes the network smarter about what happens when it can't send everything at once.**
