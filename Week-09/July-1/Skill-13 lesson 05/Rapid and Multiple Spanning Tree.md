# Rapid and Multiple Spanning Tree

## Overview

Spanning Tree Protocol prevents Layer 2 loops in networks containing redundant links.

The original IEEE 802.1D STP successfully prevents loops but suffers from slow convergence. Depending on the failure scenario, classic STP can require up to approximately 50 seconds to restore connectivity.

Modern networks require faster failure recovery.

This led to the development of:

```text
Classic STP
     ↓
Rapid Spanning Tree Protocol (RSTP)
     ↓
Multiple Spanning Tree (MST)
```

The three major implementations are:

```text
Classic STP / PVST+
        ↓
Loop Prevention

Rapid PVST+
        ↓
Fast Convergence + Per-VLAN STP

MST
        ↓
Fast Convergence + Scalability
```

---

# 1. Problem with Classic STP

Classic STP can converge slowly after a topology change.

Default timers:

| Timer         |    Default |
| ------------- | ---------: |
| Hello Time    |  2 seconds |
| Max Age       | 20 seconds |
| Forward Delay | 15 seconds |

A traditional failure scenario may involve:

```text
Network Failure
      ↓
Wait for STP Information to Expire
      ↓
Max Age
20 seconds
      ↓
Listening
15 seconds
      ↓
Learning
15 seconds
      ↓
Forwarding
```

Potential convergence time:

```text
20 + 15 + 15

=

50 seconds
```

During this period:

* users may lose connectivity
* VoIP calls may disconnect
* application sessions may timeout
* video streams may stop
* business services may become unavailable

Classic STP prevents loops effectively but does not provide the convergence speed required by modern enterprise networks.

---

# 2. Rapid Spanning Tree Protocol

Rapid Spanning Tree Protocol is defined by:

```text
IEEE 802.1w
```

RSTP provides significantly faster convergence than classic STP.

The fundamental STP concepts remain unchanged.

RSTP still uses:

* BPDUs
* Bridge IDs
* Root Bridge election
* Root Path Cost
* Root Ports
* Designated Ports
* redundant path blocking

Therefore:

```text
Classic STP Knowledge
        ↓
Still Applies to RSTP
```

The primary difference is how quickly RSTP:

* detects failures
* communicates topology changes
* identifies alternate paths
* transitions eligible ports to forwarding

---

# 3. Classic STP vs RSTP

Classic STP:

```text
Failure
   ↓
Wait for Timers
   ↓
Recalculate Topology
   ↓
Listening
   ↓
Learning
   ↓
Forwarding
```

RSTP:

```text
Failure
   ↓
Detect Failure
   ↓
Identify Known Alternate Path
   ↓
Rapid Recalculation
   ↓
Forwarding
```

The objective of RSTP is:

```text
Faster Failure Detection

+

Faster Topology Decisions

+

Faster Convergence
```

---

# 4. RSTP Port States

Classic STP defines five port states:

```text
Disabled

Blocking

Listening

Learning

Forwarding
```

RSTP simplifies these into three states:

```text
Discarding

Learning

Forwarding
```

Mapping:

| Classic STP | RSTP       |
| ----------- | ---------- |
| Disabled    | Discarding |
| Blocking    | Discarding |
| Listening   | Discarding |
| Learning    | Learning   |
| Forwarding  | Forwarding |

Therefore:

```text
Classic STP

Disabled
Blocking
Listening
    │
    ▼
RSTP Discarding
```

---

# 5. Discarding State

A port in the Discarding state:

* does not forward user traffic
* does not learn MAC addresses
* processes STP information

```text
Discarding

User Traffic
     ✗

MAC Learning
     ✗

STP Participation
     ✓
```

RSTP combines the classic:

```text
Disabled

Blocking

Listening
```

states into:

```text
Discarding
```

---

# 6. Learning State

In the Learning state, the switch:

* learns source MAC addresses
* builds the MAC address table
* processes STP information
* does not forward normal user traffic

