# Skill 13 Lesson 02 – Key STP Concepts, Simplified

## Overview

Spanning Tree Protocol (STP) prevents Layer 2 loops while allowing redundant links to remain physically connected.

Humans can examine a network diagram and identify redundant paths, but switches need a consistent set of rules to determine:

- Which switch should become the center of the STP topology?
- Which path is the best path?
- Which ports should forward traffic?
- Which redundant ports should be blocked?

STP makes these decisions using:

- Bridge Protocol Data Units (BPDUs)
- Bridge ID (BID)
- Root Bridge election
- Root Path Cost
- Root Ports
- Designated Ports
- Blocked Ports

---

# Root Bridge

The **Root Bridge** is the central reference point of the STP topology.

Every non-root switch determines its best path toward the Root Bridge.

```text
              Root Bridge
                  SW1
                 /   \
                /     \
              SW2-----SW3
```

All STP calculations are performed relative to the Root Bridge.

The Root Bridge should normally be a strategically selected switch, such as a Core or Distribution switch.

---

# Why Root Bridge Selection Matters

Consider an enterprise network containing:

- Core switches
- Distribution switches
- Access switches

The Root Bridge should be located at the logical center of the Layer 2 topology.

A poorly selected Root Bridge can result in:

- Inefficient traffic paths
- Important links being blocked
- Lower network performance
- Difficult troubleshooting
- Unpredictable failover behavior

Therefore, Root Bridge placement should be intentionally designed rather than left to chance.

---

# Bridge ID (BID)

Every switch participating in STP has a **Bridge ID (BID)**.

The Bridge ID is used during the Root Bridge election.

Simplified:

```text
Bridge ID
    │
    ├── Bridge Priority
    │
    └── MAC Address
```

The switch with the **lowest Bridge ID** becomes the Root Bridge.

```text
Lowest Bridge ID
        ↓
   Root Bridge
```

---

# Bridge Priority

The Bridge Priority is a configurable STP value.

The default STP priority is:

```text
32768
```

Lower values are preferred.

Example:

```text
SW1 Priority = 4096

SW2 Priority = 32768

SW3 Priority = 32768
```

Result:

```text
SW1
 ↓
Lowest Priority
 ↓
Root Bridge
```

---

# MAC Address as a Tiebreaker

If all switches have the same Bridge Priority, STP compares their MAC addresses.

The switch with the **lowest MAC address** becomes the Root Bridge.

Example:

| Switch | Priority | MAC Address | Result |
|--------|----------|-------------|--------|
| SW1 | 32768 | 0001.AAAA.BBBB | Root Bridge |
| SW2 | 32768 | 0010.AAAA.BBBB | Non-Root |
| SW3 | 32768 | 0020.AAAA.BBBB | Non-Root |

All priorities are equal.

Therefore:

```text
Lowest MAC Address
        ↓
       SW1
        ↓
   Root Bridge
```

---

# Why Older Switches May Become the Root Bridge

Older switches may have lower MAC addresses than newer switches.

If every switch uses the default priority:

```text
Priority = 32768
```

the switch with the lowest MAC address wins the Root Bridge election.

This may result in an older or less powerful switch becoming the Root Bridge.

However, relying on MAC addresses also prevents every newly connected switch from automatically becoming the Root Bridge and unnecessarily changing the STP topology.

In enterprise networks, administrators should not rely on this default behavior.

The Root Bridge should be intentionally selected by configuring Bridge Priority.

---

# BPDU – Bridge Protocol Data Unit

A **BPDU (Bridge Protocol Data Unit)** is an STP control message exchanged between switches.

BPDUs allow switches to exchange information about the STP topology.

BPDUs are used to:

- Elect the Root Bridge
- Advertise STP information
- Calculate the best path toward the Root Bridge
- Detect topology changes
- Maintain a loop-free Layer 2 topology

---

# BPDU Exchange

The STP process begins when switches exchange BPDUs.

Conceptually:

```text
SW1 ---- BPDU ----> SW2

SW2 ---- BPDU ----> SW3

SW3 ---- BPDU ----> SW1
```

The switches compare the STP information contained in the received BPDUs.

This information allows them to determine the Root Bridge and calculate the final loop-free topology.

---

# BPDU Hello Timer

In traditional STP, the default Hello Time is:

```text
2 seconds
```

The Root Bridge originates configuration BPDUs, which are then propagated through the STP topology.

BPDUs allow switches to continuously maintain information about the current Layer 2 topology.

---

# Root Bridge Election Process

The Root Bridge election follows this basic process:

```text
Compare Bridge IDs
        ↓
Lowest Bridge Priority Wins
        ↓
If Priority is Equal
        ↓
Lowest MAC Address Wins
        ↓
Root Bridge Selected
```

Simplified rule:

```text
Lowest BID Wins
```

---

# Port Cost

After electing the Root Bridge, every non-root switch must determine its best path toward the Root Bridge.

STP uses **Port Cost** to make this decision.

Port Cost is based on link speed.

General rule:

```text
Higher Speed
     ↓
Lower Cost
     ↓
Preferred Path
```

and:

```text
Lower Speed
     ↓
Higher Cost
     ↓
Less Preferred Path
```

---

# Important STP Cost Values

Common STP cost values:

| Link Speed | STP Cost |
|------------|----------|
| 10 Mbps | 100 |
| 100 Mbps | 19 |
| 1 Gbps | 4 |
| 10 Gbps | 2 |

Important CCNA values:

```text
100 Mbps = Cost 19

1 Gbps = Cost 4
```

Lower cost is preferred.

---

# Root Path Cost

The **Root Path Cost** is the total accumulated STP cost required to reach the Root Bridge.

Example:

```text
Root Bridge
    |
  1 Gbps
 Cost = 4
    |
   SW2
    |
 100 Mbps
 Cost = 19
    |
   SW3
```

SW3's Root Path Cost through SW2 is:

```text
19 + 4 = 23
```

STP compares all available paths and prefers the path with the lowest total Root Path Cost.

---

# Root Port (RP)

Every non-root switch selects one **Root Port**.

The Root Port is:

> The port with the lowest-cost path toward the Root Bridge.

Example:

```text
        Root Bridge
            SW1
             |
             |
            SW2
             ^
             |
         Root Port
```

Important rules:

```text
Every Non-Root Switch
        ↓
Exactly One Root Port
```

The Root Bridge has:

```text
Zero Root Ports
```

because the Root Bridge does not need a path toward itself.

---

# Designated Port (DP)

A **Designated Port** is the port selected to forward traffic for a particular network segment.

Designated Ports operate in a forwarding state.

Characteristics:

- Forward normal Ethernet traffic
- Participate in STP
- Send and receive BPDUs
- Provide the best path toward the Root Bridge for a segment

---

# Root Bridge Ports

An important STP rule:

> All active ports on the Root Bridge are Designated Ports.

Example:

```text
              Root Bridge
                  SW1

             DP /    \ DP
               /      \
             SW2      SW3
```

The Root Bridge has:

```text
Root Ports = 0

Designated Ports = All Active Ports
```

---

# Blocked Port

After selecting Root Ports and Designated Ports, STP places unnecessary redundant ports into a non-forwarding state.

Simplified in traditional STP terminology, these are commonly called **Blocked Ports**.

Example:

```text
              Root Bridge
                  SW1
                 /   \
                /     \
              SW2-----SW3
                      X
                   Blocked
```

The blocked port:

- Does not forward normal user traffic
- Prevents the Layer 2 loop
- Continues participating in STP
- Remains available as a redundant path

---

# Why Only One Side of a Redundant Link Blocks

Consider:

```text
SW2 -------- SW3
```

STP does not need to block both interfaces.

Possible result:

```text
SW2 Port → Designated Port → Forwarding

SW3 Port → Blocked Port → Non-Forwarding
```

