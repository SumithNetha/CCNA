# Skill 13 Lesson 01 – Why We Need STP

## Why Redundancy is Necessary

In an enterprise network, redundancy provides **high availability** by ensuring communication continues even if a link fails.

### Single Link

```text
SW1 -------- SW2
```

- Only one communication path exists.
- If the cable fails, connectivity is lost.

### Adding Redundancy

```text
      Link 1
SW1 ========== SW2
      Link 2
```

Advantages:
- Backup path available.
- Increased fault tolerance.
- Higher network availability.

However, multiple active Layer 2 paths introduce a serious problem.

---

# Layer 2 Switching Loops

When redundant links exist between switches, Ethernet frames can continuously circulate between switches, creating a **Layer 2 loop**.

Unlike routers, switches do not automatically prevent loops.

---

# Broadcast Storm

A broadcast frame (such as a DHCP Discover message) is flooded by every switch out all ports except the incoming port.

Example:

```text
PC
 |
SW1 ===== SW2
```

Flow:

1. PC sends a broadcast.
2. SW1 floods the broadcast.
3. SW2 receives and floods it again.
4. The frame returns to SW1.
5. The process repeats indefinitely.

Result:

```text
SW1 → SW2 → SW1 → SW2 → ...
```

This continuous circulation is called a **Broadcast Storm**.

Effects:

- Consumes all available bandwidth.
- High switch CPU utilization.
- Continuous frame replication.
- Network becomes extremely slow or unusable.
- Devices may lose connectivity.

---

# Why Routers Don't Have This Problem

Routers operate at **Layer 3** and process IP packets.

Every IP packet contains a **TTL (Time To Live)** value.

Example:

```text
TTL = 255
```

Each router decreases the TTL by one.

```text
255 → 254 → 253 → ... → 0
```

When TTL reaches zero:

- Packet is discarded.
- Routing loops are automatically terminated.

---

# Why Switches Cannot Use TTL

Switches operate at **Layer 2**.

They forward **Ethernet Frames**, not IP packets.

Switches examine:

- Source MAC Address
- Destination MAC Address

They do **not** examine:

- IP Address
- TTL

Therefore:

- Ethernet frames have no TTL countdown.
- Layer 2 loops continue indefinitely until the loop is removed.

---

# Spanning Tree Protocol (STP)

**STP (Spanning Tree Protocol)** is a Layer 2 loop prevention protocol.

Its purpose is to:

- Prevent switching loops.
- Maintain redundant links.
- Automatically recover from link failures.

---

# How STP Works

Physical topology:

```text
      Link 1
SW1 ========== SW2
      Link 2
```

Without STP:

```text
Both links Forwarding
↓

Layer 2 Loop
↓

Broadcast Storm
```

With STP:

```text
Link 1 → Forwarding

Link 2 → Blocking
```

Result:

```text
SW1 -------- SW2
 Active

SW1 ---X---- SW2
 Blocked
```

The blocked cable remains physically connected but does not forward normal traffic.

---

# Backup Link Operation

Normal state:

```text
Primary Link → Forwarding

Backup Link → Blocking
```

If the primary link fails:

```text
Primary Link → Down

Backup Link → Changes to Forwarding
```

STP recalculates the topology and activates the backup path.

This provides:

- Loop prevention
- Automatic failover
- Network redundancy

---

# Physical vs Logical Topology

## Physical Topology

Shows every cable physically installed.

```text
      SW1
     /   \
    /     \
 SW2 ----- SW3
```

---

## Logical Topology (After STP)

STP blocks one redundant path.

```text
      SW1
     /   \
    /     \
 SW2  X  SW3
```

The cable still exists physically, but logically it is prevented from forwarding traffic.

---

# Why Learning STP Is Important

Knowing only that STP blocks redundant links is insufficient.

STP must determine:

- Which switch becomes the Root Bridge.
- Which ports forward traffic.
- Which ports are blocked.
- Which path should be preferred.

Improper STP design may block a high-speed link instead of a slower backup link, reducing overall network performance.

Understanding STP allows administrators to control path selection and optimize Layer 2 traffic flow.

---

# Real-World Scenario

A common production issue occurs when someone accidentally connects an additional cable between two switches.

Without proper STP:

- Broadcast storm occurs.
- MAC address table becomes unstable.
- High CPU utilization.
- Users lose network connectivity.
- Entire VLAN may become unavailable.

Always verify STP operation before introducing redundant switch connections.

---

# Key Terms

- **Redundancy** – Multiple available network paths for fault tolerance.
- **Layer 2 Loop** – Continuous circulation of Ethernet frames between switches.
- **Broadcast Storm** – Endless broadcast traffic caused by a Layer 2 loop.
- **TTL (Time To Live)** – Layer 3 mechanism that prevents routing loops.
- **STP (Spanning Tree Protocol)** – Layer 2 protocol that prevents switching loops.
- **Forwarding Port** – Actively forwards traffic.
- **Blocking Port** – Prevents loops while remaining available as a backup path.

---

# Key Takeaways

- Redundant links improve network availability but create Layer 2 loops if unmanaged.
- Broadcast frames can circulate forever because Ethernet frames have no TTL field.
- Routers prevent loops using the IP TTL mechanism; switches cannot.
- STP creates a loop-free logical topology while preserving physical redundancy.
- Blocked links remain available and automatically become active if the forwarding path fails.
- STP is a fundamental Layer 2 technology used in enterprise networks to ensure redundancy without causing broadcast storms.