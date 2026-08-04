# Skill 16 - Lesson 02
# Flavors of First Hop Redundancy Protocols (FHRPs)

---

# Lesson Objective

In the previous lesson, we learned **why First Hop Redundancy Protocols (FHRPs) exist**.

They solve the problem of **default gateway failure** by allowing multiple routers to share a **Virtual IP (VIP)** and a **Virtual MAC (VMAC)**, ensuring hosts maintain connectivity even if the primary gateway fails.

In this lesson, we'll compare the three major FHRPs:

- HSRP (Hot Standby Redundancy Protocol)
- VRRP (Virtual Router Redundancy Protocol)
- GLBP (Gateway Load Balancing Protocol)

Although all three provide gateway redundancy, they differ in:

- Vendor support
- Router roles
- Preemption behavior
- Load balancing capabilities
- Protocol implementation

---

# Why Are There Multiple FHRPs?

Every FHRP solves the same core problem.

Without FHRP:

```text
                 Internet
                     |
                 Router1
                     |
                  Switch
                     |
                   Clients
```

Clients use:

```text
Default Gateway

192.168.10.1
```

If Router1 fails:

- Internet connectivity is lost.
- Remote networks become unreachable.
- The host continues sending traffic to the failed gateway.

---

With FHRP:

```text
                 Internet
               -------------
               |           |
           Router1      Router2
               |           |
               +-----SW----+
                     |
                 Clients

        Virtual Gateway
        192.168.10.1
```

If Router1 fails:

- Router2 automatically assumes ownership of the Virtual Gateway.
- Hosts continue using the same Default Gateway.
- No reconfiguration is required.
- Connectivity continues with little or no interruption.

---

# The Three Major FHRPs

Cisco devices support three major FHRPs.

| Protocol | Full Form |
|----------|-----------|
| HSRP | Hot Standby Redundancy Protocol |
| VRRP | Virtual Router Redundancy Protocol |
| GLBP | Gateway Load Balancing Protocol |

Although they solve the same problem, each protocol behaves differently.

---

# 1. HSRP (Hot Standby Redundancy Protocol)

HSRP is Cisco's proprietary First Hop Redundancy Protocol.

It was Cisco's first solution for gateway redundancy and remains one of the most common protocols in Cisco-only environments.

---

## Characteristics

- Cisco Proprietary
- One Active Router
- One Standby Router
- Active router forwards all traffic
- Standby router waits for failure
- Provides gateway redundancy only
- No automatic load balancing

---

## HSRP Operation

```text
          Virtual Gateway
           192.168.10.1

          ----------------
          |              |
      Router1        Router2
       Active         Standby
```

Normal Operation:

- Router1 owns the Virtual IP.
- Router1 owns the Virtual MAC.
- Router1 forwards all packets.
- Router2 continuously monitors Router1.

---

If Router1 fails:

```text
Router2

↓

Becomes Active

↓

Takes ownership of

Virtual IP

Virtual MAC
```

The host never changes its gateway configuration.

---

# HSRP Terminology

| Term | Description |
|------|-------------|
| Active Router | Router currently forwarding traffic |
| Standby Router | Backup router waiting to take over |
| Virtual IP | Shared Default Gateway IP |
| Virtual MAC | Shared MAC Address |

Remember:

HSRP always uses:

```
Active

↓

Standby
```

---

# HSRP Versions

## HSRP Version 1

Features:

- IPv4 support
- Limited HSRP groups

Used in older networks.

---

## HSRP Version 2

Enhancements:

- IPv6 support
- Larger number of HSRP groups
- Improved scalability

Most modern Cisco networks use HSRP Version 2.

---

# 2. VRRP (Virtual Router Redundancy Protocol)

VRRP is the industry-standard First Hop Redundancy Protocol.

Unlike HSRP, VRRP is an **Open Standard**, allowing routers from different vendors to participate in the same redundancy group.

Supported by vendors such as:

- Cisco
- Juniper
- Arista
- Huawei
- Nokia
- HP

---

## Characteristics

- Open Standard
- Multi-vendor support
- Gateway redundancy
- One router actively forwards traffic
- Very similar to HSRP

---

# VRRP Terminology

Unlike HSRP, VRRP uses different names.