```text
Learning

User Traffic
     ✗

MAC Learning
     ✓

STP Participation
     ✓
```

---

# 7. Forwarding State

A port in the Forwarding state:

* forwards user traffic
* learns MAC addresses
* processes STP information

```text
Forwarding

User Traffic
     ✓

MAC Learning
     ✓

STP Participation
     ✓
```

RSTP state progression:

```text
Discarding
     ↓
Learning
     ↓
Forwarding
```

---

# 8. STP Port Roles

RSTP uses four important port roles:

```text
Root Port

Designated Port

Alternate Port

Backup Port
```

---

# 9. Root Port

The Root Port is the best path from a non-root switch toward the Root Bridge.

```text
               ROOT BRIDGE
                   SW01
                     │
                     │
                     RP
                     │
                   SW02
```

Each non-root switch normally has:

```text
One Root Port
```

The Root Port operates in the Forwarding state when the topology is stable.

---

# 10. Designated Port

The Designated Port is the best port for forwarding traffic onto a network segment.

```text
                 ROOT BRIDGE
                    SW01

               DP          DP
                │           │
                │           │
              SW02--------SW03
```

All active ports on the Root Bridge are Designated Ports.

---

# 11. Alternate Port

An Alternate Port provides an alternative path toward the Root Bridge.

Example:

```text
                 ROOT BRIDGE
                    SW01

                /          \
               /            \
          DP/FWD            DP/FWD
             │                 │
             │                 │
          RP/FWD           ALT/DISC
                \           /
                 \         /
                    SW02
```

On SW02:

```text
Primary Path
     ↓
Root Port
```

Backup path:

```text
Alternative Path
       ↓
Alternate Port
```

If the Root Port fails:

```text
Root Port Failure
       ↓
Alternate Path Available
       ↓
Rapid Recalculation
       ↓
Alternate Port Becomes Active
       ↓
Forwarding
```

This explicit Alternate Port role is an important reason RSTP converges faster than classic STP.

---

# 12. Backup Port

A Backup Port provides a redundant connection to the same shared network segment.

Conceptually:

```text
             Shared Segment
          ───────────────────
              │          │
              │          │
            Port A     Port B
                \       /
                 \     /
                  SWITCH
```

Difference:

```text
Alternate Port
      ↓
Alternative Path Toward Root Bridge
```

```text
Backup Port
      ↓
Backup Connection to the Same
Shared Network Segment
```

Backup Ports are uncommon in modern switched Ethernet networks.

---

# 13. Why RSTP Converges Faster

RSTP improves convergence through:

* explicit Alternate and Backup port roles
* faster failure detection
* active BPDU generation
* Proposal and Agreement mechanisms
* rapid transition of eligible ports
* direct physical link failure detection

Therefore:

```text
Classic STP
     ↓
Timer-Dependent Convergence
```

```text
RSTP
     ↓
Communication and Topology-Based
Rapid Convergence
```

---

# 14. RSTP BPDU Behavior

Classic STP primarily depends on the Root Bridge originating configuration BPDUs.

Conceptually:

```text
Root Bridge
     ↓
BPDU
     ↓
Switch
     ↓
Relayed STP Information
     ↓
Switch
```

RSTP changes BPDU behavior.

Every switch actively generates BPDUs.

```text
SW01 ─── BPDU ───► SW02

SW02 ─── BPDU ───► SW01
```

This allows switches to monitor neighboring devices more directly.

---

# 15. RSTP Failure Detection

Default Hello Time:

```text
2 seconds
```

If three consecutive BPDUs are missed:

```text
3 × Hello Time

=

3 × 2 seconds

=

6 seconds
```

RSTP can consider the neighbor information lost.

Conceptually:

```text
Classic STP

Max Age
    ↓
Up to 20 Seconds
```

```text
RSTP

3 Missed BPDUs
    ↓
Approximately 6 Seconds
```

Important:

RSTP does not simply change the traditional Max Age timer from 20 seconds to 6 seconds.

The better explanation is:

> RSTP can age out neighbor information after missing three consecutive BPDUs based on the Hello Time.

Cisco output may still display:

```text
Max Age 20 sec
```

while Rapid PVST+ is running.

---

# 16. Direct Link Failure Detection

Consider:

```text
SW01
  │
  │
  │
SW02
```

If the cable is physically disconnected:

```text
SW01
  X
  X
  X
SW02
```

The interface immediately transitions:

```text
UP
 ↓
DOWN
```

RSTP can detect the physical failure without waiting for three missed BPDUs.

The process becomes:

```text
Root Port Fails
       ↓
Physical Interface Goes Down
       ↓
Alternate Path Available
       ↓
Rapid Recalculation
       ↓
Alternate Path Activated
```

This can provide extremely fast convergence.

---

# 17. Proposal and Agreement Process

RSTP uses a Proposal and Agreement mechanism to rapidly establish a loop-free topology.

Consider:

```text
SW01 ───────────── SW02
```

The Designated Switch sends a:

```text
PROPOSAL
```

Conceptually:

```text
SW01

"I propose that this link
become part of the active
forwarding topology."

        │
        ▼

SW02
```

SW02 verifies and synchronizes its topology.

It then sends:

```text
AGREEMENT
```

Process:

```text
SW01
  │
  │ PROPOSAL
  ▼
SW02
  │
  │ AGREEMENT
  ▼
SW01
```

Result:

```text
Proposal
    ↓
Synchronization
    ↓
Agreement
    ↓
Rapid Forwarding Transition
```

Classic STP relies heavily on timers.

RSTP relies more heavily on:

```text
Direct Switch Communication

+

Topology Synchronization
```

This is one of the primary reasons RSTP converges rapidly.

---

# 18. RSTP Link Types

RSTP recognizes three important link types:

```text
Point-to-Point

Shared

Edge
```

The link type helps RSTP determine whether rapid transitions are safe.

---

# 19. Point-to-Point Link

A full-duplex switch-to-switch connection is normally considered Point-to-Point.

```text
SW01 ═══════════════ SW02
```

RSTP can use the Proposal and Agreement process across Point-to-Point links.

Cisco output example:

```text
Fa0/1    Root    FWD    19    128.1    P2p
```

Where:

```text
P2p
 ↓
Point-to-Point
```

---

# 20. Shared Link

A half-duplex Ethernet segment is considered a Shared link.

Historical example:

```text
               HUB
             /  │  \
            /   │   \
          SW1  SW2  SW3
```

Because multiple devices share the Ethernet segment, rapid transition mechanisms are more limited.

Shared Ethernet links are uncommon in modern enterprise networks.

---

# 21. Edge Port

An Edge Port connects to an endpoint rather than another switch.

Examples:

```text
PC

Server

Printer

IP Phone
```

Topology:

```text
SWITCH ───────── PC
```

An endpoint should not create a redundant Layer 2 switching path.

Therefore, the port can rapidly transition to Forwarding.

On Cisco switches, this concept is associated with:

```text
PortFast
```

---

# 22. Rapid PVST+

Cisco combines:

```text
RSTP
+
Per-VLAN Spanning Tree
```

to create:

```text
Rapid PVST+
```

Meaning:

```text
RAPID
   ↓
Fast RSTP Convergence

PER-VLAN
   ↓
Separate Instance for Each VLAN

SPANNING TREE
   ↓
Layer 2 Loop Prevention

+
   ↓
Cisco Implementation
```

Rapid PVST+ provides:

* fast convergence
* one STP instance per VLAN
* separate Root Bridges per VLAN
* per-VLAN forwarding topologies
* Layer 2 traffic engineering

---

# 23. Configuring Rapid PVST+

Command:

```text
Switch(config)# spanning-tree mode rapid-pvst
```

Example:

```text
SW01(config)# spanning-tree mode rapid-pvst
```

Configure the mode consistently on all participating switches.

Example:

```text
SW01(config)# spanning-tree mode rapid-pvst
```

