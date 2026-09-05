# Week 14 — August 7, 2026

# Skill 21 — Using Link Layer Discovery Protocol (LLDP)

## 1. What is LLDP?

**LLDP — Link Layer Discovery Protocol** is a **standards-based Layer 2 neighbor discovery protocol**.

Its purpose is to allow directly connected network devices to discover each other and exchange useful information such as:

* Device identity
* Device capabilities
* Interface information
* Software/platform information
* IP addressing information
* In some environments, information related to power requirements or VLANs

The simplest way to remember it:

> **CDP = Cisco's neighbor discovery**
>
> **LLDP = industry-standard neighbor discovery**

LLDP is especially important in **multi-vendor networks**.

---

# 2. Why Do We Need LLDP?

Imagine walking into a network closet with:

* No network diagram
* No documentation
* Poorly labeled cables
* Several switches
* Routers
* Access points
* IP phones
* Devices from different vendors

Your first question is:

> **"What is connected to what?"**

Neighbor discovery protocols can answer that.

Without a discovery protocol, you may have to physically trace cables and manually investigate every interface.

With LLDP, devices can advertise information about themselves to directly connected neighbors.

### Example

```text
             LLDP
      ┌─────────────────┐
      │                 │
      │     Switch A    │
      │                 │
      └───────┬─────────┘
              │
              │ Ethernet
              │
      ┌───────▼─────────┐
      │                 │
      │     Switch B    │
      │                 │
      └─────────────────┘
```

Switch A can learn information about Switch B, while Switch B can learn information about Switch A.

This helps an administrator reconstruct the topology.

---

# 3. LLDP vs CDP

This is one of the most important comparisons from this lesson.

| Feature                 | CDP                                   | LLDP                                |
| ----------------------- | ------------------------------------- | ----------------------------------- |
| Full name               | Cisco Discovery Protocol              | Link Layer Discovery Protocol       |
| Type                    | Neighbor discovery protocol           | Neighbor discovery protocol         |
| Layer                   | Layer 2                               | Layer 2                             |
| Standardization         | Cisco proprietary                     | Industry standard                   |
| Vendor interoperability | Primarily Cisco ecosystem             | Multi-vendor                        |
| Purpose                 | Discover directly connected devices   | Discover directly connected devices |
| Cisco ↔ Cisco           | ✅                                     | ✅                                   |
| Cisco ↔ non-Cisco       | Limited depending on protocol support | ✅                                   |
| Cisco command to enable | `cdp run`                             | `lldp run`                          |

### Core distinction

**CDP is vendor-specific.**

**LLDP is designed for interoperability.**

That makes LLDP particularly useful when your network contains equipment from different manufacturers.

---

# 4. Multi-Vendor Networks

Real networks aren't necessarily 100% Cisco.

You might encounter something like:

```text
Cisco Switch
      │
      │
      ▼
   HP Switch
      │
      │
      ▼
 Ubiquiti AP
      │
      │
      ▼
 IP Phone
```

If you rely exclusively on Cisco-specific discovery, you may not get the same visibility across the entire environment.

LLDP provides a common discovery mechanism.

### Real-world example

A company might have:

* Cisco switches
* HPE switches
* Ubiquiti wireless equipment
* IP phones
* Servers from another vendor

LLDP can provide a common neighbor-discovery mechanism across these devices, assuming the devices support and have LLDP enabled.

---

# 5. Why LLDP Is Important for Troubleshooting

Suppose you inherit this network:

```text
SW1 Gi0/1 ───── ??? 
SW1 Gi0/2 ───── ???
SW1 Gi0/3 ───── ???
SW1 Gi0/4 ───── ???
```

You don't know what's connected.

Instead of physically tracing every cable, you can use LLDP.

```text
SW1# show lldp neighbors
```

The switch can show information about its directly connected neighbors.

This allows you to start building a topology:

```text
             Gi0/1
SW1 ───────────────── SW2

             Gi0/2
SW1 ───────────────── R1

             Gi0/3
SW1 ───────────────── AP1
```

This is extremely useful during:

* Network troubleshooting
* Network documentation
* Network migrations
* Device replacement
* Infrastructure audits
* Unknown-network reconnaissance
* Wiring-closet investigations

---

# 6. LLDP Is Not Enabled by Default on Many Cisco Devices

This is an important point from the lesson.

The lesson emphasizes that **LLDP is often not enabled by default on Cisco equipment**.

Therefore, simply running:

```text
show lldp neighbors
```

doesn't necessarily mean you'll see useful information.

You first need LLDP running.

---

# 7. Enabling LLDP

On Cisco IOS:

```cisco
configure terminal
lldp run
```

Example:

```cisco
Switch# configure terminal
Switch(config)# lldp run
```

This enables LLDP globally.

Once enabled, interfaces can participate in LLDP neighbor discovery.

---

# 8. Checking LLDP Neighbors

