# Skill 13 Lesson 03 – Three Rules for STP Port Selections

## Overview

Spanning Tree Protocol (STP) does not randomly block redundant links.

STP follows a deterministic selection process to create a loop-free Layer 2 logical topology.

The complete process is:

```text
Elect Root Bridge
        ↓
Select Root Ports
        ↓
Select Designated Ports
        ↓
Block Remaining Ports
        ↓
Loop-Free Logical Topology
```

The lesson simplifies this into three main steps because the Root Bridge election occurs before port roles are selected:

```text
1. Select Root Ports
2. Select Designated Ports
3. Block All Remaining Ports
```

The most important rule when solving STP topology questions is:

> Never guess which port will block. Always follow the STP selection process in the correct order.

---

# STP Three-Tier Hierarchical Topology

Consider the following enterprise network:

```text
                         CORE LAYER

                            CORE1
                         Root Bridge
                         /         \
                        /           \

                    DISTRIBUTION LAYER

                     DIST1---------DIST2
                      |  \         /  |
                      |   \       /   |
                      |    \     /    |

                         ACCESS LAYER

                     ACC1   ACC2   ACC3
```

Assume:

```text
All Links = 1 Gbps

STP Cost = 4
```

Bridge ID order:

```text
CORE1 = Lowest BID

DIST1 BID < DIST2 BID
```

Therefore:

```text
CORE1 = Root Bridge
```

---

# Physical Topology vs Logical Topology

The physical topology contains redundant links.

```text
                              CORE1
                             /     \
                            /       \
                         DIST1-----DIST2
                          |  \     /  |
                          |   \   /   |
                          |    \ /    |
                         ACC1  ACC2  ACC3
```

This topology provides redundancy but also creates potential Layer 2 loops.

STP must transform the redundant physical topology into a loop-free logical topology.

```text
Physical Redundant Topology
            ↓
           STP
            ↓
Loop-Free Logical Topology
```

---

# Complete STP Selection Process

The process should always be performed in this order:

```text
STEP 0
Elect Root Bridge

        ↓

STEP 1
Select One Root Port
Per Non-Root Switch

        ↓

STEP 2
Select One Designated Port
Per Network Segment

        ↓

STEP 3
Block All Remaining Ports
```

---

# Step 0 – Elect the Root Bridge

Before STP can assign port roles, the switches must elect the Root Bridge.

The Root Bridge is selected using the Bridge ID.

Simplified Bridge ID:

```text
Bridge ID
    │
    ├── Bridge Priority
    │
    └── MAC Address
```

The switch with the lowest Bridge ID becomes the Root Bridge.

Selection process:

```text
Lowest Bridge Priority
        ↓
If Tie
        ↓
Lowest MAC Address
        ↓
Root Bridge
```

In this example:

```text
CORE1 = Lowest Bridge ID
```

Therefore:

```text
CORE1 = Root Bridge
```

---

# Root Bridge Characteristics

The Root Bridge has:

```text
Root Path Cost = 0
```

The Root Bridge has:

```text
Zero Root Ports
```

All active ports on the Root Bridge become:

```text
Designated Ports
```

Therefore:

```text
CORE1 → DIST1 = Designated Port

CORE1 → DIST2 = Designated Port
```

---

# Step 1 – Select Root Ports

After electing the Root Bridge, every non-root switch must select exactly one Root Port.

The Root Port represents:

> The best path from the non-root switch toward the Root Bridge.

Important rule:

```text
Root Bridge
     ↓
0 Root Ports
```

Every non-root switch:

```text
Exactly 1 Root Port
```

---

# Root Port Selection Criteria

STP selects the Root Port using the following criteria:

```text
1. Lowest Root Path Cost

        ↓ If Tie

2. Lowest Neighbor Bridge ID

        ↓ If Tie

3. Lowest Neighbor Port ID
```

Simplified:

```text
COST
  ↓
BID
  ↓
PORT ID
```

---

# Rule 1 – Lowest Root Path Cost

The first and most important selection factor is Root Path Cost.

STP assigns a cost to links based on their speed.

