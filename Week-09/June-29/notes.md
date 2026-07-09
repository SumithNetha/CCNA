# Week 9 — June 29, 2026

## Skill 12 Lesson 10 — What Now?

### Overview

Completing the VLAN lessons does not automatically mean VLANs have been mastered. VLAN knowledge becomes practical only when networks are designed, configured, tested, intentionally broken, and troubleshot.

VLANs are fundamental to enterprise networking because they provide logical segmentation, smaller broadcast domains, improved security boundaries, and better traffic management.

---

## Why VLANs Matter

A flat network places all devices in the same broadcast domain.

```text
Employees
POS Systems
Security Cameras
Guest Wi-Fi
Servers
VoIP Phones
Network Devices
        │
        ▼
ONE LARGE BROADCAST DOMAIN
```

This creates problems with:

* Security
* Broadcast traffic
* Network management
* Troubleshooting
* Traffic control

VLANs solve this problem by logically separating devices according to their function.

```text
VLAN 10 → Employees
VLAN 20 → Point-of-Sale
VLAN 30 → Security Cameras
VLAN 40 → Guest Wi-Fi
VLAN 50 → Servers
VLAN 99 → Network Management
```

A VLAN creates a separate Layer 2 broadcast domain.

---

## Complete VLAN Learning Process

```text
Why VLANs Exist
        ↓
Create and Name VLANs
        ↓
Configure Access Ports
        ↓
Configure Trunk Links
        ↓
Configure Inter-VLAN Routing
        ↓
Control DTP
        ↓
Understand VTP
        ↓
Configure the Native VLAN
        ↓
Design the VLAN Network
        ↓
Test Connectivity
        ↓
Troubleshoot Problems
```

---

## Creating and Naming VLANs

Create a VLAN:

```cisco
Switch(config)# vlan 10
Switch(config-vlan)# name EMPLOYEES
```

Create multiple VLANs:

```cisco
Switch(config)# vlan 20
Switch(config-vlan)# name CAMERAS

Switch(config)# vlan 30
Switch(config-vlan)# name GUEST

Switch(config)# vlan 99
Switch(config-vlan)# name MANAGEMENT
```

Verify VLANs:

```cisco
Switch# show vlan brief
```

Creating a VLAN does not automatically assign switch ports to that VLAN.

---

## Configuring Access Ports

An access port normally carries traffic for a single VLAN.

```cisco
Switch(config)# interface fastEthernet0/1
Switch(config-if)# switchport mode access
Switch(config-if)# switchport access vlan 10
```

Configure multiple interfaces:

```cisco
Switch(config)# interface range fastEthernet0/1-10
Switch(config-if-range)# switchport mode access
Switch(config-if-range)# switchport access vlan 10
```

Verify:

```cisco
Switch# show vlan brief
```

```cisco
Switch# show interfaces switchport
```

---

## Access Port vs Trunk Port

### Access Port

An access port normally carries traffic belonging to one VLAN.

```text
PC
 │
 │ Access Port
 │ VLAN 10
 │
SWITCH
```

### Trunk Port

A trunk carries traffic belonging to multiple VLANs.

```text
SW1 ======================== SW2
             TRUNK

       VLAN 10
       VLAN 20
       VLAN 30
       VLAN 99
```

---

## Configuring Trunks

Configure a trunk:

```cisco
Switch(config)# interface gigabitEthernet0/1
Switch(config-if)# switchport mode trunk
```

Disable DTP negotiation when appropriate:

```cisco
Switch(config-if)# switchport nonegotiate
```

Verify trunks:

```cisco
Switch# show interfaces trunk
```

---

## Inter-VLAN Routing

Different VLANs are different Layer 2 broadcast domains and normally use different IP networks.

Example:

```text
VLAN 10
192.168.10.0/24

VLAN 20
192.168.20.0/24
```

Devices in different VLANs require a Layer 3 device to communicate.

```text
VLAN 10
   │
   ▼
ROUTER
   │
   ▼
VLAN 20
```

---

## Router-on-a-Stick

Router-on-a-Stick uses:

* One physical router interface
* Multiple logical subinterfaces
* An 802.1Q trunk between the router and switch

```text
              ROUTER
                 │
                 │ 802.1Q Trunk
                 │
              SWITCH
              /    \
             /      \
        VLAN 10    VLAN 20
```

Configuration:

```cisco
Router(config)# interface gigabitEthernet0/0.10
Router(config-subif)# encapsulation dot1Q 10
Router(config-subif)# ip address 192.168.10.1 255.255.255.0
```

```cisco
Router(config)# interface gigabitEthernet0/0.20
Router(config-subif)# encapsulation dot1Q 20
Router(config-subif)# ip address 192.168.20.1 255.255.255.0
```