```text
SW02(config)# spanning-tree mode rapid-pvst
```

```text
SW03(config)# spanning-tree mode rapid-pvst
```

---

# 24. Verifying Rapid PVST+

Display STP information:

```text
show spanning-tree
```

Classic STP output may display:

```text
Spanning tree enabled protocol ieee
```

Rapid PVST+ may display:

```text
Spanning tree enabled protocol rstp
```

Display a specific VLAN:

```text
show spanning-tree vlan 10
```

Display Root Bridge information:

```text
show spanning-tree root
```

---

# 25. Rapid PVST+ Enterprise Design

Example VLANs:

```text
VLAN 10 → Employees

VLAN 20 → Servers

VLAN 30 → Management

VLAN 40 → Voice
```

Topology:

```text
               DIST-SW01
                   ║
                   ║
               DIST-SW02

                /      \
               /        \
            ACCESS SWITCHES
```

Configure DIST-SW01:

```text
spanning-tree mode rapid-pvst

spanning-tree vlan 10,20 root primary

spanning-tree vlan 30,40 root secondary
```

Configure DIST-SW02:

```text
spanning-tree mode rapid-pvst

spanning-tree vlan 30,40 root primary

spanning-tree vlan 10,20 root secondary
```

Result:

```text
VLAN 10,20
     ↓
DIST-SW01
Root Primary
```

```text
VLAN 30,40
     ↓
DIST-SW02
Root Primary
```

This provides:

* loop prevention
* rapid convergence
* redundant Layer 2 paths
* predictable Root Bridge placement
* per-VLAN traffic engineering
* better infrastructure utilization

---

# 26. Scalability Problem with Rapid PVST+

Rapid PVST+ creates one STP instance per VLAN.

Example:

```text
VLAN 1
   ↓
STP Instance 1

VLAN 2
   ↓
STP Instance 2

VLAN 3
   ↓
STP Instance 3

...

VLAN 200
   ↓
STP Instance 200
```

Therefore:

```text
200 VLANs

=

200 STP Instances
```

Each instance requires resources to maintain its topology and process STP control information.

As the number of VLANs increases, maintaining one instance per VLAN becomes less scalable.

---

# 27. Multiple Spanning Tree

MST stands for:

```text
Multiple Spanning Tree
```

The original standard is associated with:

```text
IEEE 802.1s
```

MST was later incorporated into IEEE 802.1Q.

The central concept is:

> Multiple VLANs can be mapped to the same Spanning Tree instance.

---

# 28. Rapid PVST+ vs MST Instance Design

Rapid PVST+:

```text
VLAN 10 → STP Instance

VLAN 20 → STP Instance

VLAN 30 → STP Instance

VLAN 40 → STP Instance
```

Result:

```text
4 VLANs

=

4 STP Instances
```

MST:

```text
VLAN 10 ─┐
         ├── MST Instance 1
VLAN 20 ─┘


VLAN 30 ─┐
         ├── MST Instance 2
VLAN 40 ─┘
```

Result:

```text
4 VLANs

=

2 MST Instances
```

The primary advantage is scalability.

---

# 29. Large Enterprise MST Example

Suppose an enterprise network contains:

```text
VLAN 10-50
     ↓
Employee Networks

VLAN 51-100
     ↓
Server Networks

VLAN 101-150
     ↓
Voice Networks

VLAN 151-200
     ↓
Guest Networks
```

With Rapid PVST+:

```text
Approximately 200 VLANs

        ↓

Approximately 200 STP Instances
```

With MST:

```text
VLAN 10-50
     ↓
MST Instance 1

VLAN 51-100
     ↓
MST Instance 2

VLAN 101-150
     ↓
MST Instance 3

VLAN 151-200
     ↓
MST Instance 4
```

Therefore:

```text
Approximately 200 VLANs

        ↓

4 MST Instances
```

MST significantly reduces the number of STP instances.

---

# 30. MST Still Supports Traffic Engineering

Consider:

```text
               DIST-SW01
                   ║
                   ║
               DIST-SW02
```