Common traditional STP costs:

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

General rule:

```text
Higher Speed
     ↓
Lower Cost
     ↓
Preferred by STP
```

---

# Root Path Cost

Root Path Cost is the total accumulated cost required to reach the Root Bridge.

Example:

```text
Root Bridge
     |
     | Cost 4
     |
    SW2
     |
     | Cost 4
     |
    SW3
```

SW3 Root Path Cost:

```text
4 + 4 = 8
```

STP compares the complete path cost, not only the cost of the directly connected link.

---

# DIST1 Root Port Selection

DIST1 has two possible paths toward CORE1.

## Direct Path

```text
DIST1 → CORE1
```

Cost:

```text
4
```

## Indirect Path

```text
DIST1 → DIST2 → CORE1
```

Cost:

```text
4 + 4 = 8
```

Comparison:

```text
Direct Path   = 4

Indirect Path = 8
```

Lower cost wins.

Therefore:

```text
DIST1 port toward CORE1 = Root Port
```

---

# DIST2 Root Port Selection

DIST2 also has two possible paths.

Direct:

```text
DIST2 → CORE1

Cost = 4
```

Indirect:

```text
DIST2 → DIST1 → CORE1

Cost = 4 + 4

Cost = 8
```

Therefore:

```text
DIST2 port toward CORE1 = Root Port
```

---

# ACC1 Root Port Selection

ACC1 reaches the Root Bridge through DIST1.

```text
ACC1 → DIST1 → CORE1
```

Cost:

```text
ACC1 → DIST1 = 4

DIST1 → CORE1 = 4
```

Total:

```text
4 + 4 = 8
```

Therefore:

```text
ACC1 port toward DIST1 = Root Port
```

---

# ACC3 Root Port Selection

ACC3 reaches CORE1 through DIST2.

```text
ACC3 → DIST2 → CORE1
```

Total cost:

```text
4 + 4 = 8
```

Therefore:

```text
ACC3 port toward DIST2 = Root Port
```

---

# ACC2 Root Port Selection

ACC2 is redundantly connected to both Distribution switches.

```text
                  DIST1             DIST2
                     \               /
                      \             /
                       \           /
                           ACC2
```

ACC2 has two possible paths toward the Root Bridge.

## Path Through DIST1

```text
ACC2 → DIST1 → CORE1
```

Cost:

```text
4 + 4 = 8
```

## Path Through DIST2

```text
ACC2 → DIST2 → CORE1
```

Cost:

```text
4 + 4 = 8
```

Comparison:

```text
Path Through DIST1 = 8

Path Through DIST2 = 8
```

The Root Path Costs are equal.

Therefore, STP moves to the second tiebreaker:

```text
Lowest Neighbor Bridge ID
```

Assume:

```text
DIST1 BID < DIST2 BID
```

Therefore:

```text
ACC2 port toward DIST1 = Root Port
```

---

# Root Port Selection Results

| Switch | Root Port |
|--------|-----------|
| CORE1 | None |
| DIST1 | Toward CORE1 |
| DIST2 | Toward CORE1 |
| ACC1 | Toward DIST1 |
| ACC2 | Toward DIST1 |
| ACC3 | Toward DIST2 |

The topology now becomes:

```text
                              CORE1
                           Root Bridge

                           DP         DP
                           /           \
                          /             \
                         RP             RP
                       DIST1-----------DIST2
                        |  \           /  |
                        |   \         /   |
                        |    \       /    |
                       RP     RP           RP
                        |      \           |
                       ACC1    ACC2        ACC3
```

---

# Root Port Rule

Remember:

```text
ROOT PORT
    ↓
Selected Per Non-Root Switch
```

Every non-root switch gets:

```text
Exactly One Root Port
```

The Root Bridge gets:

```text
Zero Root Ports
```

---

# Step 2 – Select Designated Ports

After selecting Root Ports, STP selects Designated Ports.

A Designated Port is selected:

> Once per network segment.

Important difference:

```text
Root Port
    ↓
Per Non-Root Switch
```

