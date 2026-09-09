# Week 16 — August 20, 2026

# Skill 24 — QoS

## Lesson 02: Understanding Classification and Marking Process

This lesson goes one level deeper than the previous QoS overview.

The previous lesson established:

> **QoS manages traffic during congestion.**

This lesson answers:

> **Before QoS can prioritize traffic, how does the network know what each packet is?**

The answer is:

**Classification → Marking**

---

# 1. Classification vs. Marking

These two concepts are the foundation of QoS.

### Classification

**Classification identifies what the traffic is.**

Examples:

```text
Packet → Voice
Packet → Video
Packet → Web
Packet → Guest
Packet → Backup
```

Think:

> **Classification = Detective work**

The device examines traffic and determines which category it belongs to.

---

### Marking

**Marking gives the traffic a label representing how it should be treated.**

Think:

> **Marking = Putting a sticker on the packet**

For example:

```text
Voice packet
     ↓
CLASSIFY
     ↓
VOICE
     ↓
MARK
     ↓
EF
```

Now downstream devices can recognize the marking without having to perform the entire classification process again.

---

# 2. Why Do We Need Both?

Consider a packet travelling through multiple routers:

```text
          Classification
               ↓
            Router 1
               ↓
             Mark
               ↓
        ┌───────────────┐
        ↓               ↓
     Router 2        Router 3
        ↓               ↓
     Router 4        Router 5
```

If Router 1 identifies the packet as voice and marks it, subsequent devices can use that marking.

Without marking, every device might need to repeatedly determine:

> “What application generated this packet?”

With marking:

```text
Identify once
     ↓
Mark
     ↓
Trust/use marking downstream
```

This is both an organizational and operational advantage.

---

# 3. What Can Be Used for Classification?

The lesson identifies several possible classification criteria.

A device can classify traffic based on:

* **Port numbers**
* **Protocols**
* **IP addresses**
* **Interfaces**
* **VLANs**
* **Applications**

Conceptually:

```text
                 PACKET
                    │
       ┌────────────┼────────────┐
       ↓            ↓            ↓
    IP address    Port        Protocol
       │            │            │
       └────────────┼────────────┘
                    ↓
              CLASSIFICATION
```

The objective is to determine:

> **Which traffic class does this packet belong to?**

---

# 4. Classification Using IP Addresses

You can classify traffic based on source or destination IP information.

For example:

```text
10.10.10.0/24 → Administration
10.10.20.0/24 → Voice
10.10.30.0/24 → Guest
```

A QoS policy can use these characteristics to distinguish traffic.

For example:

```text
Source = Voice subnet
       ↓
Classify as VOICE
```

---

# 5. Classification Using Port Numbers

Applications commonly use TCP or UDP ports.

Therefore, ports can also be used as classification criteria.

Conceptually:

```text
Packet
  ↓
Destination/source port
  ↓
Match rule
  ↓
Traffic class
```

For example:

```text
Known application traffic
        ↓
Port-based match
        ↓
Classify
```

The important CCNA concept is not memorizing a huge list of ports here.

The important concept is:

> **Port numbers can be one of the characteristics used to identify traffic.**

---

# 6. Classification Using Protocols

Traffic can also be identified by protocol.

For example, a classification rule might distinguish:

```text
TCP
UDP
ICMP
```

or other protocol characteristics.

Again, the objective is:

```text
Packet
  ↓
Identify characteristics
  ↓
Determine class
```

---

# 7. Classification Using Interfaces

The interface where traffic enters or leaves can itself provide useful information.

For example:

```text
Gi0/1 → Voice-related traffic
Gi0/2 → Guest network
Gi0/3 → Business devices
```

The interface can therefore be part of the classification logic.

This can be particularly useful when the network architecture already separates traffic physically or logically.

---

# 8. Classification Using VLANs

VLAN membership can also help identify traffic.

For example, a coffee shop might have:

```text
VLAN 10 → Administration
VLAN 20 → Guest
VLAN 30 → Cameras
VLAN 40 → Voice
```

A QoS policy can use VLAN-related information to classify traffic.

This connects QoS directly with the network segmentation concepts you've already studied.

---

# 9. Classification Using Applications

Sometimes simply looking at IP addresses or ports isn't sufficient.

This is where application-aware identification becomes useful.

The lesson introduces:

> **NBAR — Network Based Application Recognition**

---

# 10. NBAR

**NBAR** stands for:

> **Network Based Application Recognition**

The key idea:

> **NBAR allows a router to identify applications based on application signatures rather than relying only on basic port matching.**

Conceptually:

```text
Traditional matching
        ↓
IP / Port / Protocol
        ↓
Traffic identification
```

With NBAR:

```text
Traffic
   ↓
Deeper inspection / application recognition
   ↓
Application identified
```

This provides more precise application-level classification.

---

# 11. Why NBAR Matters

Basic port-based classification isn't always enough.

Applications may not always behave exactly as expected from a simple port-based perspective.

So instead of only asking:

> “What port is this using?”

NBAR can help answer:

> **“What application does this traffic actually represent?”**

That gives the network administrator another tool for more precise traffic identification.

---

# 12. Class Maps

Once you've determined what you're trying to identify, you need a way to organize the matching conditions.

The lesson introduces:

> **Class map**

A **class map** is used to group matching criteria into a traffic class.

Think:

```text
                 CLASS MAP
                     │
          ┌──────────┼──────────┐
          ↓          ↓          ↓
       Match IP   Match port   Match protocol
          │          │          │
          └──────────┼──────────┘
                     ↓
                Traffic class
```

In simple terms:

> **A class map defines the conditions used to identify a particular class of traffic.**

---

# 13. Why Class Maps Matter

QoS eventually needs to perform actions on traffic.

But before it can perform those actions, it needs to know:

```text
What traffic?
      ↓
Which class?
      ↓
What treatment?
```

So conceptually:

```text
Traffic
   ↓
CLASSIFICATION
   ↓
CLASS MAP
   ↓
Traffic class
   ↓
MARK / QUEUE / SHAPE / POLICE
```

The class map provides the organizational structure for classification.

---

# 14. Real-World Classification Example

Imagine Castle Rysen Coffee has:

```text
Voice
Payment systems
Cameras
Guest Wi-Fi
Back-office traffic
```

You might logically classify traffic as:

```text
VOICE
BUSINESS-CRITICAL
VIDEO-SURVEILLANCE
GUEST
BULK/BACKUP
```

Conceptually:

```text
                  TRAFFIC
                     │
       ┌─────────────┼──────────────┐
       ↓             ↓              ↓
     Voice         Guest          Camera
       │             │              │
       ↓             ↓              ↓
     CLASS         CLASS          CLASS
```

Now QoS can apply different policies to each class.

---

# 15. Marking — Giving Traffic a Label

Once traffic has been classified, the next step is **marking**.

Marking assigns a value to traffic that downstream devices can recognize.

Think:

```text
Classification
      ↓
"This is voice."
      ↓
Mark it
      ↓
"EF"
```

Now the packet carries an indication of how it should be treated.

---

# 16. Layer 2 Marking — CoS

At Layer 2, the lesson introduces:

> **CoS — Class of Service**

CoS uses **3 bits** in the **802.1Q VLAN tagging information**.

Therefore:

```text
CoS = 3 bits
```

Three bits provide:

```text
2³ = 8 values
```

So CoS values range from:

```text
0 – 7
```

### Important

CoS is associated with the **802.1Q VLAN tag**, so it is particularly relevant when VLAN tagging exists, such as on **trunk links**.

---

# 17. CoS Values

The lesson presents CoS as providing different levels of traffic treatment.

Conceptually:

```text
Lower priority
      ↓
Bulk/scavenger traffic
      ↓
Normal traffic
      ↓
Higher-priority traffic
      ↓
Voice
      ↓
Network control
```

An important point from the lesson is that **network control traffic can have higher importance than voice**.

Why?

Because network control traffic helps maintain the operation of the network itself.

Examples include control-plane protocols such as:

* Routing protocols
* Spanning Tree-related control traffic

If the network's control mechanisms fail, prioritizing an individual voice call doesn't help much.

---

# 18. Why CoS Is Layer 2

CoS is carried in the 802.1Q VLAN tagging information.

Simplified:

```text
Ethernet Frame
┌───────────────┬─────────────┬─────────────┐
│ Ethernet      │ 802.1Q Tag  │ Data        │
│ Header        │             │             │
└───────────────┴─────────────┴─────────────┘
                     │
                     ↓
                    CoS
```

So remember:

> **CoS → Layer 2 → 802.1Q → 3 bits → 0–7**

---

# 19. Layer 3 Marking — IP Precedence

The lesson then moves to Layer 3 marking.

An older Layer 3 marking method is:

> **IP Precedence**

IP Precedence uses:

```text
3 bits
```

Therefore:

```text
2³ = 8 possible values
```

This is conceptually similar to CoS in terms of the number of bits available.

---

# 20. Why DSCP Was Introduced

As networks became more complex, the 3-bit marking space became limiting.

There were more applications and more differentiated traffic requirements.

This led to:

> **DSCP — Differentiated Services Code Point**

DSCP expands the marking field to:

```text
6 bits
```

Therefore:

```text
2⁶ = 64 possible values
```

Compare:

| Marking           | Bits | Possible values |
| ----------------- | ---: | --------------: |
| **CoS**           |    3 |               8 |
| **IP Precedence** |    3 |               8 |
| **DSCP**          |    6 |              64 |

This is one of the most useful tables to remember from this lesson.

---

# 21. DSCP

**DSCP = Differentiated Services Code Point**

It provides a much larger marking space than the older 3-bit IP Precedence system.

Conceptually:

```text
IP packet
     ↓
DSCP field
     ↓
6 bits
     ↓
64 possible values
```

This allows more granular traffic classification and treatment.

---

# 22. DSCP and Backward Compatibility

Networking standards often have to deal with existing infrastructure.

DSCP was designed while older systems using IP Precedence existed.

Therefore, the relationship between DSCP and IP Precedence includes backward compatibility considerations.

The important lesson-level takeaway is:

> **DSCP provides a larger marking space while retaining compatibility with older IP Precedence concepts.**

---

# 23. DSCP Names

DSCP values aren't always discussed as raw numbers.

You will often encounter names such as:

```text
AF11
AF12
AF13

AF21
AF22
AF23

...

EF
```

These names communicate information about how traffic should be treated.

The lesson focuses particularly on:

* **AF — Assured Forwarding**
* **EF — Expedited Forwarding**

---

# 24. Assured Forwarding — AF

Assured Forwarding uses a structured naming system.

Example:

```text
AF11
AF12
AF13
```

The first number represents the **traffic class**.

The second number represents the **drop preference**.

So:

```text
AF11
 ││
 │└── Drop preference
 └─── Traffic class
```

---

# 25. Understanding AF11, AF12, AF13

Consider:

```text
AF11
AF12
AF13
```

They belong to the same traffic class because:

```text
1
```

is the first number in all three.

But they have different drop preferences:

```text
AF11 → lower drop preference
AF12 → higher drop preference
AF13 → even higher drop preference
```

Therefore, during congestion:

> **AF11 is preferred over AF12, and AF12 is preferred over AF13 when considering drop preference within that class.**

---

# 26. Higher Class vs Lower Class

Now compare:

```text
AF23
```

with:

```text
AF11
```

The first numbers are:

```text
AF23 → Class 2
AF11 → Class 1
```

The lesson's rule is:

> **Traffic class takes precedence first.**

Therefore:

```text
Class 2 > Class 1
```

even though AF23 has a higher drop-preference number than AF11.

This gives you the basic hierarchy:

```text
FIRST:
Traffic class

THEN:
Drop preference
```

---

# 27. AF Naming Pattern

You don't need to memorize every AF value for CCNA-level understanding.

Instead understand the structure:

```text
AFxy

x = traffic class
y = drop preference
```

Example:

```text
AF23

2 = traffic class
3 = drop preference
```

Another:

```text
AF11

1 = traffic class
1 = drop preference
```

---

# 28. EF — Expedited Forwarding

This is the marking you should especially remember.

> **EF = Expedited Forwarding**

The lesson identifies EF as the marking **typically used for voice traffic**.

Why?

Because voice is highly sensitive to:

* Delay
* Jitter
* Packet loss

So voice needs traffic treatment designed for low latency and predictable delivery.

Think:

```text
VOICE
  ↓
EF
  ↓
Expedited treatment
```

---

# 29. Why Voice Gets Special Treatment

Imagine a congested link:

```text
                  CONGESTED LINK
                       │
        ┌──────────────┼──────────────┐
        ↓              ↓              ↓
      Voice          Camera          Guest
        │              │              │
      EF-marked       Data           Data
```

If all traffic is treated identically, voice can suffer.

With appropriate QoS policy:

```text
Voice
 ↓
Classification
 ↓
EF marking
 ↓
Priority treatment during congestion
```

This improves the likelihood that voice remains usable.

### Important nuance

**EF marking itself doesn't magically guarantee perfect voice quality.**

The network still needs an appropriate QoS policy and sufficient resources.

---

# 30. Classification + Marking

Now combine everything.

