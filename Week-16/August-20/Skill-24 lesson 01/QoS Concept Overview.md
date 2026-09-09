# Week 16 — August 20, 2026

# Skill 24 — Quality of Service (QoS)

## Lesson 01: QoS Concept Overview

> **Core idea:**
> **“If there is no congestion, there is no need for Quality of Service.”**

QoS exists because network resources are **finite**. When more traffic wants to use a link than the link can transmit, the network must decide **which traffic gets served first, which traffic waits, and which traffic may be delayed or dropped**.

---

# 1. What is QoS?

**Quality of Service (QoS)** is a collection of network mechanisms used to **identify, classify, mark, queue, shape, and police traffic**, especially when the network is experiencing congestion.

QoS does **not** create additional bandwidth.

Instead:

> **QoS determines how existing bandwidth should be used when multiple types of traffic compete for limited resources.**

### Simple example

Suppose a WAN link has:

```text
Available bandwidth = 100 Mbps
```

At a particular moment:

```text
Voice traffic       = 10 Mbps
Video traffic       = 30 Mbps
Business traffic    = 40 Mbps
Guest downloads     = 50 Mbps
                       --------
Total demand        = 130 Mbps
```

The link can only transmit:

```text
100 Mbps
```

Therefore:

```text
130 Mbps demand
       ↓
100 Mbps capacity
       ↓
CONGESTION
```

QoS allows the network administrator to determine how that 100 Mbps should be treated.

For example, voice might receive priority over a large guest download.

---

# 2. Why Does QoS Exist?

The fundamental reason is:

> **Networks get crowded.**

If bandwidth were unlimited, every packet could be transmitted immediately.

There would be little reason to prioritize one packet over another.

But real networks have:

* Limited WAN bandwidth
* Limited Internet bandwidth
* Oversubscribed uplinks
* Wireless airtime limitations
* Bursty applications
* Many simultaneous users
* Voice and video applications
* Cloud traffic
* Backups and file transfers
* Security-camera traffic

When these compete for the same network resources, **congestion** occurs.

---

# 3. What is Network Congestion?

**Congestion** occurs when:

> **More traffic wants to use a network resource than that resource can handle at that moment.**

Think of a highway.

```text
Many vehicles
     ↓
   Highway
     ↓
Too many vehicles
     ↓
Traffic jam
```

The networking equivalent is:

```text
Many traffic flows
       ↓
     Uplink
       ↓
Insufficient capacity
       ↓
   CONGESTION
```

The important point is that congestion often occurs at a **bottleneck**.

---

# 4. The Classic Congestion Example

Imagine a switch with several connected devices.

```text
PC 1 ── 1 Gbps ──┐
PC 2 ── 1 Gbps ──┤
PC 3 ── 1 Gbps ──┤
PC 4 ── 1 Gbps ──┤
PC 5 ── 1 Gbps ──┤
                  │
               SWITCH
                  │
                  │ 1 Gbps
                  ↓
               UPLINK
```

Each device might have a **1-Gbps connection**.

But the uplink is only **1 Gbps**.

If several devices transmit simultaneously:

```text
1 Gbps
+ 1 Gbps
+ 1 Gbps
+ 1 Gbps
+ ...
      ↓
   Uplink
   1 Gbps
```

The traffic demand can exceed the uplink capacity.

### Important concept

A network does not need every device to continuously transmit at maximum speed for congestion to occur.

Traffic is **bursty**.

A short period where multiple sources transmit simultaneously can create congestion.

---

# 5. Where Can Congestion Occur?

Congestion can occur anywhere there is a resource with insufficient capacity.

Common examples include:

### 5.1 Switch uplinks

```text
Multiple access ports
        ↓
   Distribution
        ↓
     Uplink
```

Many fast access connections may converge onto a smaller uplink.

---

### 5.2 WAN links

WAN links are particularly important because they may have considerably less bandwidth than LAN connections.

```text
Office LAN
   ↓
1 Gbps
   ↓
Router
   ↓
100 Mbps WAN
   ↓
Remote site
```

The LAN can generate traffic faster than the WAN can transmit it.

The WAN becomes the bottleneck.

---

### 5.3 Internet edge

A company's Internet connection can also become congested.

For example:

