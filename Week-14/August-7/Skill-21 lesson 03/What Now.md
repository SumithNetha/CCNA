# Skill 21 — What Now?

## CDP & LLDP: Practical Network Discovery

This lesson is less about learning another command and more about developing a **real-world troubleshooting workflow**.

The main idea is:

> **Get onto one known network device → discover its neighbors → follow the connections → build the network map.**

---

## 1. Discovery Is the Starting Point

CDP and LLDP are most valuable when you're dealing with an **unknown or poorly documented network**.

Imagine entering a wiring closet and seeing:

```text
Switch?
Router?
Firewall?
Uplink?
Mystery cable?
```

Instead of guessing, start with a device you can access.

For example:

```cisco
show cdp neighbors
```

or the LLDP equivalent:

```cisco
show lldp neighbors
```

The output gives you clues about directly connected devices.

---

## 2. One Device → Many Devices

The workflow is essentially recursive.

```text
              Start here
                  │
                  ▼
               Switch 1
              /         \
             ▼           ▼
         Switch 2       Router
          /   \            │
         ▼     ▼           ▼
       AP     SW3       Upstream
```

You don't need to know the entire topology beforehand.

You discover it **one hop at a time**.

> **One device turns into two. Two turns into ten.**

Eventually, the network stops looking like a collection of random devices and starts looking like a topology you can reason about.

---

# 3. The Practical Network-Discovery Workflow

The lesson gives a very straightforward workflow.

### Step 1 — Find a known device

Usually, this means obtaining access through the **console**.

```text
Console
   ↓
Network Device
```

You need somewhere to start.

---

### Step 2 — Run neighbor discovery

For Cisco CDP:

```cisco
show cdp neighbors
```

For LLDP:

```cisco
show lldp neighbors
```

---

### Step 3 — Identify connected devices

Pay attention to:

* Neighbor device
* Local interface
* Remote interface
* Other available information

For example:

```text
Local Interface → Neighbor
Gi0/1           → SW2
Gi0/2           → Router
Gi0/3           → AP
```

Now you know what is attached to those ports.

---

### Step 4 — Move to the next device

If `SW2` is connected to your original switch, access SW2 and repeat the process.

```text
SW1
 │
 └── SW2
      │
      ├── SW3
      └── Router
```

---

### Step 5 — Document everything

Don't keep the topology exclusively in your head.

Create a basic map:

```text
SW1 Gi0/1 ─── SW2 Gi0/24
SW1 Gi0/2 ─── R1 Gi0/0
SW2 Gi0/5 ─── AP1
```

It doesn't need to be a beautiful Visio diagram initially.

Even a rough sketch is valuable.

---

# 4. Why Documentation Matters

Once you've discovered the topology, you now understand:

* What is connected to what
* Where traffic might be flowing
* Which device is the next hop
* Which interfaces connect devices
* Where to begin troubleshooting

This directly reduces the amount of guessing involved.

For example, suppose someone reports:

> **"The register can't process cards."**

Instead of randomly checking devices, you can follow the topology:

```text
POS
 ↓
Access Switch
 ↓
Distribution/Core
 ↓
Router
 ↓
Internet
```

You now have a logical path to investigate.

---

# 5. Don't Trust Existing Labels

One of the strongest practical lessons here is:

> **Verify the network instead of trusting assumptions.**

Don't automatically believe:

```text
"That's the core switch."
```

Don't automatically believe:

```text
"That cable goes to the router."
```

And don't rely on:

```text
"I think this is the uplink."
```

Use actual network information to verify it.

### Better approach

```text
Known Device
     ↓
CDP / LLDP
     ↓
Identify Neighbor
     ↓
Verify Interface
     ↓
Document
```

This turns **tribal knowledge** into actual network documentation.

---

# 6. Why There Isn't a Huge Lab

The lesson intentionally doesn't finish with a complicated CDP/LLDP lab.

The reason is important.

The valuable skill isn't:

> "Can I configure LLDP in a giant Packet Tracer topology?"

The valuable skill is:

> **"Can I use neighbor discovery when I encounter an unfamiliar network?"**

The actual workflow is simple:

```text
Access device
     ↓
Run discovery command
     ↓
Follow clues
     ↓
Map topology
     ↓
Troubleshoot
```

Training labs can make this seem more complicated than it really is.

---

# 7. NetworkChuck Coffee Example

Imagine a new NetworkChuck Coffee location has been inherited from a previous installer.

You find:

```text
Unlabeled switches
Mystery uplinks
Unknown router
No documentation
```

You obtain console access to a known device.

Then:

```cisco
show cdp neighbors
```

or:

```cisco
show lldp neighbors
```

You might discover:

```text
SW1
 ├── SW2
 ├── Router
 └── AP
```

Then you move to SW2:

```text
SW2
 ├── POS Switch
 ├── Back-office device
 └── AP
```

And gradually construct:

```text
                         Internet
                            │
                           R1
                            │
                           SW1
                       ┌────┼────┐
                       │    │    │
                      SW2   AP1  SW3
                     /  \
                   POS  Office
```

Now you have something much more useful than a guess.

You have a **network map**.

---

# 8. The Most Important Real-World Principle

When inheriting an unfamiliar network:

### Don't trust:

* Labels
* Memory
* Assumptions
* "I think..."
* Tribal knowledge

### Do:

* Verify
* Discover
* Trace
* Document

This is especially important when the network is already supporting production services.

---

# 9. CDP and LLDP as Network-Detective Tools

Think of these protocols as **clues**.

You don't necessarily use them to solve the entire problem by themselves.

Instead:

```text
CDP / LLDP
     ↓
Discovery information
     ↓
Topology understanding
     ↓
Better troubleshooting decisions
```

For example:

```text
"I can't reach the Internet."
```

is a problem statement.

But after discovery you might know:

```text
Client
  ↓
SW2
  ↓
SW1
  ↓
R1
  ↓
Internet
```

Now you can systematically investigate each hop.

---

# 10. What This Adds to Your CCNA Skillset

You've already learned many technologies that **operate the network**:

```text
VLANs
STP
EtherChannel
Routing
OSPF
FHRP
IPv6
ACLs
Layer 2 Security
```

CDP and LLDP add something slightly different:

```text
              NETWORK
                 │
     ┌───────────┴───────────┐
     │                       │
Operating the network     Understanding it
     │                       │
 VLAN/STP/OSPF/etc.       CDP/LLDP
     │                       │
     └───────────┬───────────┘
                 │
          Troubleshooting
```

That's why this lesson is practical rather than configuration-heavy.

---

# 🧠 Final Takeaways

### 1. Start somewhere

You don't need to understand the whole network immediately.

> **Find one device you can access.**

### 2. Discover neighbors

```cisco
show cdp neighbors
show lldp neighbors
```

### 3. Follow the clues

Move from:

```text
Device → Neighbor → Neighbor → Neighbor
```

### 4. Build a map

Document interfaces and connections as you discover them.

### 5. Verify instead of guessing

Existing labels and assumptions may be wrong.

### 6. Use discovery to troubleshoot

Understanding topology makes troubleshooting much more systematic.

---

## 🔑 The one workflow to remember

```text
        UNKNOWN NETWORK
               │
               ▼
        Find known device
               │
               ▼
          Get console
            access
               │
               ▼
       CDP / LLDP command
               │
               ▼
       Identify neighbors
               │
               ▼
        Record interfaces
               │
               ▼
       Connect to next device
               │
               ▼
        Repeat discovery
               │
               ▼
        Build network map
               │
               ▼
          Troubleshoot
```

**The real lesson of Skill 21 isn't "memorize CDP and LLDP."**

It's:

> **When you don't understand a network, don't guess. Start with one known device and make the network reveal itself one hop at a time.**