| HSRP | VRRP |
|------|-------|
| Active | Master |
| Standby | Backup |

Example:

```text
        Virtual Gateway

       ----------------
       |              |
   Router1        Router2
    Master        Backup
```

Functionally,

Master = Active

Backup = Standby

---

# HSRP vs VRRP

Both protocols provide:

- Gateway redundancy
- Virtual IP
- Virtual MAC
- Automatic failover
- Hello messages

The major differences involve:

- Vendor support
- Default behavior
- Terminology

---

# Preemption

One of the most important differences between HSRP and VRRP is **Preemption**.

---

## What is Preemption?

Assume:

```text
Router1 Priority = 120

Router2 Priority = 100
```

Router1 is preferred.

Router1 fails.

Router2 becomes Active.

Later...

Router1 returns.

Question:

Should Router1 immediately become the Active router again?

This behavior is called:

> **Preemption**

---

# HSRP Default

In HSRP:

```
Preemption

Disabled
```

by default.

When Router1 returns:

```
Router2

continues

being Active.
```

Router1 waits unless:

```
standby preempt
```

is manually configured.

---

# VRRP Default

In VRRP:

```
Preemption

Enabled
```

by default.

When Router1 returns:

```
Router1

↓

Automatically

Becomes Master Again.
```

---

# Why Preemption Matters

Imagine Router1 failed because of:

- Faulty RAM
- Overheating
- Power problems
- IOS bug

It comes online again.

Immediately becomes Active.

Then crashes again.

Traffic now moves:

```
Router2

↓

Router1

↓

Router2

↓

Router1
```

This repeated failover causes:

- Packet loss
- Session interruptions
- Voice call drops
- User complaints

In production, many network engineers prefer verifying the router's stability before allowing it to reclaim the Active role.

---

# VRRP IP Address Advantage

One interesting VRRP feature:

The Virtual IP can also be configured as one router's physical IP.

Example:

```
Router1

192.168.10.1
```

Router1 can own:

- Physical IP
- Virtual Gateway IP

This saves one IPv4 address.

HSRP typically requires:

```
Router1

192.168.10.2

Router2

192.168.10.3

Virtual IP

192.168.10.1
```

Although small, this advantage can be useful in networks with limited IPv4 space.

---

# Virtual MAC Recognition

Every FHRP creates a Virtual MAC Address.

Network engineers often recognize which protocol is running by examining:

- ARP Tables
- MAC Address Tables
- Packet Captures
- Switch CAM Tables

Example:

```
show mac address-table

↓

Virtual MAC Found

↓

Immediately Identify

HSRP

or

VRRP

or

GLBP
```

Recognizing these MAC address patterns is valuable during troubleshooting and is commonly tested in certification exams.

---

# 3. GLBP (Gateway Load Balancing Protocol)

GLBP is Cisco's most advanced FHRP.

Unlike HSRP and VRRP,

GLBP provides:

- Gateway Redundancy
- Automatic Load Balancing

This allows multiple routers to actively forward traffic.

---

# Why Was GLBP Created?

Consider HSRP.

```text
Router1

Forwarding Traffic
```

Router2

```
Idle
```

The backup router remains unused until failure.

This wastes available bandwidth and router resources.

GLBP solves this inefficiency.

---

# GLBP Operation

```text
          Virtual Gateway
           192.168.10.1

         -------------------
         |                 |
      RouterA          RouterB
```

Instead of one forwarding router,

Both routers actively forward traffic.

Different hosts use different routers.

---

# GLBP Roles

GLBP introduces two important roles.

---

## Active Virtual Gateway (AVG)

Responsibilities:

- Manages the GLBP group.
- Answers ARP requests.
- Assigns Virtual MAC addresses.

Think of the AVG as the manager.

---

## Active Virtual Forwarder (AVF)

Responsibilities:

- Forwards packets.
- Owns Virtual MAC addresses.
- Multiple AVFs may exist.

Think of the AVFs as the workers forwarding traffic.

---

# How GLBP Performs Load Balancing

Suppose three hosts join the network.

PC1 sends ARP.

AVG replies:

```
Use Virtual MAC A
```

PC2 sends ARP.

AVG replies:

```
Use Virtual MAC B
```

