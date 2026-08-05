# Castle Rysen – Configuring HSRPv2

## Overview

In the previous lesson, we learned **why First Hop Redundancy Protocols (FHRPs)** are needed. This lesson puts those concepts into practice by implementing **HSRP Version 2 (HSRPv2)** in the Castle Rysen Fallout Shelter network.

The objective is to eliminate the **single point of failure** at the default gateway. Even if one edge router fails, users should continue accessing internal resources and the Internet without changing any network settings.

This lesson also demonstrates that **HSRP is not a standalone technology**. It relies on an already functioning network infrastructure that includes:

- Router-on-a-Stick
- VLANs
- Trunk Links
- OSPF Routing
- NAT
- Internet Connectivity

Only after these services are working correctly can HSRP provide gateway redundancy.

---

# Learning Objectives

By the end of this lesson you should be able to:

- Understand why HSRP is required.
- Explain the default gateway problem.
- Differentiate physical router addresses from virtual gateway addresses.
- Configure HSRP Version 2.
- Configure Active and Standby routers.
- Understand HSRP priority and preemption.
- Explain how virtual MAC addresses work.
- Verify HSRP operation.
- Test failover and failback.
- Understand Interface Tracking.

---

# Network Before HSRP

Before HSRP was introduced, the Fallout Shelter already contained a fully functional enterprise network.

The infrastructure already included:

- Two edge routers
- Multiple VLANs
- Layer-2 switching
- EtherChannel
- Rapid PVST
- Router-on-a-Stick
- OSPF
- NAT Overload
- Internet connectivity

Although the network had **router redundancy**, it still had a **single default gateway**.

This meant users were still dependent on one router.

---

# The Default Gateway Problem

Every host is configured with a default gateway.

Example:

```
PC
IP Address: 10.0.16.11

Default Gateway:
10.0.16.1
```

Originally,

```
10.0.16.1
```

belonged directly to **Router 1**.

Everything worked normally while Router 1 remained operational.

However,

if Router 1 failed,

the PCs would continue sending traffic to:

```
10.0.16.1
```

Since no router would answer,

all traffic would simply be discarded.

The host has **no mechanism** to automatically discover another router.

Simply adding a second router does **not** solve this problem.

---

# Why Two Routers Are Not Enough

Many beginners assume redundancy looks like this:

```
Internet
     |
 Router 1

Internet
     |
 Router 2
```

Since two routers exist,

they assume the network is redundant.

Actually,

every client still points toward only one gateway.

If that gateway disappears,

the backup router is never used.

The hosts have no idea another router even exists.

This is exactly why **First Hop Redundancy Protocols** were created.

---

# What is First Hop Redundancy?

The **First Hop** is the very first Layer-3 device a host sends traffic to.

Usually,

this is the **default gateway**.

HSRP allows multiple routers to appear as **one virtual gateway**.

Instead of hosts communicating directly with Router 1 or Router 2,

they communicate with a **shared virtual IP address**.

Internally,

the routers decide which one forwards traffic.

From the user's perspective,

there is always one gateway.

---

# Preparing the Network Before HSRP

One of the most important lessons in this module is that **HSRP cannot fix an incomplete network.**

Before configuring HSRP,

the following services had to be operational.

## Internet Connectivity

Each router required its own Internet connection.

Without Internet connectivity,

HSRP could successfully move the gateway,

but Internet traffic would still fail.

---

## Router-on-a-Stick

Every VLAN required its own router subinterface.

Each subinterface needed:

- 802.1Q encapsulation
- IP address
- NAT configuration

Without Router-on-a-Stick,

the routers would never receive VLAN traffic.

---

## Trunk Links

The switch uplinks needed to operate as trunks.

These trunk ports carried VLAN traffic between switches and routers.

Without trunking,

Router-on-a-Stick would never function.

---

## OSPF

Both routers needed to exchange routing information.

Without OSPF,

the backup router would not know how to reach remote networks after failover.

HSRP only changes the default gateway.

It does **not** exchange routes.

---

## NAT

Each router required its own NAT configuration.

Without NAT,

failover would successfully change the gateway,

but Internet communication would fail because private addresses could not be translated.

---

# Simulating Two Internet Connections

Packet Tracer has limitations when simulating multiple Internet Service Providers.

To overcome this,

the lab uses a switch to emulate two separate ISP handoffs.

Although acceptable for a lab,

this is **not** how redundant Internet should be designed in production.

---

# Real-World Design Tip

When purchasing redundant Internet circuits,

do not simply order two connections.

Always verify:

- Different Internet Service Providers whenever possible.
- Separate physical cable paths into the building.

If both Internet circuits share the same underground conduit,

one cable cut can disconnect both connections.

True redundancy eliminates **single points of failure**.

---

# Separating the Gateway from the Router

Originally,

Router 1 owned the gateway addresses.

Example:

```
Router 1

10.0.16.1
```

With HSRP,

the addressing changes.

```
Virtual Gateway

10.0.16.1

↓

Router 1
10.0.16.2

↓

Router 2
10.0.16.3
```

Now,

the gateway belongs to **HSRP**,

not to any physical router.

This separates the identity of the gateway from the hardware providing it.

---

# HSRPv2

HSRP Version 2 is the modern implementation of Cisco's Hot Standby Router Protocol.

It provides improvements over Version 1 including:

- Better scalability
- Larger HSRP group numbers
- IPv6 support
- Modern enterprise compatibility

---

# HSRP Components

## Active Router

The Active Router forwards all traffic.

Only one Active Router exists per HSRP group.

---

## Standby Router

The Standby Router continuously monitors the Active Router.

It remains ready to immediately assume forwarding responsibilities if the Active Router fails.

---

## Virtual IP Address

Hosts never use the physical router addresses.

Instead,

they use a shared Virtual IP.

Example:

```
Virtual Gateway

10.0.16.1
```

Regardless of which router is Active,

clients continue using this address.

---

## Virtual MAC Address

Hosts also learn a **Virtual MAC Address** rather than the physical MAC of either router.

This allows the gateway to remain consistent during failover.

The host never knows which router is actually forwarding traffic.

---

# HSRP Groups

Each HSRP-enabled interface belongs to a group.

Both participating routers must use the same group number.

A simple design strategy is:

```
VLAN 10

↓

HSRP Group 10
```

```
VLAN 20

↓

HSRP Group 20
```

This makes configurations much easier to understand and troubleshoot.

---

# Priority

Priority determines which router becomes Active.

Default Priority:

```
100
```

Higher values are preferred.

Example:

```
Router 1

Priority 105
```

```
Router 2

Priority 100
```

Router 1 becomes the Active Router.

---

# Preempt

Preempt allows the preferred router to reclaim the Active role after recovering from a failure.

Without Preempt:

```
Router 1 fails

↓

Router 2 becomes Active

↓

Router 1 returns

↓

Router 2 remains Active
```

With Preempt:

```
Router 1 fails

↓

Router 2 becomes Active

↓

Router 1 returns

↓

Router 1 automatically becomes Active again
```

This keeps the network operating according to its intended design.

---

# HSRP Election Process

The Active Router is selected using the following order:

1. Highest Priority
2. Highest Interface IP Address (if priorities are equal)

---

# Verifying HSRP

After configuration,

verify HSRP using:

```
show standby brief
```

The output displays:

- Active Router
- Standby Router
- Priority
- Virtual IP
- Group Number
- Preempt status

Always verify HSRP before testing failover.

---

# Testing Failover

Configuration alone does not guarantee redundancy.

The network must be tested.

Typical validation includes:

- Continuous ping
- Verify Active Router
- Verify NAT translations
- Simulate failure
- Observe failover
- Verify connectivity

During failover,

a small number of packets may be lost while HSRP transitions.

This is expected.

---

# Failback

When the preferred router returns,

Preempt allows it to resume the Active role.

The network automatically returns to its original design.

This process is called **Failback**.

---

# Interface Tracking

HSRP normally determines Active status using Priority.

However,

consider this situation:

- Router is still powered on.
- VLAN interfaces remain operational.
- Internet-facing interface fails.

Without Interface Tracking,

the router still believes it should remain Active.

Hosts continue forwarding traffic to a router that no longer has Internet access.

Interface Tracking solves this problem by automatically reducing the router's HSRP priority whenever a monitored interface fails.

This allows the healthier router to become Active.

---

# Why HSRP Matters

HSRP is not designed to protect routers.

It is designed to protect **users**.

The users should never know:

- which router is Active
- when a router fails
- when failover occurs

Their only expectation is:

- Applications continue working.
- Internet remains available.
- Business operations continue uninterrupted.

---

# Real-World Best Practices

- Never assume two routers provide redundancy.
- Configure routing before HSRP.
- Configure NAT before HSRP.
- Verify trunk links.
- Verify OSPF neighbors.
- Test every failover scenario.
- Always test failback.
- Use meaningful HSRP group numbers.
- Configure Preempt when a preferred Active Router exists.
- Consider Interface Tracking for WAN interfaces.

---

# Key Takeaways

- HSRP provides **default gateway redundancy**.
- Hosts always communicate with a **Virtual IP**, not a physical router.
- HSRP does not replace routing, NAT, VLANs, or switching.
- Router 1 and Router 2 appear as one logical gateway.
- Only one router forwards traffic at a time.
- The Active Router is selected using Priority.
- Preempt allows the preferred router to reclaim the Active role after recovery.
- Virtual MAC addresses hide physical router changes from hosts.
- Redundancy is only complete after successful failover and failback testing.
- A redundancy design that has never been tested cannot be considered reliable.