The primary command is:

```cisco
show lldp neighbors
```

This provides a neighbor summary.

Think of it as:

> **"Who is directly connected to me?"**

A typical conceptual output looks like:

```text
Device ID        Local Intf    Hold-time    Capability    Port ID
SW2              Gi0/1         120          B             Gi0/1
R1               Gi0/2         120          R             Gi0/0
AP1              Gi0/3         120          W             Gi0
```

The exact output depends on the Cisco platform and device.

---

# 9. Detailed LLDP Information

For more information:

```cisco
show lldp neighbors detail
```

Instead of simply asking:

> "Who is connected?"

you're asking:

> **"Tell me everything useful you know about that neighbor."**

Depending on the platform and neighbor, this may provide information such as:

* Device identity
* Local interface
* Remote interface
* Platform information
* Software information
* IP address
* Capabilities
* Other advertised information

---

# 10. The Two Most Important Commands

Memorize these:

```cisco
lldp run
```

→ Enable LLDP globally.

```cisco
show lldp neighbors
```

→ Show directly connected LLDP neighbors.

And for deeper information:

```cisco
show lldp neighbors detail
```

---

# 11. LLDP Is a Layer 2 Protocol

LLDP operates at **Layer 2 — the Data Link layer**.

This is important because neighbor discovery happens between **directly connected devices**.

You aren't using LLDP to discover an arbitrary router five hops away.

Conceptually:

```text
SW1 ─── SW2 ─── R1 ─── SW3
```

From SW1, LLDP can discover:

```text
SW2
```

It does **not** automatically discover:

```text
R1
SW3
```

because those are not directly connected to SW1.

### Remember

> **LLDP discovers neighbors, not the entire network.**

---

# 12. LLDP and Security

Neighbor discovery is extremely useful, but it also exposes information.

If a device advertises information about itself, a connected device may be able to learn things about it.

For example:

```text
Device
  ↓
LLDP advertisement
  ↓
Neighbor learns information
```

This can potentially reveal useful infrastructure information.

Therefore, LLDP should be treated as both:

* A **visibility/troubleshooting tool**
* Something that requires **appropriate security consideration**

---

# 13. One Major LLDP Advantage: Directional Control

One feature emphasized in the lesson is that LLDP can be controlled **directionally**.

You can control whether an interface:

* **Transmits** LLDP information
* **Receives** LLDP information
* Does **both**

Conceptually:

```text
             LLDP
              │
       ┌──────┴──────┐
       │             │
    Transmit       Receive
       │             │
       ▼             ▲
    Advertise      Learn
    information   information
```

This provides more granular control than simply thinking:

```text
LLDP = ON
LLDP = OFF
```

---

# 14. Why Directional Control Matters

Imagine an interface connected to an IP phone.

You might want your switch to **receive** LLDP information from the phone while controlling whether the switch advertises its own information back.

Conceptually:

```text
Switch ────────► Phone
       LLDP TX

Phone ─────────► Switch
       LLDP RX
```

Or you could configure the interface so that LLDP operates in both directions.

This gives administrators greater control over **visibility and information exposure**.

---

# 15. Real-World Troubleshooting Workflow

When you enter an unknown network, a useful discovery workflow is:

### Step 1 — Check whether CDP is running

```cisco
show cdp neighbors
```

### Step 2 — Check whether LLDP is running

```cisco
show lldp neighbors
```

### Step 3 — If LLDP is available, obtain details

```cisco
show lldp neighbors detail
```

### Step 4 — Build the topology

For example:

```text
                 R1
                 │
                 │
              Gi0/1
                 │
                 ▼
        ┌────────────────┐
        │      SW1       │
        └─┬──────┬───────┘
          │      │
       Gi0/2   Gi0/3
          │      │
          ▼      ▼
         SW2     AP1
```

### Step 5 — Verify physically

Discovery information is useful, but don't blindly trust it.

Verify:

* Cabling
* Interface status
* Device identity
* Port assignments
* Network documentation

---

# 16. What If Neither CDP Nor LLDP Is Running?

This is an important troubleshooting lesson.

If:

```text
CDP = OFF
LLDP = OFF
```

you lose the easy neighbor-discovery mechanism.

You may have to:

* Trace cables
* Inspect interfaces
* Check MAC address tables
* Check interface descriptions
* Identify devices manually
* Examine IP configurations
* Document the topology yourself

So the absence of discovery information is itself useful information.

It may indicate that the network:

* Is not configured for discovery
* Has intentionally disabled discovery
* Has poor documentation
* Requires a more careful investigation

---

# 17. LLDP in the Castle Rysen Environment

This lesson fits directly into the larger Castle Rysen network project.

The RFP requires a network spanning:

* Central Office
* Fallout Shelters
* District Shops

and requires network infrastructure containing routers, switches, wireless infrastructure, security, routing, and ongoing support. 