```text
Designated Port
    ↓
Per Network Segment
```

---

# Designated Port Selection Criteria

For every network segment, STP compares:

```text
1. Lowest Root Path Cost

        ↓ If Tie

2. Lowest Bridge ID

        ↓ If Tie

3. Lowest Port ID
```

The winner becomes the Designated Port.

---

# Root Bridge Designated Ports

All active ports on the Root Bridge automatically become Designated Ports.

Therefore:

```text
CORE1 → DIST1 = DP

CORE1 → DIST2 = DP
```

Why?

Because:

```text
CORE1 Root Path Cost = 0
```

No other switch can have a better path to the Root Bridge than the Root Bridge itself.

---

# DIST1 to ACC1 Segment

Consider:

```text
DIST1 -------- ACC1
```

Root Path Costs:

```text
DIST1 = 4

ACC1 = 8
```

Lower cost wins.

Therefore:

```text
DIST1 Port = Designated Port

ACC1 Port = Root Port
```

---

# DIST2 to ACC3 Segment

Consider:

```text
DIST2 -------- ACC3
```

Root Path Costs:

```text
DIST2 = 4

ACC3 = 8
```

Therefore:

```text
DIST2 Port = Designated Port

ACC3 Port = Root Port
```

---

# DIST1 to ACC2 Segment

Consider:

```text
DIST1 -------- ACC2
```

Root Path Costs:

```text
DIST1 = 4

ACC2 = 8
```

Therefore:

```text
DIST1 Port = Designated Port

ACC2 Port = Root Port
```

---

# DIST2 to ACC2 Segment

Consider:

```text
DIST2 -------- ACC2
```

Root Path Costs:

```text
DIST2 = 4

ACC2 = 8
```

DIST2 has the lower Root Path Cost.

Therefore:

```text
DIST2 Port = Designated Port
```

The ACC2 interface is:

```text
Not Root Port
```

because ACC2 already selected its connection toward DIST1 as the Root Port.

The ACC2 interface is also:

```text
Not Designated Port
```

because DIST2 won the Designated Port election.

Therefore, this interface will become blocked during Step 3.

---

# DIST1 to DIST2 Segment

Consider:

```text
DIST1 -------- DIST2
```

Compare Root Path Costs:

```text
DIST1 = 4

DIST2 = 4
```

Tie.

Move to the second criterion:

```text
Lowest Bridge ID
```

Assume:

```text
DIST1 BID < DIST2 BID
```

Therefore:

```text
DIST1 Port = Designated Port
```

The DIST2 interface is:

```text
Not Root Port
```

and:

```text
Not Designated Port
```

Therefore, it will become blocked during Step 3.

---

# Designated Port Rule

Remember:

```text
DESIGNATED PORT
       ↓
Selected Per Network Segment
```

Every segment gets:

```text
Exactly One Designated Port
```

Also:

```text
All Active Root Bridge Ports
              ↓
      Designated Ports
```

---

# Step 3 – Block All Remaining Ports

After Root Ports and Designated Ports have been selected, STP examines the remaining ports.

Any port that is:

```text
Not Root Port
```

and:

```text
Not Designated Port
```

becomes:

```text
Blocked / Non-Forwarding
```

---

# Blocked Port 1 – DIST2 Toward DIST1

On the DIST1-DIST2 segment:

```text
DIST1 Root Path Cost = 4

DIST2 Root Path Cost = 4
```

Tie.

Compare Bridge IDs:

```text
DIST1 BID < DIST2 BID
```

Therefore:

```text
DIST1 = Designated Port

DIST2 = Blocked Port
```

---

# Blocked Port 2 – ACC2 Toward DIST2

ACC2 selected its connection toward DIST1 as its Root Port.

```text
ACC2 → DIST1 = Root Port
```

On the ACC2-DIST2 segment:

```text
DIST2 Root Path Cost = 4

ACC2 Root Path Cost = 8
```

Therefore:

```text
DIST2 = Designated Port

ACC2 = Blocked Port
```

---

# Final STP Port Roles

The completed topology is:

```text
                              CORE1
                           ROOT BRIDGE

                           DP         DP
                           /           \
                          /             \
                         RP             RP
                       DIST1-----------DIST2
                         DP             BLK
                        |  \           /  |
                       DP   DP        DP  DP
                        |     \       /    |
                        |      \     /     |
                       RP       RP BLK     RP
                        |         \ /       |
                       ACC1       ACC2     ACC3
```

---

# Final Logical STP Topology

Mentally remove the blocked connections.

Physical topology:

```text
                              CORE1
                             /     \
                            /       \
                         DIST1-----DIST2
                          |  \     /  |
                          |   \   /   |
                          |    \ /    |
                         ACC1  ACC2  ACC3
```

Blocked connections:

```text
DIST2 → DIST1 = Blocked

ACC2 → DIST2 = Blocked
```

Logical topology:

```text
                              CORE1
                             /     \
                            /       \
                         DIST1     DIST2
                          /  \        |
                         /    \       |
                      ACC1   ACC2    ACC3
```

The result is:

```text
Full Connectivity

        +

No Layer 2 Loops

        +

Redundant Backup Links
```

---

# What Happens During a Link Failure?

Initially:

```text
ACC2 → DIST1 = Root Port

ACC2 → DIST2 = Blocked
```

Suppose DIST1 fails.

```text
DIST1
  X
DOWN
```

ACC2 loses its path toward the Root Bridge.

STP detects the topology change and recalculates the topology.

The redundant connection:

```text
ACC2 → DIST2
```

can transition to forwarding.

New logical path:

```text
ACC2 → DIST2 → CORE1
```

Therefore:

```text
Redundancy
     +
STP Loop Prevention
     =
Network Availability
```

---

# Complete STP Decision Process

```text
                    START
                      ↓
              ELECT ROOT BRIDGE
                      ↓
               Lowest Bridge ID
                      ↓
            ┌─────────┴─────────┐
            ↓                   ↓
       ROOT BRIDGE        NON-ROOT SWITCHES
            ↓                   ↓
     Zero Root Ports      Select One RP Each
            ↓                   ↓
   All Active Ports DP    Lowest Root Path Cost
                                ↓
                         Lowest Neighbor BID
                                ↓
                      Lowest Neighbor Port ID
                                ↓
                    SELECT DESIGNATED PORTS
                                ↓
                    One DP Per Network Segment
                                ↓
                       Lowest Root Path Cost
                                ↓
                            Lowest BID
                                ↓
                          Lowest Port ID
                                ↓
                      BLOCK ALL LEFTOVERS
                                ↓
                    LOOP-FREE LOGICAL TOPOLOGY
```

---

# Three Main STP Port Selection Rules

## Rule 1 – Select Root Ports

Every non-root switch selects exactly one Root Port.

Selection:

```text
Lowest Root Path Cost
        ↓
Lowest Neighbor Bridge ID
        ↓
Lowest Neighbor Port ID
```

Remember:

```text
ROOT PORT = PER NON-ROOT SWITCH
```

---

## Rule 2 – Select Designated Ports

Every network segment selects exactly one Designated Port.

Selection:

```text
Lowest Root Path Cost
        ↓
Lowest Bridge ID
        ↓
Lowest Port ID
```

Remember:

```text
DESIGNATED PORT = PER SEGMENT
```

Also:

```text
ALL ROOT BRIDGE PORTS = DESIGNATED PORTS
```

---

## Rule 3 – Block All Remaining Ports

Any port that is neither:

```text
Root Port
```

nor:

```text
Designated Port
```

becomes:

```text
Blocked / Non-Forwarding
```

---

# Important CCNA STP Tiebreakers

## Root Port Selection

```text
1. Lowest Root Path Cost

2. Lowest Neighbor Bridge ID

3. Lowest Neighbor Port ID
```

---

## Designated Port Selection

```text
1. Lowest Root Path Cost

2. Lowest Bridge ID

3. Lowest Port ID
```

---

# Root Port vs Designated Port