---

## DTP

DTP stands for:

```text
Dynamic Trunking Protocol
```

DTP allows Cisco switches to dynamically negotiate trunk connections.

Instead of unnecessarily relying on dynamic negotiation, explicitly configure ports.

Access port:

```cisco
switchport mode access
```

Trunk port:

```cisco
switchport mode trunk
switchport nonegotiate
```

Explicit configuration provides:

* Better predictability
* Improved security
* Easier troubleshooting

---

## VTP

VTP stands for:

```text
VLAN Trunking Protocol
```

VTP can distribute VLAN information between Cisco switches.

Important VTP modes include:

```text
Server
Client
Transparent
```

Incorrect VTP configurations can cause serious network problems.

The important principle is to understand and control how VLAN information is distributed throughout the network.

---

## Native VLAN

On an 802.1Q trunk, normal VLAN traffic is tagged.

```text
VLAN 10 → Tagged
VLAN 20 → Tagged
VLAN 30 → Tagged
```

Native VLAN traffic is untagged by default.

```text
Native VLAN → Untagged
```

Configure the native VLAN:

```cisco
Switch(config)# interface gigabitEthernet0/1
Switch(config-if)# switchport trunk native vlan 99
```

The native VLAN must match on both ends of the trunk.

```text
SW1 Native VLAN 99
          =
SW2 Native VLAN 99
```

A mismatch can create connectivity and security problems.

---

## Build VLAN Networks Yourself

A good VLAN practice network could use:

| VLAN | Name       | Network         | Gateway      |
| ---- | ---------- | --------------- | ------------ |
| 10   | EMPLOYEES  | 192.168.10.0/24 | 192.168.10.1 |
| 20   | CAMERAS    | 192.168.20.0/24 | 192.168.20.1 |
| 30   | GUEST      | 192.168.30.0/24 | 192.168.30.1 |
| 99   | MANAGEMENT | 192.168.99.0/24 | 192.168.99.1 |

The implementation process should be:

```text
Design
   ↓
Create VLANs
   ↓
Configure Access Ports
   ↓
Configure Trunks
   ↓
Configure Routing
   ↓
Test
   ↓
Troubleshoot
```

---

## Important Verification Commands

### Check VLANs

```cisco
show vlan brief
```

### Check Trunks

```cisco
show interfaces trunk
```

### Check Switchport Configuration

```cisco
show interfaces switchport
```

### Check Interface Status

```cisco
show ip interface brief
```

### Check Configuration

```cisco
show running-config
```

### Test Connectivity

```cisco
ping <destination-ip>
```

---

## Practice by Breaking the Network

Intentionally create configuration problems.

### Wrong Access VLAN

```cisco
switchport access vlan 20
```

when the device should belong to VLAN 10.

Verify:

```cisco
show vlan brief
```

---

### Missing VLAN

Create a VLAN on one switch but not another.

Verify:

```cisco
show vlan brief
```

---

### Trunk Misconfiguration

Incorrectly configure one side of the trunk.

Verify:

```cisco
show interfaces trunk
```

```cisco
show interfaces switchport
```

---

### Native VLAN Mismatch

Example:

```text
SW1 Native VLAN → 99

SW2 Native VLAN → 1
```

Troubleshoot the mismatch.

---

### Incorrect Inter-VLAN Routing

Possible problems:

* Incorrect VLAN ID
* Incorrect `encapsulation dot1Q` command
* Incorrect gateway
* Incorrect IP address
* Router interface shutdown

Verify:

```cisco
show ip interface brief
```

```cisco
show running-config
```

```cisco
ping <destination-ip>
```

---

## VLAN Troubleshooting Method

When VLAN communication fails, follow the traffic path.

```text
Source Device
      ↓
Correct IP Address?
      ↓
Correct Subnet Mask?
      ↓
Correct Default Gateway?
      ↓
Correct Access VLAN?
      ↓
Does the VLAN Exist?
      ↓
Is the Trunk Working?
      ↓
Is the VLAN Allowed on the Trunk?
      ↓
Do Native VLANs Match?
      ↓
Is Inter-VLAN Routing Working?
      ↓
Is the Destination Configured Correctly?
```

Do not randomly change configurations.

Troubleshoot systematically.

---

## Key Takeaway

> VLAN knowledge becomes a practical skill when you can design, configure, verify, break, troubleshoot, and repair a VLAN network yourself.

---

# Skill 13 Lesson 00 — Why We Need This Skill

## Spanning Tree Protocol (STP)

### Overview

Spanning Tree Protocol is one of the most important Layer 2 technologies used in switched networks.

The fundamental problem is:

```text
Networks Need Redundancy
          ↓
Redundant Layer 2 Links
          ↓
Potential Switching Loops
          ↓
Broadcast Storms
          ↓
Network Failure
```

STP solves this problem.

> **Spanning Tree Protocol prevents Layer 2 switching loops by logically blocking redundant paths while keeping those paths available as backups.**

---

## Why Networks Need Redundancy

Consider two switches with only one connection:

```text
SW1
 │
 │
SW2
```

If the connection fails:

```text
SW1

 X    Link Failure

SW2
```

Communication stops.

This connection is a:

```text
Single Point of Failure
```

The obvious solution is to add another connection.

```text
       Link 1
SW1 =========== SW2
     ===========
       Link 2
```

Now the network has redundancy.

If Link 1 fails:

```text
       Link 1
SW1 -----X----- SW2
     ===========
       Link 2
```

traffic can use Link 2.

However, redundant Layer 2 connections introduce another problem.

---

## Layer 2 Switching Loops

Consider three interconnected switches:

```text
              SW1
             /   \
            /     \
          SW2-----SW3
```

There are multiple paths between the switches.

This provides redundancy.

However, it also creates the possibility of a Layer 2 switching loop.

---

## How a Broadcast Loop Happens

Suppose a device sends a broadcast frame.

```text
Destination MAC:

FF:FF:FF:FF:FF:FF
```

SW1 receives the broadcast.

A switch forwards broadcast traffic out all appropriate ports in the same VLAN except the port on which the frame was received.

SW1 forwards the frame toward SW2 and SW3.

```text
              SW1
             ↙   ↘
           SW2   SW3
```

SW2 receives the frame and forwards it.

SW3 also receives the frame and forwards it.

```text
              SW1
             ↙   ↘
           SW2 ↔ SW3
```

The broadcast frames can continue circulating through the network.

---

## Ethernet Frames Do Not Have TTL

This is one of the major reasons Layer 2 loops are dangerous.

IP packets contain:

```text
TTL
Time To Live
```

Every router decreases the TTL value.

```text
TTL 3
  ↓
TTL 2
  ↓
TTL 1
  ↓
TTL 0
  ↓
Packet Discarded
```

Ethernet frames do not have a TTL field.

Therefore:

```text
Ethernet Frame
      ↓
Switching Loop
      ↓
No TTL
      ↓
Frame Continues Circulating
```

The frame has no built-in mechanism that causes it to expire after crossing a certain number of switches.

---

## Broadcast Storm

A broadcast storm occurs when broadcast frames continuously circulate and multiply within a Layer 2 network.

```text
Broadcast Frame
      ↓
     SW1
    ↙   ↘
  SW2   SW3
    ↘   ↙
     Loop
      ↓
Frames Continue Circulating
      ↓
More Network Traffic
      ↓
Network Resources Overwhelmed
      ↓
NETWORK FAILURE
```

Possible consequences include:

* High network utilization
* Bandwidth exhaustion
* Switch resource exhaustion
* Legitimate traffic unable to pass
* Network instability
* Complete network outage

---

## Why Not Remove Redundant Links?

Removing redundant links would prevent Layer 2 loops.

```text
SW1 -------- SW2 -------- SW3
```

However, this introduces single points of failure.

If one connection fails:

```text
SW1 -------- SW2 ----X---- SW3
```

SW3 becomes isolated.

Therefore:

```text
WITHOUT REDUNDANCY

Advantage:
No Layer 2 Loop

Disadvantage:
Single Point of Failure
```

Compared with:

```text
WITH REDUNDANCY

Advantage:
Backup Paths

Disadvantage:
Potential Layer 2 Loops
```

What the network needs is:

```text
REDUNDANCY
     +
NO LAYER 2 LOOPS
```

This is the purpose of STP.

---

## What Is STP?

STP stands for:

```text
Spanning Tree Protocol
```

Classic STP is defined by:

```text
IEEE 802.1D
```

Its purpose is to create a loop-free logical Layer 2 topology.

Consider the physical topology:

```text
              SW1
             /   \
            /     \
          SW2-----SW3
```

There is physical redundancy.

STP logically blocks one redundant path:

```text
              SW1
             /   \
            /     \
          SW2--X--SW3
```

The physical cable still exists.

However, the selected redundant path does not forward normal user traffic.

---

## STP Does Not Physically Disable the Link

This is an important distinction.

STP does not:

```text
Remove the Cable
```

Instead, STP:

```text
Logically Blocks the Redundant Path
```

Example:

```text
SW2 ---- BLOCKED ---- SW3
```

The redundant link remains available.

If the primary path fails:

```text
              SW1
             X   \
                  \
          SW2-----SW3
```

