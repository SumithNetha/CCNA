# Week 16 — August 21

# Skill 24 Lesson 04 — What Now?

This is the **wrap-up lesson for QoS**. Your study plan places it immediately after **“QoS Treatment Methods (Queuing, Shaping, Policing)”** and the associated lab. 

The main purpose of this lesson isn't to introduce another major configuration technique. It is to make sure you understand **when QoS is useful and why it matters in a real network**.

---

# 1. The Big Picture of QoS

Don't reduce QoS to:

> "QoS = prioritize voice."

That's only one use case.

The bigger idea is:

> **QoS controls how different traffic is treated when network resources become constrained.**

Imagine:

```text
                WAN LINK
                  │
             100 Mbps
                  │
       ┌──────────┼──────────┐
       │          │          │
      Voice     Business    Guest
       │          │          │
       └──────────┼──────────┘
                  │
             CONGESTION
                  │
                  ▼
                  QoS
```

When everything fits comfortably:

```text
Traffic demand < Available bandwidth
```

QoS isn't doing much.

But when:

```text
Traffic demand > Available bandwidth
```

QoS becomes important.

---

# 2. QoS Does Not Create Bandwidth

This is the most important takeaway.

Suppose you have:

```text
100 Mbps WAN
```

QoS cannot turn that into:

```text
200 Mbps WAN
```

Instead:

```text
100 Mbps
    +
Traffic prioritization
    +
Traffic management
    ↓
Better behavior during congestion
```

So:

> **QoS manages limited bandwidth; it does not manufacture additional bandwidth.**

---

# 3. Why Voice Is Used So Often as an Example

Voice makes QoS problems easy to notice.

Without appropriate treatment, congestion can cause:

* choppy audio
* delays
* awkward pauses
* people talking over each other

For example:

```text
Voice packet
     ↓
Congested link
     ↓
Delayed
     ↓
Jitter
     ↓
Poor call quality
```

Because humans immediately notice problems with a phone conversation, voice is an excellent example for understanding QoS.

But **QoS is much broader than voice**.

---

# 4. QoS Can Protect Business-Critical Applications

Imagine a company WAN carrying:

```text
                    WAN
                     │
       ┌─────────────┼─────────────┐
       │             │             │
   Payments        Voice        Guest Wi-Fi
       │             │             │
   Critical        High          Lower
```

Should all traffic necessarily receive identical treatment?

No.

If guest users are streaming large amounts of video while payment transactions are trying to cross the WAN, you don't want:

```text
"First come, first served."
```

You want the traffic that is most important to the business to receive appropriate treatment.

---

# 5. NetworkChuck Coffee Example

The lesson applies this directly to **NetworkChuck Coffee**.

Imagine multiple coffee shops connected over a WAN.

Traffic might include:

```text
POS / payment systems
Inventory synchronization
Security cameras
Voice calls
Guest Wi-Fi
Back-office applications
```

Now imagine congestion.

```text
              WAN
               │
        ┌──────┼─────────┐
        │      │         │
       POS    Voice    Guest
        │      │         │
        └──────┼─────────┘
               │
          CONGESTION
```

You don't necessarily want every application competing equally.

The business-critical traffic should receive appropriate priority.

For example:

```text
HIGHER PRIORITY
    ↓
Payment transactions
Voice/support calls
Business applications

LOWER PRIORITY
    ↓
Guest browsing
Less-critical transfers
```

The exact policy depends on the organization's requirements.

---

# 6. QoS Is a Business Decision

This is an important shift in thinking.

QoS isn't simply:

> "Which protocol should I configure?"

Instead, first ask:

> **Which applications are important to the organization?**

For example:

### Coffee shop

```text
Payment systems
       ↓
Business critical

Guest Wi-Fi
       ↓
Lower priority
```

### Bank

```text
ATM transactions
       ↓
Critical

Account transactions
       ↓
Critical

Internal applications
       ↓
Important

Web browsing
       ↓
Lower priority
```

The technology comes **after** understanding the business requirements.

---

# 7. Banking Example

The lesson uses a private banking network as another example.

Imagine several branches connected through a private network.

Traffic includes:

* ATM traffic
* account transactions
* internal applications
* voice
* web browsing

Now congestion occurs.

Which traffic should receive preferential treatment?

```text
ATM / Transactions
        ↓
     Critical
        ↓
   Appropriate QoS
```

while less-critical traffic can receive less preferential treatment.

The key lesson:

> **QoS protects important business functions when everything cannot be transmitted equally at the same time.**

---

# 8. ISP Example

QoS isn't limited to enterprise networks.

ISPs can establish different traffic/service classes.

The lesson gives the conceptual example:

```text
Gold
Silver
Bronze
```

These classes can receive different treatment.

So:

```text
Packet A → Gold
Packet B → Bronze
```

may receive different treatment because of the QoS policies applied within the provider's network.

The important concept is **service differentiation**.

---

# 9. QoS Doesn't Mean "Everything Important Goes First"

This is a subtle but important point.

You shouldn't simply classify everything as:

```text
HIGH PRIORITY
```

because then nothing is meaningfully prioritized.

Instead, you need to determine:

```text
What is critical?
What is important?
What can wait?
What can be limited?
```

For example:

```text
             TRAFFIC
                │
       ┌────────┼────────┐
       ↓        ↓        ↓
    Critical  Important  Best effort
       │        │        │
       ↓        ↓        ↓
    Priority   Class     Normal
```

QoS is therefore about **policy and trade-offs**.

---

# 10. Recognizing When You Need QoS

A very useful real-world skill is recognizing the symptoms.

Suppose someone tells you:

> "Everything works fine until the WAN gets busy. Then our calls become terrible and our important applications slow down."

You should immediately start thinking:

```text
CONGESTION
    ↓
QoS?
```

You don't necessarily know the exact configuration yet.

But you've identified the **solution category**.

That's valuable troubleshooting knowledge.

---

# 11. What QoS Tools Have You Learned?

At this point, your QoS toolbox includes:

### Classification

Determine what type of traffic you're dealing with.

```text
"What is this?"
```

### Marking

Give traffic a QoS label.

```text
"How should this traffic be treated?"
```

### Trust boundary

Determine where QoS markings should be trusted.

```text
"Do I trust this marking?"
```

### Queuing

Determine which traffic gets transmission opportunities when congestion exists.

```text
"Who gets served first?"
```

### WRED

Begin dropping selected traffic before queues completely fill.

```text
"Can I start controlling congestion early?"
```

### Shaping

Buffer excess traffic and transmit it later.

```text
"Can I slow this down without immediately dropping it?"
```

### Policing

Enforce a traffic-rate limit.

```text
"You're over the limit."
```

---

# 12. The Complete QoS Mental Model

You should now be able to visualize the entire process:

```text
                         TRAFFIC
                            │
                            ▼
                    CLASSIFICATION
                            │
                    "What is it?"
                            │
                            ▼
                         MARKING
                            │
                    "How important?"
                            │
                            ▼
                     TRUST BOUNDARY
                            │
                    "Do I trust it?"
                            │
                            ▼
                       CONGESTION
                            │
             ┌──────────────┼──────────────┐
             ▼              ▼              ▼
          QUEUING         WRED          SHAPING
             │              │              │
        Who goes first?   Drop early    Buffer
             │              │           excess
             │              │              │
             └──────────────┼──────────────┘
                            │
                            ▼
                        POLICING
                            │
                     Enforce limits
                            │
                            ▼
                        TRANSMIT
```

---

# 13. The Most Important Real-World Skill

The lesson makes an important career point:

You don't always need to know every command immediately.

A valuable network engineer can look at a problem and say:

> **"This is congestion, and QoS may be the appropriate tool."**

That's much better than staring at the problem without knowing what technology could address it.

The progression is:

```text
Problem
  ↓
Recognize the symptoms
  ↓
Identify the technology
  ↓
Choose the appropriate mechanism
  ↓
Configure it
  ↓
Verify the result
```

You're currently strengthening the **technology-selection** part.

---

# 14. Don't Memorize QoS as "Phone Traffic"

Your mental model should be:

### ❌ Too narrow

```text
QoS → Voice
```

### ✅ Correct

```text
QoS
 ↓
Traffic prioritization and management
 ↓
Used when resources are constrained
 ↓
Protect important applications
```

Voice is simply one particularly obvious example.

---

# 15. QoS in the Castle Rysen RFP

This is directly connected to your larger Castle Rysen project.

The RFP requires **QoS features** as part of the network monitoring and management requirements. 

The district shops also have multiple traffic types, including:

* administrative traffic
* guest traffic
* Plex video
* video surveillance

The RFP specifically requires the district shop to support Plex-based video streaming and surveillance while maintaining network segmentation. 

That gives you a realistic environment in which QoS policies could become relevant.

For example:

```text
Castle Rysen District Shop
             │
             ▼
          WAN Link
             │
     ┌───────┼────────┐
     │       │        │
    Voice   Plex    Guest
     │       │        │
     ▼       ▼        ▼
   High    Important  Lower
```

The exact priority hierarchy should come from the business requirements rather than from a generic rule.

---

# 16. What You Should Remember for CCNA

### 1. QoS doesn't create bandwidth.

```text
QoS ≠ more bandwidth
```

It manages bandwidth when resources are constrained.

### 2. QoS is broader than voice.

It can be used for:

* voice
* critical applications
* transaction systems
* video
* carrier traffic classes
* other business-critical traffic

### 3. Recognizing the need for QoS is important.

If you see:

```text
Congestion
+
Important traffic suffering
```

think:

```text
QoS
```

---

# 17. One-Minute Revision

If you only revise one thing from this lesson, remember:

```text
                 QoS
                  │
       "Everything can't win."
                  │
                  ▼
         Decide what matters
                  │
                  ▼
      Protect important traffic
                  │
                  ▼
      Manage congestion intelligently
```

And:

> **QoS is not about making the network faster. It is about making the network behave intelligently when there isn't enough bandwidth for everything.**

---

## Skill 24 Complete

You have now covered the conceptual QoS progression:

```text
QoS Concept
    ↓
Classification & Marking
    ↓
Trust Boundary
    ↓
Queuing
    ↓
WRED
    ↓
Shaping
    ↓
Policing
    ↓
Real-world QoS decisions
```

Your study plan then moves into **Week 17: Wireless**, beginning August 24. 