```text
Employees
Servers
Cameras
Cloud applications
Guest Wi-Fi
       ↓
   Firewall/Router
       ↓
 Internet Link
       ↓
    Internet
```

If everyone uses the Internet simultaneously, the Internet connection may become the congestion point.

This is why users often report:

> **“The Internet is slow.”**

The actual problem may be congestion at the Internet edge.

---

# 6. Wireless Congestion Is Different

Wireless introduces another important limitation:

> **Radio airtime is a shared resource.**

Unlike a switched Ethernet network where devices can have dedicated physical links, wireless clients share the same radio medium.

Conceptually:

```text
Client A ──┐
Client B ──┤
Client C ──┤── Wi-Fi Radio
Client D ──┤
Client E ──┘
```

Clients have to **take turns transmitting**.

As the number of competing clients increases:

```text
More clients
     ↓
More competition for airtime
     ↓
More waiting
     ↓
Potential performance degradation
```

So Wi-Fi congestion isn't simply about Internet bandwidth.

It can also be about **limited radio airtime**.

---

# 7. What Happens During Congestion?

Congestion creates three major problems emphasized in this lesson:

1. **Delay**
2. **Jitter**
3. **Packet loss / Drop**

These are particularly problematic for **real-time applications**, such as:

* VoIP
* Video conferencing
* Streaming
* Interactive voice/video

---

# 8. Delay

**Delay** is the amount of time it takes for traffic to travel through the network.

In the context of congestion, packets may have to **wait in a queue** before they can be transmitted.

```text
Packet arrives
      ↓
Link is busy
      ↓
Packet waits
      ↓
Packet transmitted
```

That waiting contributes to delay.

---

## Real-world example

During a voice call:

```text
You: "Hello"
       ↓
Network delay
       ↓
Other person receives it
```

If the delay becomes noticeable, conversations become awkward.

For example:

```text
Person A talks
      ↓
     delay
      ↓
Person B hears it
```

This can result in people:

* Talking over each other
* Stopping simultaneously
* Waiting unnecessarily
* Repeating themselves

### Troubleshooting translation

User says:

> “Everyone keeps talking over each other on calls.”

Network engineer thinks:

> **Possible latency/delay problem.**

---

# 9. Jitter

**Jitter = variation in packet delay.**

This distinction is extremely important.

Delay means:

> Packets are taking a certain amount of time to arrive.

Jitter means:

> **The amount of delay is changing from packet to packet.**

Example:

```text
Packet 1 → 80 ms
Packet 2 → 90 ms
Packet 3 → 70 ms
Packet 4 → 110 ms
Packet 5 → 75 ms
```

The packets aren't arriving at a consistent rate.

That's **jitter**.

---

## Delay vs Jitter

| Concept              | Meaning                                              |
| -------------------- | ---------------------------------------------------- |
| **Delay**            | Time taken for packets to travel                     |
| **Jitter**           | Variation in that delay                              |
| **Packet loss/drop** | Packets don't arrive or arrive too late to be useful |

### Easy memory trick

```text
Delay  = "How late?"
Jitter = "How inconsistent is the lateness?"
Drop   = "Did it make it?"
```

---

# 10. Why Jitter Is Bad for Voice and Video

Voice and video are **time-sensitive**.

Imagine audio packets arriving like this:

```text
Expected:

1 ── 2 ── 3 ── 4 ── 5
```

But because of jitter:

```text
1 ───── 2 ─ 3 ───────── 4 ─ 5
```

The receiving device has to deal with packets arriving at inconsistent times.

Too much variation can produce:

* Choppy audio
* Distorted conversations
* Video interruptions
* Unnatural playback

---

# 11. Packet Loss / Drop

A **drop** occurs when a packet doesn't successfully reach its destination or arrives too late to be useful.

This is also commonly called:

> **Packet loss**

Example:

```text
Packet 1 ✓
Packet 2 ✓
Packet 3 ✗
Packet 4 ✓
Packet 5 ✓
```

Packet 3 was lost.

For applications such as voice and video, this can be noticeable.

---

## Symptoms of packet loss

Users might report:

* “The call keeps cutting out.”
* “The audio sounds robotic.”
* “The video is blocky.”
* “The meeting keeps freezing.”
* “The connection feels unstable.”

These are **user-facing symptoms** of possible network problems.

---

# 12. Translate User Complaints Into Network Problems

