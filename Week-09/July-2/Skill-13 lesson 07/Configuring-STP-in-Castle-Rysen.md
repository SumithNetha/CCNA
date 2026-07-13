# Configuring STP in Castle Rysen

## Overview

After learning how Spanning Tree Protocol (STP), Rapid STP, PortFast, and BPDU Guard work, the next step is implementing these technologies in a realistic enterprise network.

The Castle Rysen / NetworkChuck Coffee network contains multiple switches, redundant Layer 2 connections, several VLANs, and endpoint-facing access ports.

The goal of this implementation is to create a Layer 2 topology that is:

* Loop-free
* Fast-converging
* Predictable
* Redundant
* Protected from accidental switch connections
* Easy to troubleshoot

The STP implementation follows five major steps:

```text
1. Enable Rapid PVST+
        ↓
2. Select Primary and Secondary Root Bridges
        ↓
3. Verify VLAN and Trunk Consistency
        ↓
4. Configure PortFast and BPDU Guard
        ↓
5. Verify and Save the Configuration
```

---

# Network Design

The Castle Rysen network consists of two major locations used in this lab:

```text
Castle Rysen Network
        |
        +----------------------+
        |                      |
        v                      v
      Cafe              Fallout Shelter
        |                      |
        v                      v
   Cafe Switches        FO-SW01 - FO-SW06
```

The Fallout Shelter network uses the following VLANs:

| VLAN | Name     |
| ---- | -------- |
| 1    | Default  |
| 10   | MGMT     |
| 20   | INTERNAL |
| 30   | VIDEO    |
| 40   | GUEST    |

The Layer 2 topology contains redundant links between switches.

Redundancy is necessary because it provides backup paths if a connection fails.

However:

```text
Redundant Layer 2 Links
          +
No Loop Prevention
          =
Layer 2 Loop
          |
          v
Broadcast Storm
          |
          v
Network Failure
```

Therefore, STP must be intentionally designed and configured.

---

# STP Implementation Checklist

Before configuring the switches, the following implementation checklist was used:

```text
[ ] Step 1: Enable Rapid PVST+ on every switch

[ ] Step 2: Select the Primary and Secondary Root Bridges

[ ] Step 3: Verify consistent VLAN and trunk configuration

[ ] Step 4: Configure PortFast and BPDU Guard on access ports

[ ] Step 5: Verify the topology and save the configuration
```

The checklist approach is important in production environments because STP configuration affects the entire Layer 2 topology.

A missing configuration, incorrect trunk, or unexpected root bridge can change traffic paths and potentially cause outages.

---

# Step 1 — Enable Rapid PVST+

## Why Use Rapid PVST+?

Cisco switches may run traditional PVST+ depending on their existing configuration.

Traditional STP has slower convergence behavior.

The Castle Rysen network was migrated to:

```text
Rapid PVST+
```

Rapid PVST+ provides:

* Faster convergence
* Rapid STP behavior
* A separate STP instance for each VLAN
* Independent STP topology decisions per VLAN

Conceptually:

```text
VLAN 1  → Separate STP Instance

VLAN 10 → Separate STP Instance

VLAN 20 → Separate STP Instance

VLAN 30 → Separate STP Instance

VLAN 40 → Separate STP Instance
```

---

## Configuration

Rapid PVST+ is enabled globally using:

```text
configure terminal

spanning-tree mode rapid-pvst
```

This configuration must be applied consistently across the Layer 2 switching infrastructure.

---

## Cafe Implementation

Rapid PVST+ was enabled on:

```text
cafe01-sw1
cafe01-sw02
```

Configuration:

```text
configure terminal

spanning-tree mode rapid-pvst
```

---

## Fallout Shelter Implementation

The same configuration was applied to the Fallout Shelter switches:

```text
FO-SW01
FO-SW02
FO-SW03
FO-SW04
FO-SW05
FO-SW06
```

Configuration:

```text
configure terminal

spanning-tree mode rapid-pvst
```

---

## Verification

Before the migration, the STP output showed:

```text
Spanning tree enabled protocol ieee
```

After enabling Rapid PVST+:

```text
Spanning tree enabled protocol rstp
```

Example from `FO-SW03`:

```text
VLAN0001
  Spanning tree enabled protocol rstp
```

The same change was observed on other Fallout Shelter switches.

---

## What the Verification Proves