PC3 sends ARP.

AVG replies:

```
Use Virtual MAC A
```

Traffic becomes:

```text
PC1

↓

RouterA
```

```text
PC2

↓

RouterB
```

```text
PC3

↓

RouterA
```

Every PC still uses:

```
192.168.10.1
```

Only the Virtual MAC differs.

This distributes traffic across multiple routers.

---

# Can HSRP or VRRP Load Balance?

Not automatically.

However,

Network administrators often perform manual load balancing.

Example:

```text
VLAN10

Router1 Active

Router2 Standby
```

```text
VLAN20

Router2 Active

Router1 Standby
```

Now:

- VLAN10 traffic primarily uses Router1.
- VLAN20 traffic primarily uses Router2.

This is called **manual load balancing**.

Unlike GLBP, traffic is balanced per VLAN rather than automatically per host.

---

# Multiple VLAN Considerations

Every VLAN has:

- Different Network
- Different Default Gateway

Example:

| VLAN | Gateway |
|------|----------|
| VLAN10 | 192.168.10.1 |
| VLAN20 | 192.168.20.1 |
| VLAN30 | 192.168.30.1 |
| VLAN40 | 192.168.40.1 |

Each VLAN requires:

- Separate HSRP Group
- Separate VRRP Group
- Separate GLBP Group

Large enterprise networks may have dozens or even hundreds of FHRP groups.

---

# Real-World Design Recommendations

### Cisco-Only Networks

Preferred Choice:

```
HSRP
```

Reason:

- Native Cisco support
- Easy integration
- Widely used in Cisco environments

---

### Multi-Vendor Networks

Preferred Choice:

```
VRRP
```

Reason:

- Industry Standard
- Vendor Independent
- Broad compatibility

---

### Networks Requiring Gateway Load Balancing

Preferred Choice:

```
GLBP
```

Reason:

- Multiple routers actively forward traffic.
- Better utilization of available bandwidth.
- Automatic host distribution.

---

# Practical Perspective vs CCNA Perspective

## In Real Networks

VRRP is very common because:

- Standards-based
- Vendor-neutral
- Similar functionality to HSRP

Many enterprise environments with mixed vendors deploy VRRP.

---

## For the CCNA Exam

Cisco emphasizes HSRP because:

- It is Cisco's proprietary protocol.
- Cisco IOS labs commonly use HSRP.
- Configuration examples typically demonstrate HSRP.

Understanding HSRP thoroughly makes learning VRRP much easier because the operational concepts are nearly identical.

---

# Comparison Table

| Feature | HSRP | VRRP | GLBP |
|----------|-------|-------|-------|
| Full Form | Hot Standby Redundancy Protocol | Virtual Router Redundancy Protocol | Gateway Load Balancing Protocol |
| Vendor | Cisco Proprietary | Open Standard | Cisco Proprietary |
| Router Roles | Active / Standby | Master / Backup | AVG / AVF |
| Gateway Redundancy | Yes | Yes | Yes |
| Automatic Load Balancing | No | No | Yes |
| Active Forwarding Routers | One | One | Multiple |
| Preemption Default | Disabled | Enabled | Enabled (role-based) |
| Multi-Vendor Support | No | Yes | No |
| Best Use Case | Cisco-only environments | Multi-vendor environments | Cisco networks needing redundancy + load balancing |

---

# Key Takeaways

- All FHRPs solve the same problem: **providing a resilient default gateway for hosts.**
- **HSRP** is Cisco's proprietary solution and uses **Active/Standby** terminology.
- **VRRP** is an open standard that uses **Master/Backup** terminology and supports multi-vendor environments.
- **Preemption** controls whether the preferred router automatically regains control after recovering from a failure.
- **HSRP disables preemption by default**, whereas **VRRP enables it by default**.
- **GLBP** is unique because it provides **both gateway redundancy and automatic load balancing**.
- HSRP and VRRP can achieve limited load balancing by making different routers active for different VLANs, but this requires manual configuration.
- Every VLAN has its own default gateway, so **each VLAN requires its own FHRP configuration**.
- For Cisco certification studies, focus on **HSRP**, but understand the operational differences between HSRP, VRRP, and GLBP for both exams and real-world deployments.