This is an important real-world networking skill.

Users normally won't say:

> “There is excessive jitter on the WAN interface.”

Instead they'll say:

> “My Teams call sounds weird.”

or:

> “The video keeps freezing.”

Your job as a network engineer is to translate:

```text
User complaint
      ↓
Observed symptom
      ↓
Possible network characteristic
      ↓
Troubleshooting
```

Examples:

| User complaint                | Possible network symptom           |
| ----------------------------- | ---------------------------------- |
| Calls have awkward pauses     | Delay                              |
| People talk over each other   | Delay                              |
| Audio is inconsistent/choppy  | Jitter                             |
| Voice sounds robotic          | Packet loss/jitter                 |
| Video freezes                 | Packet loss/congestion             |
| Internet slows during backups | Congestion                         |
| Calls break during peak hours | Congestion + insufficient priority |

These are **possible causes**, not automatic diagnoses. Troubleshooting requires measurement and verification.

---

# 13. Jitter Buffer

A **jitter buffer** is a mechanism used to compensate for variation in packet arrival times.

Think of it as a small waiting room.

Instead of immediately playing a packet when it arrives:

```text
Packet arrives
     ↓
Play immediately
```

the system can temporarily buffer packets:

```text
Packets arrive
      ↓
Jitter Buffer
      ↓
Smooth timing
      ↓
Playback
```

---

# 14. Why Does a Jitter Buffer Help?

Suppose packets arrive like this:

```text
Packet 1 → 80 ms
Packet 2 → 90 ms
Packet 3 → 70 ms
Packet 4 → 100 ms
```

The receiver can temporarily hold packets and smooth out small timing differences.

Conceptually:

```text
Irregular arrival
       ↓
[JITTER BUFFER]
       ↓
More consistent playback
```

This can make voice/video appear smoother.

---

# 15. Jitter Buffer Limitation

A jitter buffer isn't magic.

It can compensate for **small variations**.

But if packets arrive far too late, the buffer can't hide the problem indefinitely.

Eventually:

```text
Excessive delay/jitter
        ↓
Packet arrives too late
        ↓
Not useful anymore
        ↓
Effectively treated as a drop
```

### Key idea

> **A jitter buffer can smooth variation, but it cannot repair a fundamentally poor network.**

---

# 16. QoS Does NOT Increase Bandwidth

This is one of the most important points from the lesson.

### Wrong:

> “QoS gives me more bandwidth.”

### Correct:

> **QoS does not create additional bandwidth.**

If you have:

```text
100 Mbps link
```

QoS doesn't turn it into:

```text
150 Mbps
```

The physical capacity remains:

```text
100 Mbps
```

What QoS changes is **how that capacity is allocated and treated during congestion**.

---

# 17. The QoS Mental Model

Think of a road.

```text
Road capacity = fixed
```

You cannot create another lane just by installing QoS.

Instead, QoS can determine:

```text
Emergency vehicles → priority
Normal vehicles    → normal treatment
Large trucks       → controlled flow
```

Networking works similarly.

```text
Limited bandwidth
       ↓
Multiple traffic types
       ↓
Classification
       ↓
Marking
       ↓
Queueing / Shaping / Policing
       ↓
Controlled treatment
```

---

# 18. The Five Major QoS Concepts

The lesson introduces five major QoS mechanisms/concepts:

1. **Classification**
2. **Marking**
3. **Queuing**
4. **Shaping**
5. **Policing**

Memorize this sequence.

```text
CLASSIFY
   ↓
MARK
   ↓
QUEUE
   ↓
SHAPE / POLICE
```

The exact implementation can vary, but this gives you the conceptual workflow.

---

# 19. Classification

**Classification** means determining what type of traffic a packet belongs to.

Essentially:

> **“What is this traffic?”**

Examples:

```text
Voice
Video
Web
Backups
Guest traffic
Business applications
```

A device can use different characteristics to identify traffic.

The lesson mentions examples such as:

* IP addresses
* Port numbers
* Other match criteria

Conceptually:

```text
Packet
  ↓
Examine characteristics
  ↓
Determine traffic type
  ↓
Assign it to a class
```

---

# 20. Marking

After traffic is classified, it can be **marked**.

Marking means:

> **Adding an indication of how the traffic should be treated.**