| Feature | Root Port | Designated Port |
|---------|-----------|-----------------|
| Selected Per | Non-root switch | Network segment |
| Quantity | One per non-root switch | One per segment |
| Purpose | Best path toward Root Bridge | Best forwarding port for the segment |
| State | Forwarding | Forwarding |
| Root Bridge | Has no Root Ports | All active ports are Designated |

---

# Enterprise Network Design Perspective

In an enterprise hierarchical network:

```text
                    CORE
                      ↓
                DISTRIBUTION
                      ↓
                   ACCESS
```

Root Bridge placement should be intentionally designed.

The Root Bridge should normally be placed on a powerful, centrally located switch.

However, in many modern enterprise three-tier networks, the Core Layer operates primarily at Layer 3.

Therefore, STP is commonly more relevant between:

```text
Distribution Layer
        ↕
Access Layer
```

A more realistic enterprise design may use:

```text
                   DIST1                 DIST2
              Primary Root         Secondary Root
                  /   \                 /   \
                 /     \               /     \
              ACC1     ACC2         ACC3     ACC4
```

This provides:

- Predictable Layer 2 forwarding paths
- Controlled redundancy
- Better failover behavior
- Easier troubleshooting
- Efficient network performance

---

# Real-World Best Practice

Never intentionally leave Root Bridge selection to chance.

Configure:

```text
Primary Root Bridge
        ↓
Lowest STP Priority
```

and:

```text
Secondary Root Bridge
        ↓
Second Lowest STP Priority
```

Access switches should not normally become the Root Bridge.

---

# Best Method for Solving STP Topology Questions

Whenever solving an STP topology, use this exact order:

```text
1. FIND THE ROOT BRIDGE

        ↓

2. FIND ONE ROOT PORT
   ON EVERY NON-ROOT SWITCH

        ↓

3. FIND ONE DESIGNATED PORT
   ON EVERY NETWORK SEGMENT

        ↓

4. BLOCK ALL REMAINING PORTS
```

Never begin by guessing which link will block.

Always calculate the topology systematically.

---

# Key Terms

| Term | Definition |
|------|------------|
| Root Bridge | Central reference switch for all STP calculations |
| Bridge ID | Value used to elect the Root Bridge |
| Root Path Cost | Total accumulated STP cost toward the Root Bridge |
| Root Port | Best path toward the Root Bridge on a non-root switch |
| Designated Port | Best forwarding port for a network segment |
| Blocked Port | Remaining redundant port prevented from forwarding traffic |
| BPDU | STP control message used to exchange topology information |
| Port ID | STP identifier containing Port Priority and Port Number |

---

# Key Takeaways

- STP does not randomly block redundant links.
- STP follows a deterministic selection process.
- First, the Root Bridge is elected.
- The switch with the lowest Bridge ID becomes the Root Bridge.
- Every non-root switch selects exactly one Root Port.
- Root Ports are selected using Root Path Cost and STP tiebreakers.
- Every network segment selects exactly one Designated Port.
- All active Root Bridge ports are Designated Ports.
- Ports that are neither Root Ports nor Designated Ports become non-forwarding.
- Root Ports are selected per non-root switch.
- Designated Ports are selected per network segment.
- Lower STP values are preferred.
- Root Bridge placement should be intentionally designed in enterprise networks.
- Redundant physical links remain available even when STP prevents them from forwarding.
- If an active path fails, STP can recalculate the topology and activate a redundant path.

---

# Final Memory Formula

```text
ROOT BRIDGE
     ↓
Lowest Bridge ID
     ↓
ROOT PORTS
     ↓
One Per Non-Root Switch
     ↓
Lowest Cost → Lowest Neighbor BID → Lowest Neighbor Port ID
     ↓
DESIGNATED PORTS
     ↓
One Per Network Segment
     ↓
Lowest Cost → Lowest BID → Lowest Port ID
     ↓
BLOCK EVERYTHING ELSE
     ↓
LOOP-FREE LOGICAL TOPOLOGY
```

> **Never guess the blocked port. Elect the Root Bridge, select Root Ports, select Designated Ports, and then block everything left over.**