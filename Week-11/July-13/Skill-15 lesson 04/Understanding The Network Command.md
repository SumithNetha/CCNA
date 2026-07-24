# Understanding the `network` Command (OSPF)

## Overview

The **`network` command** is one of the most important OSPF configuration commands.

Many beginners think it simply advertises a network, but it actually performs **two jobs simultaneously**.

> **The `network` command does NOT directly advertise the IP address you type. It first matches interfaces. Once an interface is matched, OSPF runs on that interface and advertises the connected network.**

---

# Syntax

```bash
router ospf <process-id>
network <network-address> <wildcard-mask> area <area-id>
```

Example:

```bash
router ospf 1
network 10.0.18.0 0.0.0.31 area 0
```

---

# Two Functions of the `network` Command

The `network` command performs two operations:

## 1. Identifies Which Interfaces Run OSPF

- Checks every interface on the router.
- Matches interfaces using the IP address and wildcard mask.
- Enables OSPF on matching interfaces.
- Starts sending OSPF Hello packets (unless configured as passive).

---

## 2. Advertises Connected Networks

After matching an interface, OSPF automatically advertises the **connected network** of that interface to OSPF neighbors.

---

# OSPF Process Flow

```
network Command
        │
        ▼
Find Matching Interface
        │
        ▼
Enable OSPF on Interface
        │
        ▼
Send Hello Packets
(Unless Passive)
        │
        ▼
Form Neighbor Relationship
        │
        ▼
Advertise Connected Network
```

---

# Example Topology

```
             PC
              │
        10.0.18.0/27
              │
         Ethernet0/0
        +-------------+
        | Cafe Router |
        +-------------+
         Ethernet0/1
              │
      192.168.1.0/24
              │
        Fallout Router
```

---

# Example 1 – Matching a Network

Router Interface:

```
Ethernet0/0

IP Address : 10.0.18.1
Subnet Mask: /27
```

Configuration:

```bash
router ospf 1
network 10.0.18.0 0.0.0.31 area 0
```

### OSPF Process

```
Check all interfaces

↓

10.0.18.1/27

↓

Matches

↓

Enable OSPF

↓

Advertise

10.0.18.0/27
```

---

# Example 2 – Matching a Single Interface

Configuration:

```bash
router ospf 1
network 10.0.18.1 0.0.0.0 area 0
```

### OSPF Process

```
Check all interfaces

↓

Find interface with IP

10.0.18.1

↓

Enable OSPF

↓

Advertise

10.0.18.0/27
```

---

# Important Observation

Both configurations produce the **same advertisement**.

### Method 1

```bash
network 10.0.18.0 0.0.0.31 area 0
```

↓

Matches interface

↓

Advertises

```
10.0.18.0/27
```

---

### Method 2

```bash
network 10.0.18.1 0.0.0.0 area 0
```

↓

Matches same interface

↓

Advertises

```
10.0.18.0/27
```

---

# Key Concept

The `network` command is **not telling OSPF which route to advertise.**

Instead, it tells OSPF:

> **"Find this interface."**

Once the interface is found:

- OSPF runs on that interface.
- OSPF advertises the interface's connected network.

---

# OSPF Hello Packets

When OSPF is enabled on an interface, it sends **Hello packets**.

Purpose:

- Discover neighboring routers
- Maintain neighbor relationships
- Detect failed neighbors

Example:

```
Cafe Router

Hello
   ↓

Fallout Router

Hello
```

After successful Hello exchange:

```
Neighbor Relationship Established
```

---

# OSPF Neighbor Relationship

Once neighbors are formed:

- Routing information is exchanged.
- Each router learns remote networks.
- Routing tables are updated.

---

# Passive Interface

Some interfaces do not connect to routers.

Example:

- PC LAN
- Server VLAN
- Printer Network
- User VLAN

Running OSPF Hellos here is unnecessary.

Cisco provides:

```bash
passive-interface <interface>
```

---

## What Passive Interface Does

✔ Advertises the connected network

✘ Stops sending Hello packets

✘ Prevents neighbor formation

---

## Why Use Passive Interface?

Benefits:

- Improves security
- Reduces unnecessary OSPF traffic
- Prevents rogue devices from becoming neighbors
- Best practice in production environments

---

# Wildcard Mask

OSPF uses **Wildcard Masks** instead of subnet masks.

Formula:

```
Wildcard Mask = 255 - Subnet Mask
```

---

## Common Wildcard Masks

| Subnet Mask | Prefix | Wildcard Mask |
|-------------|--------|---------------|
| 255.255.255.0 | /24 | 0.0.0.255 |
| 255.255.255.128 | /25 | 0.0.0.127 |
| 255.255.255.192 | /26 | 0.0.0.63 |
| 255.255.255.224 | /27 | 0.0.0.31 |
| 255.255.255.240 | /28 | 0.0.0.15 |
| 255.255.255.248 | /29 | 0.0.0.7 |
| 255.255.255.252 | /30 | 0.0.0.3 |

---

# Wildcard Calculation Example

Subnet Mask:

```
255.255.255.224
```

Calculation:

```
255-255 = 0

255-255 = 0

255-255 = 0

255-224 = 31
```

Wildcard Mask:

```
0.0.0.31
```

---

# Interface Matching Summary

```
network Command

        │

        ▼

Matches Interface

        │

        ▼

Runs OSPF

        │

        ▼

Sends Hello Packets
(Unless Passive)

        │

        ▼

Advertises Connected Network
```

---

# Key Takeaways

- The **`network` command performs two functions**:
  1. Matches interfaces that should participate in OSPF.
  2. Advertises the connected networks of those matched interfaces.
- OSPF **does not advertise the IP address specified in the `network` command**.
- The IP address and wildcard mask are used **only to find matching interfaces**.
- Once an interface is matched, OSPF reads the interface's IP address and subnet mask to determine the connected network to advertise.
- A wildcard mask of **`0.0.0.0`** matches a single, exact interface IP address.
- Both commands below advertise the same connected network if they match the same interface:

```bash
network 10.0.18.0 0.0.0.31 area 0
```

```bash
network 10.0.18.1 0.0.0.0 area 0
```

- **Hello packets** are used to discover and maintain OSPF neighbors.
- Use **`passive-interface`** on interfaces that do not connect to other routers. The network is still advertised, but Hello packets are not sent.
- **Wildcard Mask = 255 − Subnet Mask**.
- The easiest way to remember the command is:

> **The `network` command finds interfaces. The matched interface runs OSPF and advertises its connected network.**