```text
BEFORE

show spanning-tree
        |
        v
protocol ieee
        |
        v
Traditional STP


AFTER

spanning-tree mode rapid-pvst
        |
        v
show spanning-tree
        |
        v
protocol rstp
        |
        v
Rapid STP Operational
```

---

## Lab Observation: Topology Reconvergence

While changing the STP mode on `FO-SW05`, the output temporarily showed:

```text
Fa0/1    Desg BLK
Fa0/2    Root LIS
```

This occurred while the STP topology was reconverging.

The important lesson is:

> Immediately after changing STP configuration, the topology may temporarily transition through different port roles and states.

Always allow convergence to complete before making final conclusions from verification output.

---

# Step 2 — Select the Root Bridge

## Why Manually Select the Root Bridge?

STP automatically elects a Root Bridge.

However, allowing STP to choose the root based purely on default Bridge IDs can create an unpredictable network design.

The switch with the lowest Bridge ID wins the election.

The Bridge ID contains:

```text
Bridge Priority
      +
Extended System ID
      +
MAC Address
```

If all switches use the default priority:

```text
32768
```

the switch with the lowest MAC address may become the Root Bridge.

That means the network topology could be determined by hardware MAC addresses rather than intentional network architecture.

In an enterprise network:

> The Root Bridge should be deliberately selected.

---

# Selecting FO-SW01 as the Primary Root Bridge

The network topology was examined to determine which switch should become the Root Bridge.

`FO-SW01` was selected because of its central position in the Fallout Shelter Layer 2 topology.

The following VLANs were configured:

```text
VLAN 1
VLAN 10
VLAN 20
VLAN 30
VLAN 40
```

Configuration:

```text
configure terminal

spanning-tree vlan 1,10,20,30,40 priority 4096
```

This gives FO-SW01 a lower STP priority than the other switches.

Since:

```text
Lower Bridge ID = Better
```

FO-SW01 wins the Root Bridge election.

---

# Selecting FO-SW02 as the Secondary Root Bridge

A production STP design should also have a predictable backup Root Bridge.

Therefore, FO-SW02 was configured with:

```text
spanning-tree vlan 1,10,20,30,40 priority 8192
```

The resulting hierarchy is:

```text
FO-SW01
Priority 4096
PRIMARY ROOT
      |
      v
FO-SW02
Priority 8192
SECONDARY ROOT
      |
      v
Other Switches
Priority 32768
```

If FO-SW01 fails:

```text
FO-SW01 Failure
       |
       v
New STP Election
       |
       v
FO-SW02
       |
       v
Becomes Root Bridge
```

This makes the topology predictable.

---

# Alternative Root Bridge Commands

Cisco also provides:

```text
spanning-tree vlan 1,10,20,30,40 root primary
```

and:

```text
spanning-tree vlan 1,10,20,30,40 root secondary
```

Both approaches are valid.

In this lab, explicit priority values were used.

---

# Verifying FO-SW01 as the Root Bridge

The command used was:

```text
show spanning-tree
```

For VLAN 1:

```text
VLAN0001

Root ID    Priority    4097
           Address     00D0.5826.B7C5
           This bridge is the root

Bridge ID  Priority    4097
           Address     00D0.5826.B7C5
```

The most important line is:

```text
This bridge is the root
```

Also:

```text
Root ID MAC Address
        =
Bridge ID MAC Address
```

Therefore:

```text
FO-SW01 = Root Bridge
```

---

# Understanding the Displayed STP Priority

FO-SW01 was configured with:

```text
priority 4096
```

However, the output for VLAN 1 displayed:

```text
Priority 4097
```

This occurs because Cisco uses the Extended System ID.

The displayed priority is:

```text
Configured Bridge Priority + VLAN ID
```

Therefore:

| VLAN | Configured Priority | VLAN ID | Displayed Priority |
| ---- | ------------------: | ------: | -----------------: |
| 1    |                4096 |       1 |               4097 |
| 10   |                4096 |      10 |               4106 |
| 20   |                4096 |      20 |               4116 |
| 30   |                4096 |      30 |               4126 |
| 40   |                4096 |      40 |               4136 |

This exactly matched the lab output.

---

# Why FO-SW01 Has No Root Port

The output showed all operational FO-SW01 ports as:

```text
Desg FWD
```

For example:

```text
Fa0/1    Desg FWD
Fa0/2    Desg FWD
Fa0/3    Desg FWD
Fa0/4    Desg FWD
Fa0/5    Desg FWD
Fa0/6    Desg FWD
Fa0/7    Desg FWD
```

Where:

```text
Desg = Designated Port

FWD = Forwarding
```

The Root Bridge does not have a Root Port.

Why?

A Root Port is:

> The best port on a non-root switch toward the Root Bridge.

FO-SW01 is already the Root Bridge.

Therefore:

```text
ROOT BRIDGE

No Root Port
     +
All Operational Ports
are Designated Ports
```

The output is consistent with correct STP behavior.

---

# Step 3 — Verify VLAN and Trunk Consistency

## Why This Step Matters

Rapid PVST+ creates a separate STP instance for every VLAN.

Therefore, inconsistent VLAN or trunk configuration can cause unexpected Layer 2 behavior.

For example:

```text
SW1
VLANs 1,10,20,30,40
          |
          | TRUNK
          |
SW2
VLANs 1,10,20,30
```

VLAN 40 is missing from SW2.

The topology may appear correct for other VLANs while VLAN 40 has a configuration problem.

Therefore:

> Never assume that a correct VLAN 1 STP topology means every VLAN is correctly configured.

---

# Verification Commands

The primary commands used were:

```text
show vlan
```

and:

```text
show interfaces trunk
```

---

# VLAN Verification on FO-SW01

The output showed:

```text
1    default      active

10   MGMT         active

20   INTERNAL     active

30   VIDEO        active

40   GUEST        active
```

Therefore, all required VLANs existed and were active.

```text
VLAN 1   ✓

VLAN 10  ✓

VLAN 20  ✓

VLAN 30  ✓

VLAN 40  ✓
```

---

# Understanding Why Trunk Ports Do Not Appear Under `show vlan`

The `show vlan` output showed ports such as:

```text
Fa0/8
Fa0/9
Fa0/10
...
Fa0/24
```

However, Fa0/1 through Fa0/7 were not listed as VLAN 1 access ports.

This is expected because those interfaces were operating as trunks.

Use:

```text
show vlan
```

primarily to inspect VLAN existence and access-port membership.

Use:

```text
show interfaces trunk
```

to inspect trunk interfaces.

---

# Trunk Verification on FO-SW01

The output showed:

```text
Port        Mode         Encapsulation  Status

Fa0/1       on           802.1q         trunking

Fa0/2       desirable    n-802.1q       trunking

Fa0/3       desirable    n-802.1q       trunking

Fa0/4       desirable    n-802.1q       trunking

Fa0/5       desirable    n-802.1q       trunking

Fa0/6       desirable    n-802.1q       trunking

Fa0/7       desirable    n-802.1q       trunking
```

All seven interfaces were operational trunks.

---

# Static Trunk vs Dynamic Desirable

An important configuration difference was discovered.

Fa0/1 showed:

```text
Mode: on
```

This means it was statically configured as a trunk:

```text
switchport mode trunk
```

Fa0/2 through Fa0/7 showed:

```text
Mode: desirable
```

These ports were using DTP dynamic trunk negotiation.

Conceptually:

```text
Fa0/1
   |
   v
Static Trunk


Fa0/2-7
   |
   v
Dynamic Desirable
   |
   v
DTP Negotiation
   |
   v
Trunk
```

Operationally, the trunks were functioning.

However, enterprise environments generally benefit from predictable, explicit switch configuration.

For intentional switch-to-switch trunks, a cleaner design is:

```text
interface range fa0/1-7

switchport mode trunk

switchport nonegotiate
```

This disables unnecessary DTP negotiation.

---

# Allowed and Active VLANs

The output showed:

```text
Vlans allowed on trunk

1-1005
```

This means all normal VLANs were permitted by the trunk configuration.

The next section showed:

```text
Vlans allowed and active in management domain

1,10,20,30,40
```

This means only these VLANs currently existed and were active.

Conceptually:

```text
VLANs Allowed
     |
     v
1-1005
     |
     v
VLANs Existing and Active
     |
     v
1,10,20,30,40
```

---

# STP Forwarding Verification

The output showed:

```text
Vlans in spanning tree forwarding state and not pruned

Fa0/1    1,10,20,30,40
Fa0/2    1,10,20,30,40
Fa0/3    1,10,20,30,40
Fa0/4    1,10,20,30,40
Fa0/5    1,10,20,30,40
Fa0/6    1,10,20,30,40
Fa0/7    1,10,20,30,40
```