Only one side needs to stop forwarding traffic to break the Layer 2 loop.

Blocking both sides would unnecessarily remove the entire connection from the logical topology.

---

# How All STP Concepts Work Together

The complete STP decision process can be simplified as:

```text
Switches Exchange BPDUs
          ↓
Compare Bridge IDs
          ↓
Elect Root Bridge
          ↓
Calculate Root Path Costs
          ↓
Select One Root Port Per Non-Root Switch
          ↓
Select One Designated Port Per Segment
          ↓
Place Remaining Redundant Ports
Into Non-Forwarding State
          ↓
Create Loop-Free Logical Topology
```

---

# Enterprise Network Design Perspective

Consider a hierarchical enterprise topology:

```text
                  Core Switch
                  Root Bridge
                    /     \
                   /       \
          Distribution   Distribution
              Switch         Switch
               /               \
              /                 \
         Access Switch      Access Switch
```

The Root Bridge should normally be intentionally placed within the network hierarchy.

This provides:

- Predictable Layer 2 paths
- Better traffic flow
- Easier troubleshooting
- Controlled redundancy
- Predictable failover behavior

A network engineer should understand which links STP will forward and block before deploying the topology.

---

# Real-World Best Practice

Never intentionally leave Root Bridge selection to chance in a production network.

Configure the desired Core or Distribution switch with a lower STP priority.

Conceptually:

```text
Primary Root Bridge
Priority = Lowest

Secondary Root Bridge
Priority = Second Lowest

Access Switches
Priority = Default/Higher
```

This creates a predictable STP hierarchy.

If the Primary Root Bridge fails, the Secondary Root Bridge can take over.

---

# Important STP Rules

## Rule 1: Lowest Bridge ID Wins

```text
Lowest BID
    ↓
Root Bridge
```

---

## Rule 2: Every Non-Root Switch Has One Root Port

```text
Non-Root Switch
       ↓
One Root Port
```

---

## Rule 3: Root Port Uses the Best Path to the Root Bridge

```text
Lowest Root Path Cost
          ↓
      Root Port
```

---

## Rule 4: Every Segment Has One Designated Port

```text
Network Segment
       ↓
One Designated Port
```

---

## Rule 5: Root Bridge Ports Are Designated Ports

```text
Root Bridge
     ↓
All Active Ports
     ↓
Designated Ports
```

---

## Rule 6: Remaining Redundant Ports Do Not Forward Normal Traffic

```text
Unnecessary Redundant Path
           ↓
     Non-Forwarding
           ↓
       Loop Prevented
```

---

# Key Terms

| Term | Definition |
|------|------------|
| STP | Layer 2 protocol used to prevent switching loops |
| Root Bridge | Central reference switch for all STP calculations |
| Bridge ID | Identifier used to elect the Root Bridge |
| Bridge Priority | Configurable component of the Bridge ID |
| BPDU | STP control message exchanged between switches |
| Port Cost | STP metric based on link speed |
| Root Path Cost | Total accumulated cost to reach the Root Bridge |
| Root Port | Best port toward the Root Bridge on a non-root switch |
| Designated Port | Port selected to forward traffic for a network segment |
| Blocked Port | Redundant port prevented from forwarding normal traffic |

---

# Key Takeaways

- STP uses a structured election and selection process to create a loop-free Layer 2 topology.
- The switch with the lowest Bridge ID becomes the Root Bridge.
- Bridge ID selection is based primarily on Bridge Priority and MAC address.
- Lower values are preferred in STP calculations.
- BPDUs carry STP information between switches.
- Every non-root switch selects exactly one Root Port.
- The Root Port provides the best path toward the Root Bridge.
- Every network segment selects a Designated Port.
- All active ports on the Root Bridge are Designated Ports.
- Remaining redundant ports are placed into a non-forwarding state to prevent loops.
- Root Bridge placement should be intentionally designed in enterprise networks.
- Understanding these concepts is essential before learning the detailed STP port selection rules.