Configure:

```text
MST Instance 1

VLAN 10-50

Root = DIST-SW01
```

Configure:

```text
MST Instance 2

VLAN 51-100

Root = DIST-SW02
```

Result:

```text
VLAN 10-50
     ↓
MST Instance 1
     ↓
DIST-SW01 Root
```

```text
VLAN 51-100
     ↓
MST Instance 2
     ↓
DIST-SW02 Root
```

Therefore, MST provides:

* fewer STP instances
* multiple Layer 2 forwarding topologies
* Root Bridge load distribution
* rapid convergence
* improved scalability

---

# 31. MST Region

Switches participating in the same MST Region must agree on:

```text
Region Name

Revision Number

VLAN-to-Instance Mapping
```

Example:

```text
SW01

Region Name:
CASTLE-RYSEN

Revision:
1

VLAN 10-50:
Instance 1

VLAN 51-100:
Instance 2
```

SW02 must have matching configuration:

```text
SW02

Region Name:
CASTLE-RYSEN

Revision:
1

VLAN 10-50:
Instance 1

VLAN 51-100:
Instance 2
```

If the configurations do not match, the switches are not members of the same MST Region.

---

# 32. Configuring MST

Enter MST configuration mode:

```text
Switch(config)# spanning-tree mst configuration
```

Configure the region name:

```text
Switch(config-mst)# name CASTLE-RYSEN
```

Configure the revision:

```text
Switch(config-mst)# revision 1
```

Map VLANs to Instance 1:

```text
Switch(config-mst)# instance 1 vlan 10-50
```

Map VLANs to Instance 2:

```text
Switch(config-mst)# instance 2 vlan 51-100
```

Exit MST configuration mode:

```text
Switch(config-mst)# exit
```

Enable MST:

```text
Switch(config)# spanning-tree mode mst
```

---

# 33. Verifying MST

Display MST configuration:

```text
show spanning-tree mst configuration
```

Display MST topology information:

```text
show spanning-tree mst
```

---

# 34. Castle Rysen Enterprise Example

Example network segments:

```text
Management VLANs

Internal VLANs

Camera VLANs

Guest VLANs
```

For a smaller Castle Rysen environment, Rapid PVST+ could provide:

```text
Each VLAN
    ↓
Separate Rapid STP Instance
```

For a large Castle Rysen regional infrastructure, MST could group VLANs:

```text
Management VLANs
       ↓
MST Instance 1

Internal VLANs
       ↓
MST Instance 2

Camera VLANs
       ↓
MST Instance 3

Guest VLANs
       ↓
MST Instance 4
```

Possible Root Bridge design:

```text
DIST-SW01

Root:
MST Instance 1
MST Instance 2
```

```text
DIST-SW02

Root:
MST Instance 3
MST Instance 4
```

This provides:

* Layer 2 loop prevention
* rapid convergence
* scalability
* deterministic Root Bridge placement
* multiple forwarding topologies
* better infrastructure utilization

---

# 35. Classic STP vs Rapid PVST+ vs MST

| Feature                     | Classic STP / PVST+ | Rapid PVST+       | MST                             |
| --------------------------- | ------------------- | ----------------- | ------------------------------- |
| Primary Standard            | IEEE 802.1D         | IEEE 802.1w       | Originally IEEE 802.1s          |
| Convergence                 | Slow                | Fast              | Fast                            |
| Per-VLAN Instances          | Yes                 | Yes               | No                              |
| Multiple VLANs Per Instance | No                  | No                | Yes                             |
| Root Bridge Election        | Yes                 | Yes               | Yes                             |
| Root Port                   | Yes                 | Yes               | Yes                             |
| Designated Port             | Yes                 | Yes               | Yes                             |
| Explicit Alternate Role     | No                  | Yes               | Yes                             |
| Proposal/Agreement          | No                  | Yes               | Yes                             |
| Large-Scale Efficiency      | Lower               | Moderate          | Higher                          |
| Primary Purpose             | Loop Prevention     | Rapid Convergence | Rapid Convergence + Scalability |