This matched the previous `show spanning-tree` output.

FO-SW01 was the Root Bridge.

Therefore, its operational STP ports were Designated Forwarding ports.

```text
FO-SW01
ROOT BRIDGE
      |
      v
Designated Ports
      |
      v
Forwarding
      |
      v
VLANs 1,10,20,30,40
Forwarding Across Trunks
```

---

# Important Verification Principle

Checking only one switch does not prove that the entire Layer 2 topology is consistent.

The proper workflow is:

```text
FO-SW01
   |
   v
show vlan
show interfaces trunk
   |
   v
FO-SW02
   |
   v
show vlan
show interfaces trunk
   |
   v
FO-SW03
   |
   v
Continue Across All Switches
   |
   v
Compare Results
```

STP verification should always be topology-wide.

---

# Step 4 — Configure PortFast and BPDU Guard

## Why Configure Edge Ports?

Ports connected to end devices do not normally need to participate in the complete STP convergence process.

Examples include:

* PCs
* Printers
* Cameras
* Servers
* Other endpoint devices

Two features were configured:

```text
PortFast
    +
BPDU Guard
```

PortFast provides fast forwarding behavior.

BPDU Guard protects the network from unexpected switches.

---

# PortFast

PortFast allows an endpoint-facing port to transition rapidly to the forwarding state.

Conceptually:

```text
WITHOUT PORTFAST

Endpoint Connected
       |
       v
STP Transition
       |
       v
Forwarding


WITH PORTFAST

Endpoint Connected
       |
       v
Immediate Forwarding
```

---

# BPDU Guard

BPDU Guard protects endpoint-facing ports from unexpected BPDUs.

```text
Access Port
     |
     v
BPDU Guard Enabled
     |
     v
BPDU Received?
    / \
   /   \
 NO     YES
 |       |
 v       v
Stay    Err-Disabled
Up
```

PortFast and BPDU Guard complement each other:

```text
PORTFAST
    +
BPDU GUARD
    =
FAST + PROTECTED EDGE PORT
```

---

# Fallout Shelter Implementation

The `show spanning-tree` output on switches such as FO-SW06 showed:

```text
Fa0/1    Root FWD

Fa0/2    Altn BLK
```

These were infrastructure links participating in the STP topology.

Therefore, they were excluded from the edge-port configuration.

The selected interface range was:

```text
Fa0/3-24
```

Configuration:

```text
configure terminal

interface range fa0/3-24

switchport mode access

spanning-tree portfast

spanning-tree bpduguard enable
```

This configuration was applied to:

```text
FO-SW03
FO-SW04
FO-SW05
FO-SW06
```

The resulting design was:

```text
             FALLOUT SHELTER SWITCH

         Fa0/1              Fa0/2
           |                   |
           v                   v
      STP Uplink          STP Uplink

         Root /          Alternate /
       Designated          Designated

                 Fa0/3-24
                     |
                     v
                Access Ports
                     |
             +-------+-------+
             |               |
             v               v
         PortFast        BPDU Guard
```

---

# Cafe Implementation

The Cafe switches required different interface ranges because their existing trunk configuration was different.

---

## Cafe01-SW1

The existing configuration showed:

```text
Fa0/1  → Trunk

Fa0/2  → Trunk

Fa0/24 → Trunk
```

Therefore, the endpoint range was:

```text
Fa0/3-23
```

Configuration:

```text
configure terminal

interface range fa0/3-23

switchport mode access

spanning-tree portfast

spanning-tree bpduguard enable
```

Final design:

```text
Cafe01-SW1

Fa0/1     → Trunk

Fa0/2     → Trunk

Fa0/3-23  → Access + PortFast + BPDU Guard

Fa0/24    → Trunk
```

---

## Cafe01-SW02

The existing configuration showed:

```text
Fa0/1 → Trunk

Fa0/2 → Trunk
```

Therefore:

```text
configure terminal

interface range fa0/3-24

switchport mode access

spanning-tree portfast

spanning-tree bpduguard enable
```

Final design:

```text
Cafe01-SW02

Fa0/1-2   → Trunks

Fa0/3-24  → Access + PortFast + BPDU Guard
```

---

# Lab Troubleshooting and Lessons Learned

