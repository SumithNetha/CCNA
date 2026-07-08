# Skill 12 Lesson 07 – Building the Fallout Shelter Network

## Why the Cafe Network Was No Longer Enough

The original Castle Rysen Cafe network was designed as a **SOHO (Small Office/Home Office)** network.

It was excellent for learning:

- Basic switching
- VLANs
- Router configuration
- NAT
- DHCP
- Static Routing

However, it became too small to demonstrate real enterprise networking concepts.

### Limitations of the Cafe Network

- Very few switches
- Single router
- No redundant links
- No backup devices
- No Layer 2 loops
- No gateway redundancy
- No enterprise design

Since there is only one path between devices, advanced technologies such as **STP, HSRP, EtherChannel, and Redundancy** cannot be demonstrated effectively.

---

# Why Introduce the Fallout Shelter Network?

The Fallout Shelter is much larger than a single coffee shop.

Instead of supporting one location, it supports:

- Approximately **50 users**
- Multiple downstream district coffee shops
- Regional network services
- Business-critical operations

If a district coffee shop fails:

- Only one shop loses connectivity.

If the Fallout Shelter fails:

- Multiple coffee shops lose connectivity.
- Regional services stop.
- Business operations are interrupted.

This changes the design priorities from **simple connectivity** to **high availability and redundancy**.

---

# Network Design Evolution

## Small Office Design

```text
Internet
    |
 Router
    |
 Switch
    |
 End Devices
```

Characteristics:

- Simple
- Low cost
- Minimal redundancy
- Suitable for small offices

---

## Enterprise Design

```text
                 Internet
                     |
              Redundant Routers
                 /         \
        Distribution   Distribution
            |      \   /      |
        Access    Access    Access
             \      |      /
             End Devices
```

Characteristics:

- Multiple switches
- Multiple routers
- Redundant paths
- Fault tolerance
- High availability

---

# Enterprise Hierarchical Network Design

Enterprise networks are divided into logical layers.

Benefits:

- Scalability
- Easier troubleshooting
- Better performance
- Simplified management
- High availability

---

# Three-Tier Architecture

```text
        Core
          |
-------------------
|                 |
Distribution   Distribution
|                 |
Access         Access
|                 |
End Devices   End Devices
```

Each layer has a different responsibility.

---

# Access Layer

The Access Layer is where end devices connect.

Examples:

- PCs
- Laptops
- Printers
- Wireless Access Points
- IP Phones
- Cameras
- Servers

Responsibilities:

- Connect end devices
- Assign VLANs
- Apply Port Security
- Forward traffic toward Distribution Layer

Access switches normally do **not** perform routing.

---

# Distribution Layer

The Distribution Layer connects multiple Access Switches.

Responsibilities:

- Aggregate Access Switches
- Inter-VLAN Routing
- Policy Enforcement
- ACLs
- Route Summarization
- Redundant Connections
- Gateway Services

Think of it as the traffic manager of the LAN.

---

# Core Layer

The Core Layer is the network backbone.

Responsibilities:

- High-speed forwarding
- Connect multiple buildings
- Transport traffic quickly
- Connect campuses

The Core Layer should focus on speed rather than filtering or security policies.

---

# Why No Core Layer?

The Fallout Shelter only supports around **50 users**.

Adding a dedicated Core Layer would:

- Increase hardware cost
- Increase complexity
- Require more switches
- Provide little additional benefit

Instead, Cisco recommends using a **Collapsed Core Design**.

---

# Collapsed Core Design

Instead of:

```text
Core
 |
Distribution
 |
Access
```

Use:

```text
Collapsed Core
       |
------------------
|                |
Access        Access
```

The Distribution Switches perform both:

- Core Layer functions
- Distribution Layer functions

Advantages:

- Lower cost
- Easier management
- High scalability
- Enterprise reliability

---

# Fallout Shelter Physical Topology

```text
                     ISP
                    /   \
                  R1     R2
                  |       |
             DSW1--------DSW2
              |\  \    /  /|
              | \  \  /  / |
              |  \  \/  /  |
             AS1 AS2 AS3 AS4
```

Where:

- R1, R2 = Routers
- DSW = Distribution Switches
- AS = Access Switches

---

# ISP Connection

The organization connects to the Internet through an ISP.

Instead of using one router:

```text
ISP
 |
Router
```

the design uses:

```text
ISP
 / \
R1 R2
```

Benefits:

- Internet redundancy
- Router failover
- High availability

---

# Why Two Routers?

If Router 1 fails:

```text
ISP
 |
 R1 (Failed)

 R2 (Operational)
```

Router 2 continues forwarding traffic.

Later, these routers will use **HSRP (Hot Standby Router Protocol)**.

---

# HSRP Preview

Normally:

```text
PC

Default Gateway = Router1
```

If Router1 fails:

- Gateway disappears
- Users lose connectivity

HSRP allows:

```text
          Virtual Gateway
          /            \
      Router1      Router2
```

If Router1 fails:

- Router2 immediately becomes the active gateway.
- Users usually do not notice the failure.

---

# Distribution Switches

The Fallout Shelter contains two Distribution Switches.

Responsibilities:

- Aggregate Access Switches
- Provide redundancy
- Perform Inter-VLAN Routing
- Apply network policies
- Connect to routers

They also act as the **Collapsed Core**.

---

# Distribution Switch Interconnection

```text
DSW1 -------- DSW2
```

Purpose:

- Exchange Layer 2 traffic
- Exchange VLAN information
- Provide redundancy
- Future EtherChannel connection

---

# Access Switches

The Access Layer contains four switches.

Responsibilities:

- Connect user devices
- Assign VLANs
- Forward traffic upward
- Connect printers, APs, cameras, PCs

---