The RFP also explicitly calls for **network monitoring** and maintaining connectivity across the organization's locations. 

LLDP can therefore help during the operational phase by answering questions such as:

> Which switch is connected to this router?

> Which access point is connected to this switch port?

> Which uplink connects this device to the rest of the network?

> What device is physically attached to this interface?

This becomes especially valuable when the network grows.

---

# 18. LLDP vs MAC Address Table

Don't confuse these two.

### MAC address table

Answers:

> **"Which MAC address did I learn on which port?"**

```text
MAC Address        Port
AAAA.BBBB.CCCC     Gi0/1
DDDD.EEEE.FFFF     Gi0/2
```

### LLDP

Answers:

> **"What network device is connected to this port?"**

```text
Local Port    Neighbor
Gi0/1         SW2
Gi0/2         R1
Gi0/3         AP1
```

So:

```text
CAM/MAC Table
      ↓
MAC → Interface

LLDP
      ↓
Neighbor Device → Interface
```

They complement each other during troubleshooting.

---

# 19. LLDP vs CDP — Mental Model

Don't overcomplicate this.

Think:

```text
                 Neighbor Discovery
                        │
             ┌──────────┴──────────┐
             │                     │
            CDP                   LLDP
             │                     │
          Cisco                 Industry
         ecosystem               standard
             │                     │
       Cisco-focused          Multi-vendor
```

### One-line memory trick

> **CDP = Cisco Discovery**
>
> **LLDP = Everyone Discovery**

The lesson's phrase **"Everybody Else version of CDP"** is a useful way to remember the distinction.

---

# 20. Packet Tracer Warning

The lesson specifically warns that **Packet Tracer may behave inconsistently with LLDP**, particularly in VLAN-heavy topologies.

So if you see unexpected behavior in the simulator:

```text
Configuration looks correct
        ↓
LLDP output looks strange
        ↓
Don't immediately assume
your networking concept is wrong
```

The simulator may not perfectly reproduce real-device behavior.

The important thing is understanding the protocol and its operational purpose.

---

# 21. Exam / Interview Essentials

### Q: What is LLDP?

**A:** A standards-based Layer 2 neighbor discovery protocol used to discover directly connected devices and exchange information.

### Q: What is the biggest advantage of LLDP over CDP?

**A:** **Multi-vendor interoperability.**

### Q: Is LLDP Cisco proprietary?

**A:** **No.** LLDP is an industry-standard protocol.

### Q: What command enables LLDP globally on Cisco IOS?

```cisco
lldp run
```

### Q: How do you view LLDP neighbors?

```cisco
show lldp neighbors
```

### Q: How do you see detailed LLDP information?

```cisco
show lldp neighbors detail
```

### Q: What layer does LLDP operate at?

**Layer 2 — Data Link layer.**

### Q: Does LLDP discover devices several hops away?

**No.** It discovers **directly connected neighbors**.

### Q: Why is LLDP particularly useful in mixed-vendor networks?

Because it provides a **common standards-based neighbor discovery mechanism** across different vendors.

### Q: What special interface-level capability does the lesson highlight?

LLDP can be controlled **directionally**, allowing transmit and receive behavior to be managed independently.

---

# 22. Commands to Put in Your CCNA Command Arsenal

```cisco
! Enable LLDP globally
lldp run

! Display LLDP neighbors
show lldp neighbors

! Display detailed neighbor information
show lldp neighbors detail
```

For troubleshooting, remember the discovery pair:

```cisco
show cdp neighbors
show lldp neighbors
```

and their detailed versions:

```cisco
show cdp neighbors detail
show lldp neighbors detail
```

---

# 23. The Big Picture

You've now learned two complementary discovery protocols:

```text
                  Network Discovery
                         │
             ┌───────────┴───────────┐
             │                       │
            CDP                     LLDP
             │                       │
      Cisco proprietary       Industry standard
             │                       │
      Cisco environments       Multi-vendor
             │                       │
             └───────────┬───────────┘
                         │
                         ▼
               Discover neighbors
                         │
                         ▼
                Build network map
                         │
                         ▼
                 Troubleshoot faster
```

The key progression across the two lessons is:

> **CDP and LLDP don't route traffic. They help you understand the network.**

That's an important distinction.

You use routing protocols such as **OSPF** to determine paths through the network.

You use **STP** to prevent Layer 2 loops.

You use **LLDP/CDP** to understand **what is physically/logically connected to what**.

---

## 🧠 What you should be able to recall tomorrow without notes

If I ask you:

**"You walk into a completely undocumented, multi-vendor network. What protocol would you hope is enabled?"**

Your answer should immediately be:

> **LLDP — because it's a standards-based Layer 2 neighbor discovery protocol designed for multi-vendor environments.**

And if you're on Cisco IOS:

```cisco
lldp run
show lldp neighbors
show lldp neighbors detail
```

**That's the core of today's lesson.**