---

# 36. Choosing the Appropriate STP Implementation

```text
Legacy Network
     ↓
Classic STP / PVST+
```

```text
Small-to-Medium Cisco Network
     ↓
Rapid PVST+
```

```text
Large Network with Many VLANs
     ↓
MST
```

The exact choice depends on:

* network architecture
* number of VLANs
* hardware platform
* operational requirements
* interoperability requirements
* scalability requirements

---

# 37. Applying RSTP to the Packet Tracer Lab

Original topology:

```text
                 cafe01-SW01
                  ROOT BRIDGE

               Fa0/1      Fa0/2
                 │           │
                 │           │
               Fa0/1      Fa0/2
              RP/FWD     ALT/BLK

                 cafe01-SW02
```

Classic STP output:

```text
Spanning tree enabled protocol ieee
```

Enable Rapid PVST+:

```text
cafe01-SW01(config)#
spanning-tree mode rapid-pvst
```

```text
cafe01-SW02(config)#
spanning-tree mode rapid-pvst
```

Verify:

```text
show spanning-tree
```

The final topology may remain:

```text
SW01
   ↓
Root Bridge

SW02 Fa0/1
   ↓
Root Port

SW02 Fa0/2
   ↓
Alternate Port
```

The major difference is not necessarily the final topology.

The major difference is:

```text
Classic STP

Failure
   ↓
Slow Reconvergence
```

versus:

```text
Rapid PVST+

Failure
   ↓
Rapid Reconvergence
```

> RSTP does not necessarily create a different Layer 2 topology. It reaches and repairs the topology much faster.

---

# 38. Complete STP Evolution

```text
REDUNDANT LAYER 2 NETWORK
          │
          ▼
CLASSIC STP
          │
          ├── Prevents Loops
          ├── Root Bridge Election
          ├── Root Port Selection
          ├── Designated Port Selection
          └── Slow Convergence
          │
          ▼
RSTP
          │
          ├── Same Fundamental STP Logic
          ├── Faster Failure Detection
          ├── Alternate Ports
          ├── Backup Ports
          ├── Proposal/Agreement
          └── Rapid Convergence
          │
          ▼
RAPID PVST+
          │
          ├── RSTP Convergence
          └── One Instance Per VLAN
          │
          ▼
SCALABILITY PROBLEM
          │
          └── Large Number of VLAN Instances
          │
          ▼
MST
          │
          ├── Multiple VLANs Per Instance
          ├── Fewer STP Instances
          ├── Rapid Convergence
          ├── Multiple Topologies
          └── Better Scalability
```

# Key Takeaways

```text
Classic STP
     ↓
Slow Convergence
```

```text
RSTP
     ↓
IEEE 802.1w
     ↓
Fast Convergence
```

RSTP states:

```text
Discarding
     ↓
Learning
     ↓
Forwarding
```

RSTP roles:

```text
Root Port

Designated Port

Alternate Port

Backup Port
```

Rapid PVST+:

```text
RSTP
+
One STP Instance Per VLAN
```

Configuration:

```text
spanning-tree mode rapid-pvst
```

Verification:

```text
show spanning-tree
```

MST:

```text
Multiple VLANs
      ↓
Mapped to
      ↓
Fewer STP Instances
```

MST Region requirements:

```text
Same Region Name

Same Revision Number

Same VLAN-to-Instance Mapping
```

MST configuration:

```text
spanning-tree mst configuration

name CASTLE-RYSEN

revision 1

instance 1 vlan 10-50

instance 2 vlan 51-100

exit

spanning-tree mode mst
```

> **Classic STP, Rapid PVST+, and MST solve the same fundamental problem: preventing Layer 2 loops. Classic STP provides basic loop prevention but converges slowly. Rapid PVST+ adds fast convergence while maintaining one STP instance per VLAN. MST uses rapid convergence while grouping multiple VLANs into fewer instances, making it better suited to large-scale Layer 2 environments.**