STP recalculates the topology.

The previously blocked path can become active.

```text
SW2 → SW3 → SW1
```

Therefore, STP provides:

```text
Loop Prevention
       +
Redundancy
```

---

## Basic STP Operation

The complete STP process contains several mechanisms.

At a high level:

```text
Elect the Root Bridge
          ↓
Select Root Ports
          ↓
Select Designated Ports
          ↓
Block Remaining Redundant Paths
          ↓
Create Loop-Free Topology
```

The three major decisions are:

### Step 1 — Elect the Root Bridge

```text
All Switches
     ↓
Compare STP Information
     ↓
One Switch Becomes
ROOT BRIDGE
```

### Step 2 — Select Root Ports

Every non-root switch selects its best path toward the Root Bridge.

```text
Non-Root Switch
       ↓
Best Path to Root Bridge
       ↓
ROOT PORT
```

### Step 3 — Select Designated Ports

Each network segment selects a Designated Port.

```text
Network Segment
       ↓
Best Port for Forwarding
       ↓
DESIGNATED PORT
```

Remaining redundant ports can be placed into a non-forwarding state to prevent loops.

---

## STP Concepts Covered in Upcoming Lessons

Important STP concepts include:

* Root Bridge
* Bridge ID
* Bridge Priority
* MAC Address
* BPDU
* Root Port
* Designated Port
* Non-Designated Port
* Path Cost
* Port Roles
* Port States
* STP Timers
* Topology Changes

---

## Why STP Matters in Real Networks

Enterprise networks require redundant connections.

Example:

```text
                 CORE
                /    \
               /      \
             SW1------SW2
```

Without STP:

```text
Redundant Links
       ↓
Switching Loop
       ↓
Broadcast Storm
       ↓
Network Outage
```

With STP:

```text
                 CORE
                /    \
               /      \
             SW1--X---SW2
```

One redundant path is logically blocked.

If an active connection fails:

```text
                 CORE
                X    \
                      \
             SW1------SW2
```

STP can use the backup path.

---

## Important CCNA Distinction

Incorrect statement:

> STP prevents broadcasts.

Correct statement:

> STP prevents Layer 2 switching loops that could cause broadcasts to circulate indefinitely and create broadcast storms.

Broadcast traffic is normal and necessary in Ethernet networks.

The problem is not the existence of broadcasts.

The problem is:

```text
Broadcast
    +
Layer 2 Loop
    +
No Ethernet TTL
    =
Broadcast Storm
```

---

## VLANs to STP Learning Progression

The transition from VLANs to STP follows the growth of the switched network.

```text
VLANs
   ↓
Logical Network Segmentation
   ↓
Trunks
   ↓
Extend VLANs Between Switches
   ↓
Inter-VLAN Routing
   ↓
Communication Between VLANs
   ↓
Larger Switch Networks
   ↓
Need Redundancy
   ↓
Multiple Layer 2 Paths
   ↓
Potential Switching Loops
   ↓
SPANNING TREE PROTOCOL
```

---

# Combined Skill 12 and Skill 13 Summary

```text
VLANs
   ↓
Separate Broadcast Domains
   ↓
Access Ports
   ↓
Connect End Devices to VLANs
   ↓
Trunks
   ↓
Carry Multiple VLANs
   ↓
Inter-VLAN Routing
   ↓
Allow Controlled Communication
   ↓
DTP / VTP / Native VLAN
   ↓
Control and Secure VLAN Operation
   ↓
Build Larger Switched Networks
   ↓
Add Redundant Links
   ↓
Potential Layer 2 Loops
   ↓
Broadcast Storms
   ↓
STP
   ↓
Block Redundant Paths
   ↓
Maintain Loop-Free Topology
   ↓
Keep Backup Paths Available
```

# Key CCNA Exam Points

* A VLAN creates a separate Layer 2 broadcast domain.
* Access ports normally carry traffic for one VLAN.
* Trunk ports carry traffic for multiple VLANs.
* 802.1Q is used for VLAN tagging.
* Native VLAN traffic is untagged by default.
* Different VLANs require Layer 3 routing to communicate.
* DTP dynamically negotiates trunking.
* VTP distributes VLAN information between Cisco switches.
* Redundant Layer 2 links can create switching loops.
* Ethernet frames do not have a TTL field.
* Switching loops can cause broadcast storms.
* STP prevents Layer 2 loops.
* STP logically blocks redundant paths.
* Blocked redundant paths remain available as backups.
* Classic STP is IEEE 802.1D.
* STP first elects a Root Bridge.
* Non-root switches select a Root Port.
* Network segments select a Designated Port.
* STP provides loop prevention while maintaining network redundancy.