```text
                     PACKET
                       │
                       ↓
                 CLASSIFICATION
                       │
          ┌────────────┼────────────┐
          ↓            ↓            ↓
       IP/Port       VLAN        Application
       Protocol      Interface      NBAR
          │            │            │
          └────────────┼────────────┘
                       ↓
                   CLASS MAP
                       │
                       ↓
                    MARKING
                       │
              ┌────────┴────────┐
              ↓                 ↓
             CoS              DSCP
           Layer 2           Layer 3
           3 bits             6 bits
              │                 │
              ↓                 ↓
            0–7              0–63
```

Then downstream devices can use those markings:

```text
                 Marked packet
                      ↓
            ┌─────────┼─────────┐
            ↓         ↓         ↓
         Router 1  Router 2  Router 3
            │         │         │
            └─────────┼─────────┘
                      ↓
             QoS treatment
```

---

# 31. Classification vs Marking — Don't Mix Them Up

This is probably the most important distinction in the lesson.

| Classification                                            | Marking                          |
| --------------------------------------------------------- | -------------------------------- |
| Identifies traffic                                        | Labels traffic                   |
| Determines traffic class                                  | Assigns a value                  |
| “What is this?”                                           | “How should this be recognized?” |
| Uses match criteria                                       | Uses QoS marking fields          |
| Can use IP, ports, protocol, VLAN, interface, application | Can use CoS, DSCP, etc.          |

### Memory trick

> **Classification = Identify**
> **Marking = Label**

---

# 32. CoS vs DSCP

Another distinction you should know well:

| Feature         | CoS                     | DSCP                               |
| --------------- | ----------------------- | ---------------------------------- |
| Full name       | Class of Service        | Differentiated Services Code Point |
| Layer           | Layer 2                 | Layer 3                            |
| Associated with | 802.1Q VLAN tagging     | IP packet                          |
| Bits            | 3                       | 6                                  |
| Possible values | 8                       | 64                                 |
| Range           | 0–7                     | 0–63                               |
| Main purpose    | Layer 2 traffic marking | Layer 3 traffic marking            |

### Memory trick

```text
CoS → Ethernet / VLAN → L2
DSCP → IP → L3
```

---

# 33. IP Precedence vs DSCP

| Feature | IP Precedence        | DSCP                           |
| ------- | -------------------- | ------------------------------ |
| Layer   | Layer 3              | Layer 3                        |
| Bits    | 3                    | 6                              |
| Values  | 8                    | 64                             |
| Role    | Older marking method | Expanded/modern marking method |

The important historical progression is:

```text
IP Precedence
      ↓
Limited 3-bit marking
      ↓
Need for more differentiation
      ↓
DSCP
      ↓
6-bit marking
```

---

# 34. Real-World Castle Rysen Example

Suppose a district coffee shop has:

```text
VLAN 10 → Administration
VLAN 20 → Guest
VLAN 30 → Cameras
VLAN 40 → Voice
```

Traffic enters the network.

### Step 1 — Classification

The network identifies:

```text
Voice      → VOICE class
Cameras    → VIDEO class
Admin      → BUSINESS class
Guest      → GUEST class
```

Classification might use:

```text
IP
Port
Protocol
VLAN
Interface
Application
```

---

### Step 2 — Marking

Traffic is then marked appropriately.

Conceptually:

```text
VOICE
  ↓
EF

Other traffic
  ↓
Appropriate DSCP/CoS marking
```

---

### Step 3 — Downstream treatment

Other network devices can inspect the markings:

```text
Marked packet
      ↓
Downstream router/switch
      ↓
Recognize traffic class
      ↓
Apply QoS policy
```

This avoids repeatedly performing expensive classification throughout the network.

---

# 35. Why Classify Near the Network Edge?

The lesson provides an important real-world recommendation:

> **Classify traffic as close to the edge of the network as possible.**

Why?

Because classification can require processing.

You don't want:

```text
Edge → inspect
Core → inspect again
Distribution → inspect again
WAN → inspect again
```

Instead:

```text
             EDGE
              ↓
         CLASSIFY
              ↓
            MARK
              ↓
       ┌──────┼──────┐
       ↓      ↓      ↓
      Core  Router  WAN
       ↓      ↓      ↓
     Trust/use marking
```

The basic principle is:

> **Classify once, mark it, and let downstream devices use the marking.**

---

# 36. Important Real-World Caveat

The lesson's recommendation to trust markings downstream assumes that the network has a **trust boundary and policy**.