# Redundant Uplinks

Each Access Switch connects to **both Distribution Switches**.

Example:

```text
      DSW1
      /  \
    AS1   \
      \    \
       \   DSW2
```

Instead of:

```text
DSW

|

AS
```

Benefits:

- No single point of failure
- Automatic alternate path
- Higher availability

---

# Why So Many Cables?

The additional cables provide:

- Backup paths
- Redundancy
- Failure protection

They are **not** added to increase bandwidth.

---

# Addressing Plan

The Fallout Shelter subnet:

```text
10.0.16.0/23
```

Subnet Mask:

```text
255.255.254.0
```

Address Range:

```text
10.0.16.0

↓

10.0.17.255
```

Hosts:

- Total Addresses = 512
- Usable Hosts = 510

This subnet will later be divided into multiple VLANs.

---

# Enterprise Address Allocation

## Central Office

```text
10.0.0.0/20
```

Range:

```text
10.0.0.0

↓

10.0.15.255
```

Supports approximately 200 users.

---

## Fallout Shelter

```text
10.0.16.0/23
```

Supports approximately 50 users.

---

## District Shop

```text
10.0.18.0/26
```

Range:

```text
10.0.18.0

↓

10.0.18.63
```

Supports approximately 15 users.

---

# IP Addressing Convention

Infrastructure devices use static addresses.

Example:

| Address | Device |
|----------|--------|
| .1 | Default Gateway |
| .2 | Distribution Switch 1 |
| .3 | Distribution Switch 2 |
| .4 | Server |
| .5 | Camera Server |
| .6 | NAS |

DHCP clients receive addresses dynamically.

Example:

```
Infrastructure

1–10

DHCP Clients

10–50
```

---

# Base Configuration

Before beginning the lesson, all devices receive a standard base configuration.

Typical configuration includes:

- Hostname
- Console Password
- Enable Secret
- Banner
- VTY Password
- SSH Preparation
- Basic Security

This avoids repeating identical configuration during every lesson.

---

# Understanding Red Links

Packet Tracer displays red links because router interfaces are disabled by default.

Example:

```text
Router
  X
Switch
```

Reason:

- Interface is administratively down.

Enable using:

```cisco
interface GigabitEthernet0/0

no shutdown
```

After enabling:

- Link becomes green.

---

# Understanding Orange Links

Orange links indicate **Spanning Tree Protocol (STP)** has blocked the port.

Example:

```text
Switch

||

Switch
```

One link becomes blocked.

---

# Why Does STP Block Links?

Without STP:

```text
Switch1
 |     |
 |     |
Switch2
```

Frames continuously circulate.

This causes:

- Broadcast Storms
- Duplicate Frames
- MAC Table Instability
- Complete Network Failure

STP blocks one redundant path:

```text
Switch1
 |     X
 |
Switch2
```

The blocked link becomes a standby path.

---

# Do More Cables Mean More Bandwidth?

No.

Many beginners believe:

```
More cables = More speed
```

Not true.

With STP:

- One link is active.
- One link is blocked.

The blocked cable provides redundancy only.

Bandwidth remains unchanged.

---

# Future Technologies

## EtherChannel

Instead of blocking redundant links, EtherChannel combines them.

Example:

```text
2 × 1 Gbps Links

↓

2 Gbps Logical Link
```

All links actively forward traffic.

---

## STP Optimization

Later lessons will cover:

- Root Bridge Selection
- Primary Paths
- Backup Paths

This improves traffic flow.

---

## VLAN Design

The /23 network will later be divided into VLANs such as:

- Management VLAN
- Admin VLAN
- User VLAN
- Guest VLAN
- Camera VLAN
- Voice VLAN

---

# Why Build an Imperfect Network First?

The instructor intentionally builds an imperfect network.

Reason:

Real networking follows this process:

```text
Build

↓

Observe

↓

Identify Problems

↓

Optimize

↓

Deploy
```

If students only see perfect networks:

- They never understand why redundancy matters.
- They never appreciate STP.
- They never understand HSRP.
- They never see the value of EtherChannel.

The Fallout Shelter network provides a realistic environment where these technologies become necessary.

---

# Real-World Tip

When building practice labs:

Do **not** always build the smallest possible network.

Small labs often hide why enterprise technologies exist.

A slightly larger network naturally introduces:

- Redundancy
- Loop Prevention
- Gateway Failover
- Traffic Segmentation
- High Availability
- Enterprise Design Principles

These concepts become much easier to understand when failures have visible consequences.

---

# Key Takeaways

- The Cafe network is suitable for learning basic networking concepts but is too small for enterprise design.
- The Fallout Shelter introduces a realistic enterprise topology supporting approximately 50 users and multiple downstream coffee shops.
- Enterprise networks use a hierarchical architecture consisting of Access, Distribution, and Core layers.
- The Fallout Shelter uses a **Collapsed Core Design**, combining the Core and Distribution layers.
- Access switches connect end devices, while Distribution switches aggregate traffic and provide routing and policy enforcement.
- Two routers provide Internet redundancy and will later use **HSRP** for gateway failover.
- Each Access Switch connects to both Distribution Switches, eliminating single points of failure.
- The Fallout Shelter uses the subnet **10.0.16.0/23**, which will later be divided into multiple VLANs.
- Router interfaces appear red in Packet Tracer because they are administratively down until the **no shutdown** command is issued.
- Orange links indicate STP is blocking redundant paths to prevent Layer 2 switching loops.
- Redundant links improve network availability but do not increase bandwidth unless technologies such as **EtherChannel** are implemented.
- The larger enterprise topology prepares the network for future topics including VLAN implementation, STP optimization, HSRP, EtherChannel, OSPF, Layer 2 security, and enterprise network services