Think of it as putting a label on the packet.

```text
VOICE packet
     ↓
[HIGH PRIORITY MARK]
```

Downstream devices can then recognize the marking instead of having to perform the same classification process from scratch.

### Important distinction

```text
Classification → "What is this?"
Marking        → "How should this traffic be recognized/treated?"
```

---

# 21. Why Marking Is Useful

Imagine traffic passing through several network devices:

```text
Device A
   ↓
Device B
   ↓
Device C
   ↓
Device D
```

If Device A identifies voice traffic and marks it, downstream devices can use that marking.

Conceptually:

```text
Classify
   ↓
Mark
   ↓
========================
↓          ↓           ↓
Device B  Device C   Device D
========================
      recognize mark
```

This allows QoS treatment to be applied consistently across the network.

---

# 22. Queuing

**Queuing** determines how packets waiting for transmission are handled.

When a link is congested:

```text
Packet A ─┐
Packet B ─┤
Packet C ─┤→ Queue → Link
Packet D ─┤
Packet E ─┘
```

Packets may have to wait.

QoS can influence which traffic receives preferential treatment.

Conceptually:

```text
Voice       → Priority
Business    → Important
Guest       → Lower priority
Backup      → Lower priority
```

So when congestion occurs:

```text
Who gets served first?
        ↓
QoS queueing decision
```

---

# 23. Shaping

**Traffic shaping** smooths traffic by temporarily **buffering traffic** rather than transmitting bursts immediately.

Think:

> **“Slow down and smooth the traffic.”**

Example:

```text
Large traffic burst
████████████████████
          ↓
       SHAPER
          ↓
████  ████  ████  ████
```

Instead of sending a huge burst all at once, shaping can spread the traffic over time.

### Key characteristic

**Shaping generally buffers traffic.**

It is therefore a more controlled/gentle mechanism.

---

# 24. Policing

**Traffic policing** is more aggressive.

Policing establishes a traffic rate or limit and can intentionally **drop traffic that exceeds the configured rate**.

Think:

> **“You are using too much. Stop.”**

Conceptually:

```text
Traffic
   ↓
POLICER
   ↓
Within limit → Allowed
Over limit   → Dropped
```

### Key characteristic

**Policing can drop excess traffic.**

---

# 25. Shaping vs Policing

This distinction is highly important.

| Feature        | Shaping          | Policing       |
| -------------- | ---------------- | -------------- |
| Main purpose   | Smooth traffic   | Enforce a rate |
| Excess traffic | Usually buffered | Can be dropped |
| Behavior       | Gentler          | Stricter       |
| Mental model   | “Slow down”      | “Stop”         |

### Memory trick

> **Shape = Save it temporarily.**
> **Police = Punish excess.**

---

# 26. All Five QoS Mechanisms Together

Imagine Castle Rysen Coffee has this traffic:

```text
                     Network Link
                         │
      ┌──────────────────┼──────────────────┐
      ↓                  ↓                  ↓
     VoIP              Video             Guest
      │                  │                  │
      └────────────── Traffic ──────────────┘
                         │
                         ↓
                   CLASSIFICATION
                         │
                         ↓
                      MARKING
                         │
                         ↓
                      QUEUING
                         │
                  ┌──────┴──────┐
                  ↓             ↓
               SHAPING       POLICING
                  │             │
                  ↓             ↓
             Smooth bursts   Limit/drop
```

The purpose is not to make the link faster.

The purpose is to make the link **behave intelligently when demand exceeds capacity**.

---

# 27. Castle Rysen Coffee Example

Consider a district coffee shop.

The RFP requires the shop to support things such as:

* Business/administrative devices
* Guest Wi-Fi
* Plex video streaming
* Video surveillance
* Internet connectivity

The RFP specifically calls for QoS features to help monitor and manage the network. 

Imagine the Internet/WAN link is busy:

```text
                    WAN LINK
                      │
       ┌──────────────┼──────────────┐
       ↓              ↓              ↓
      VoIP          Cameras        Guest
       │              │              │
       │              │              │
   Critical        Important        Bulk
```

A guest might be downloading a huge file while a business voice call is occurring.

Without QoS:

```text
All traffic competes
       ↓
Congestion
       ↓
Voice may suffer
```

With QoS:

```text
Identify voice
      ↓
Classify
      ↓
Mark
      ↓
Give appropriate priority
      ↓
Voice has better treatment during congestion
```

The guest traffic isn't necessarily stopped.

Instead, **critical traffic receives better treatment when resources are scarce**.

---

# 28. What QoS Is Really Solving

QoS is fundamentally a **resource-allocation problem**.

You have:

```text
LIMITED RESOURCE
       +
COMPETING TRAFFIC
       ↓
CONGESTION
       ↓
TRAFFIC MANAGEMENT
       ↓
QoS
```

Without QoS, treatment may effectively be based on the default queueing behavior.

With QoS, the network administrator can establish deliberate policies.

---

# 29. QoS Is About Trade-offs

Suppose a link has only:

```text
100 Mbps
```

but traffic demand is:

```text
Voice       10 Mbps
Business    30 Mbps
Video       30 Mbps
Guest       50 Mbps
--------------------
Total      120 Mbps
```

There is no way to transmit all 120 Mbps simultaneously through a 100-Mbps link.

Something has to happen.

Possible strategies include:

```text
Priority
Queueing
Shaping
Policing
Dropping
```

QoS gives the administrator control over those trade-offs.

---

# 30. The Most Important CCNA Takeaways

### 1. QoS is primarily about congestion

> **No congestion → QoS generally isn't needed for prioritization.**

---

### 2. QoS doesn't increase bandwidth

```text
QoS ≠ bandwidth upgrade
```

It manages existing capacity.

---

### 3. Congestion creates problems

The lesson focuses on:

```text
Delay
Jitter
Packet loss / Drop
```

---

### 4. Voice and video are especially sensitive

Real-time applications are much less tolerant of delay, jitter, and packet loss than many ordinary data applications.

---

### 5. Classification identifies traffic

```text
"What is this traffic?"
```

---

### 6. Marking labels traffic

```text
"How should other devices recognize this traffic?"
```

---

### 7. Queuing determines who gets served

```text
"Who gets transmitted first?"
```

---

### 8. Shaping smooths traffic

```text
"Slow down and buffer bursts."
```

---

### 9. Policing enforces limits

```text
"You're exceeding the allowed rate → limit/drop."
```

---

# 31. Quick Comparison

| QoS Concept        | Question It Answers                    | Basic Action     |
| ------------------ | -------------------------------------- | ---------------- |
| **Classification** | What is this traffic?                  | Identify         |
| **Marking**        | How should it be recognized?           | Label            |
| **Queuing**        | Which traffic gets served first?       | Prioritize/order |
| **Shaping**        | How can traffic bursts be smoothed?    | Buffer           |
| **Policing**       | Is traffic exceeding its allowed rate? | Limit/drop       |

---

# 32. Delay vs Jitter vs Drop

| Problem    | Definition                              | Example symptom             |
| ---------- | --------------------------------------- | --------------------------- |
| **Delay**  | Packet takes longer to arrive           | People talk over each other |
| **Jitter** | Variation in packet delay               | Choppy/inconsistent audio   |
| **Drop**   | Packet is lost or too late to be useful | Audio cuts/video artifacts  |

### Remember:

```text
DELAY  → Late
JITTER → Uneven timing
DROP   → Missing
```

---

# 33. Final Mental Model

When troubleshooting QoS, start with this chain:

```text
                 Is there congestion?
                         │
                ┌────────┴────────┐
                │                 │
               NO                YES
                │                 │
          QoS priority       What traffic
          isn't needed       is competing?
                                  │
                                  ↓
                            CLASSIFY
                                  │
                                  ↓
                              MARK
                                  │
                                  ↓
                              QUEUE
                                  │
                         ┌────────┴────────┐
                         ↓                 ↓
                      SHAPE             POLICE
                         ↓                 ↓
                  Buffer/smooth       Limit/drop
                         │                 │
                         └────────┬────────┘
                                  ↓
                        Better use of
                        existing capacity
```

## 🔑 One-line exam memory

> **QoS does not make the network faster; it makes the network smarter about how it uses limited bandwidth during congestion.**

And the core workflow to remember:

> **Classify → Mark → Queue → Shape/Police**

This is the foundation for the next lesson, **Classification and Marking**, where the focus moves from *why QoS exists* to *how the network identifies and labels traffic*.