## 1. `interface` vs `interface range`

An incorrect command was entered:

```text
interface fa0/3-24
```

IOS returned:

```text
% Invalid input detected
```

The reason is that:

```text
interface
```

selects one interface.

To select multiple interfaces, use:

```text
interface range fa0/3-24
```

The abbreviated form also works:

```text
int ran fa0/3-24
```

Mental model:

```text
ONE INTERFACE

interface fa0/3


MULTIPLE INTERFACES

interface range fa0/3-24
```

---

## 2. PortFast Warning

PortFast was initially configured before explicitly setting the interfaces to access mode:

```text
spanning-tree portfast
```

IOS displayed:

```text
Portfast has been configured on FastEthernet0/3
but will only have effect when the interface is
in a non-trunking mode.
```

This was a warning, not a configuration failure.

PortFast was accepted.

The interfaces were later configured with:

```text
switchport mode access
```

A cleaner command order is:

```text
interface range fa0/3-24

switchport mode access

spanning-tree portfast

spanning-tree bpduguard enable
```

Mental model:

```text
DEFINE PORT ROLE
        |
        v
ACCESS PORT
        |
        v
ENABLE PORTFAST
        |
        v
ENABLE BPDU GUARD
```

---

## 3. Do Not Blindly Configure Every Interface

The Cafe lab demonstrated why the existing topology must be examined before applying interface ranges.

On `Cafe01-SW1`:

```text
Fa0/24
```

was already a trunk.

Therefore:

```text
interface range fa0/3-24
```

would have incorrectly included the trunk interface.

Instead:

```text
interface range fa0/3-23
```

was used.

This is an important real-world lesson:

> Never blindly paste an interface-range configuration without understanding what is connected to those interfaces.

---

## 4. PortFast and BPDU Guard Should Not Be Applied to Normal Infrastructure Links

Switch-to-switch links receive BPDUs as part of normal STP operation.

For example:

```text
SW1 ================= SW2

       BPDU Exchange
```

Applying BPDU Guard to such a link could cause the interface to become err-disabled when a BPDU is received.

Therefore:

```text
INFRASTRUCTURE LINK

Normal STP Participation


EDGE PORT

PortFast + BPDU Guard
```

---

## 5. Access Mode Does Not Assign the Business VLAN

The command:

```text
switchport mode access
```

defines the port as an access port.

It does not assign the interface to VLAN 10, 20, 30, or 40.

VLAN assignment requires:

```text
switchport access vlan <VLAN-ID>
```

For example:

```text
interface fa0/3

switchport mode access

switchport access vlan 20
```

The distinction is:

```text
switchport mode access
        |
        v
PORT TYPE


switchport access vlan 20
        |
        v
VLAN MEMBERSHIP
```

---

## 6. Avoid Pasting CLI Output as Commands

During the lab, console output and another switch's prompt were accidentally pasted into the Cisco CLI.

Example:

```text
FO-SW06(config-if-range)#spanning-tree bpduguard en
```

was pasted while already at another switch's configuration prompt.

IOS returned:

```text
% Invalid input detected
```

The same occurred when PortFast warning text was accidentally pasted as input.

The switch rejected these invalid commands, so no configuration damage occurred.

The operational lesson is:

> When copying commands between switches, copy only the actual commands—not device prompts, output, or error messages.

---

# Step 5 — Verify and Save

After configuration, the complete STP topology must be verified.

Useful commands include:

```text
show spanning-tree
```

```text
show vlan
```

```text
show vlan brief
```

```text
show interfaces trunk
```

```text
show running-config
```

---

# Final STP Verification

The final `show spanning-tree` output on FO-SW01 confirmed:

```text
Spanning tree enabled protocol rstp
```

Therefore:

```text
Rapid PVST+ ✓
```

The output also showed:

```text
This bridge is the root
```

for:

```text
VLAN 1
VLAN 10
VLAN 20
VLAN 30
VLAN 40
```

Therefore:

```text
FO-SW01 Primary Root Bridge ✓
```

All operational FO-SW01 interfaces showed:

```text
Desg FWD
```

which is consistent with Root Bridge behavior.

---

# Save the Configuration

After verification, the configuration was saved:

```text
copy running-config startup-config
```

Abbreviated:

```text
copy run start
```

The reason is:

```text
running-config
       |
       v
Stored in RAM
       |
       v
Lost After Reboot


startup-config
       |
       v
Stored in NVRAM
       |
       v
Loaded After Reboot
```

Therefore:

```text
VERIFY
   |
   v
CONFIGURATION CORRECT?
   |
   +------ NO ------> TROUBLESHOOT
   |
  YES
   |
   v
SAVE CONFIGURATION
   |
   v
copy run start
```

---

# Complete Castle Rysen STP Workflow

```text
                    START
                      |
                      v
        Examine the Layer 2 Topology
                      |
                      v
          Enable Rapid PVST+
                      |
                      v
     spanning-tree mode rapid-pvst
                      |
                      v
         Select Primary Root Bridge
                      |
                      v
             FO-SW01 → 4096
                      |
                      v
        Select Secondary Root Bridge
                      |
                      v
             FO-SW02 → 8192
                      |
                      v
            Verify Root Election
                      |
                      v
           show spanning-tree
                      |
                      v
         Verify VLAN Configuration
                      |
                      v
               show vlan
                      |
                      v
         Verify Trunk Configuration
                      |
                      v
         show interfaces trunk
                      |
                      v
       Identify Endpoint-Facing Ports
                      |
                      v
           Configure Access Mode
                      |
                      v
            Enable PortFast
                      |
                      v
           Enable BPDU Guard
                      |
                      v
         Verify Complete STP Topology
                      |
                      v
            show spanning-tree
                      |
                      v
           Save the Configuration
                      |
                      v
              copy run start
                      |
                      v
                     DONE
```

---

# Configuration Summary

## Rapid PVST+

```text
spanning-tree mode rapid-pvst
```

---

## Primary Root Bridge

```text
spanning-tree vlan 1,10,20,30,40 priority 4096
```

---

## Secondary Root Bridge

```text
spanning-tree vlan 1,10,20,30,40 priority 8192
```

---

## Verify STP

```text
show spanning-tree
```

---

## Verify VLANs

```text
show vlan brief
```

---

## Verify Trunks

```text
show interfaces trunk
```

---

## Configure Edge Ports

```text
interface range fa0/3-24

switchport mode access

spanning-tree portfast

spanning-tree bpduguard enable
```

---

## Save Configuration

```text
copy running-config startup-config
```

---

# Key Real-World Lessons

1. **Do not allow the Root Bridge election to depend on default priorities and MAC addresses.** Intentionally select primary and secondary Root Bridges.

2. **Enable Rapid PVST+ consistently across the switching infrastructure.** Mixed or unintended STP configurations make troubleshooting more difficult.

3. **Verify STP per VLAN.** A topology that works correctly for VLAN 1 may still contain configuration problems affecting other VLANs.

4. **Verify both sides of every trunk.** VLAN and trunk consistency cannot be confirmed by checking only one switch.

5. **Use static, predictable configurations for intentional infrastructure links.** Avoid unnecessary dependence on dynamic trunk negotiation.

6. **Apply PortFast and BPDU Guard only after identifying genuine endpoint-facing ports.** Never blindly configure interface ranges.

7. **Use PortFast and BPDU Guard together on edge ports.** PortFast provides rapid connectivity; BPDU Guard protects against unexpected Layer 2 devices.

8. **Understand command output instead of only checking whether a command succeeded.** `show spanning-tree`, `show vlan`, and `show interfaces trunk` reveal whether the network is actually behaving as designed.

9. **Wait for STP convergence before making final troubleshooting conclusions.** Port roles and states can temporarily change after topology modifications.

10. **Always verify before saving.** A saved misconfiguration survives reboot just as effectively as a correct configuration.

---

# Final Takeaway

Configuring STP in Castle Rysen was not simply a matter of entering:

```text
spanning-tree mode rapid-pvst
```

The complete implementation required a deliberate Layer 2 design:

```text
Rapid PVST+
      +
Intentional Root Bridge Selection
      +
Secondary Root Bridge
      +
Consistent VLANs
      +
Correct Trunks
      +
PortFast
      +
BPDU Guard
      +
Verification
      +
Saving the Configuration
      =
PREDICTABLE, REDUNDANT, AND LOOP-FREE LAYER 2 NETWORK
```

The most important operational principle from the lab is:

> **Configure STP based on the actual network topology, verify the resulting behavior across every relevant VLAN and switch, protect only genuine edge ports, and save the configuration only after the network behaves as intended.**