You shouldn't blindly trust markings coming from arbitrary endpoints.

For example:

```text
Unknown endpoint
      ↓
"I am EF!"
      ↓
Network
```

A malicious or misconfigured endpoint could mark its own traffic as high priority.

Therefore, enterprise networks commonly establish **QoS trust boundaries** and decide where markings are trusted, rewritten, or ignored.

The lesson's core principle remains:

> **Classify and appropriately mark traffic near the edge, then use those markings downstream.**

---

# 37. QoS End-to-End Mental Model

Put the entire lesson into one picture:

```text
                  TRAFFIC ENTERS
                        │
                        ↓
                 CLASSIFICATION
                        │
        ┌───────────────┼────────────────┐
        ↓               ↓                ↓
      IP/Port          VLAN          Application
     Protocol        Interface           NBAR
        │               │                │
        └───────────────┼────────────────┘
                        ↓
                    CLASS MAP
                        │
                        ↓
                     MARKING
                        │
              ┌─────────┴─────────┐
              ↓                   ↓
             CoS                DSCP
             L2                  L3
          3 bits               6 bits
          0–7                  0–63
              │                   │
              └─────────┬─────────┘
                        ↓
                DOWNSTREAM DEVICES
                        │
                        ↓
                QoS TREATMENT
                        │
              ┌─────────┼─────────┐
              ↓         ↓         ↓
           Queue      Shape     Police
```

---

# 38. What You Should Actually Memorize

For CCNA, prioritize these concepts rather than trying to memorize every DSCP table.

### Must know

**Classification**

> Identifies traffic based on characteristics such as IP addresses, ports, protocols, interfaces, VLANs, or applications.

**Marking**

> Assigns a value to traffic so downstream devices can recognize and treat it appropriately.

**Class map**

> Groups matching criteria into a traffic class.

**NBAR**

> Network Based Application Recognition; helps identify applications using deeper application recognition.

**CoS**

> Layer 2 marking associated with 802.1Q; 3 bits; values 0–7.

**IP Precedence**

> Older Layer 3 marking method using 3 bits.

**DSCP**

> Layer 3 marking; 6 bits; 64 possible values.

**AF**

> Assured Forwarding; first number represents traffic class, second represents drop preference.

**EF**

> Expedited Forwarding; typically associated with voice traffic.

---

# 39. High-Value Memory Sheet

```text
QoS
│
├── CLASSIFICATION
│     ├── IP address
│     ├── Port
│     ├── Protocol
│     ├── Interface
│     ├── VLAN
│     └── Application / NBAR
│
└── MARKING
      │
      ├── Layer 2
      │    └── CoS
      │         └── 3 bits
      │         └── 0–7
      │
      └── Layer 3
           ├── IP Precedence
           │    └── 3 bits
           │
           └── DSCP
                └── 6 bits
                └── 0–63
                ├── AF
                └── EF
                     └── Voice
```

---

# 40. Final Exam-Oriented Summary

### Classification

**Classification asks:**

> **“What kind of traffic is this?”**

Possible criteria:

```text
IP
Port
Protocol
Interface
VLAN
Application
```

Tools/concepts:

```text
ACL
NBAR
Class map
```

---

### Marking

**Marking asks:**

> **“What label/value should this traffic carry so other devices know how to treat it?”**

Layer 2:

```text
CoS
3 bits
8 values
802.1Q
```

Layer 3:

```text
IP Precedence
3 bits
```

Modern Layer 3:

```text
DSCP
6 bits
64 values
```

Important DSCP concepts:

```text
AF → class + drop preference
EF → typically voice
```

---

## 🔥 The 5 things I'd make sure you can answer without notes

1. **What is the difference between classification and marking?**

   * Classification identifies traffic; marking labels it.

2. **What can be used to classify traffic?**

   * IP addresses, ports, protocols, interfaces, VLANs, and applications.

3. **What is the difference between CoS and DSCP?**

   * CoS is Layer 2/802.1Q and uses 3 bits; DSCP is Layer 3 and uses 6 bits.

4. **What does AF23 mean conceptually?**

   * Traffic class **2**, drop preference **3**.

5. **Which DSCP marking is especially important for voice?**

   * **EF (Expedited Forwarding).**

> **Core QoS chain:**
> **Identify → Classify → Mark → Trust/recognize → Apply treatment**

This lesson is the bridge between **“QoS prioritizes traffic”** and actually understanding **how Cisco devices know which traffic deserves that treatment**.
