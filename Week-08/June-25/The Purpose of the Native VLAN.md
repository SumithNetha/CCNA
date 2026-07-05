# Skill 12 Lesson 06 – The Purpose of the Native VLAN

## Overview

A **Native VLAN** is the VLAN assigned to **untagged Ethernet frames** received on an **802.1Q trunk port**.

When traffic crosses a trunk link, most Ethernet frames contain an **802.1Q VLAN tag** that identifies the VLAN to which they belong. If a frame arrives **without a VLAN tag**, the receiving switch places it into the configured **Native VLAN**.

The Native VLAN acts as the **default VLAN for untagged traffic** on a trunk.

---

# Why Do We Need a Native VLAN?

A trunk link carries traffic for multiple VLANs over a single physical connection.

Example:

```
VLAN 10
VLAN 20
VLAN 30

↓

Single Trunk Link

↓

Another Switch
```

Normally every frame contains a VLAN tag.

Example:

```
Ethernet Frame

↓

802.1Q VLAN Tag

↓

Payload
```

The receiving switch reads the VLAN tag and forwards the frame to the correct VLAN.

However, if a frame arrives without a VLAN tag, the switch must still determine which VLAN the frame belongs to.

The answer is the **Native VLAN**.

---

# Definition

A **Native VLAN** is the VLAN that receives **all untagged traffic** arriving on an **802.1Q trunk port**.

Think of it as the **default VLAN** for untagged frames.

---

# Tagged vs Untagged Frames

## Tagged Frame

A tagged frame contains VLAN information.

```
PC (VLAN 20)

↓

Switch

↓

Frame + VLAN Tag (20)

↓

Trunk

↓

Switch
```

The switch immediately knows that the frame belongs to VLAN 20.

---

## Untagged Frame

An untagged frame contains no VLAN information.

```
Ethernet Frame

(No VLAN Tag)

↓

Switch

↓

Native VLAN
```

The switch assigns the frame to the configured Native VLAN.

---

# Historical Background

Before switches became common, networks used **Ethernet hubs**.

A hub:

- Does not understand VLANs.
- Does not understand MAC addresses.
- Cannot insert VLAN tags.
- Simply repeats electrical signals to every port.

Example:

```
PC

↓

Hub

↓

Switch Trunk Port
```

Because hubs transmitted untagged traffic, switches needed a default VLAN for these frames.

Cisco introduced the **Native VLAN** for this purpose.

---

# Why Native VLAN Still Exists

Although hubs are almost obsolete, the Native VLAN remains important because modern infrastructure devices still generate or receive untagged management traffic.

Examples include:

- Cisco Switches
- Wireless Access Points
- Hypervisors
- VMware ESXi Hosts
- Firewalls
- Infrastructure Appliances

---

# Native VLAN and Trunk Ports

The Native VLAN is only relevant on **trunk ports**.

### Access Port

```
PC

↓

Access Port

↓

VLAN 10
```

An access port belongs to only one VLAN, so every frame automatically belongs to that VLAN.

No Native VLAN is involved.

---

### Trunk Port

```
VLAN 10
VLAN 20
VLAN 30

↓

Trunk

↓

Switch
```

Since multiple VLANs share one cable, VLAN tags are required.

If a frame is missing a tag, the switch assigns it to the Native VLAN.

---

# Native VLAN in Virtualization

A virtualization server hosts several virtual machines.

Example:

```
VM1 → VLAN 10

VM2 → VLAN 20

VM3 → VLAN 30
```

All virtual machines communicate through one physical network adapter connected to a trunk.

```
Cisco Switch

↓

Trunk

↓

VMware ESXi Host
```

The virtual machines send tagged traffic.

However, the physical ESXi host also requires a management interface.

Many deployments use the Native VLAN to carry this untagged management traffic.

Example:

```
ESXi Management

↓

Untagged Frame

↓

Native VLAN

↓

Switch
```

The switch places the untagged frame into the configured Native VLAN.

---

# Native VLAN with Wireless Access Points

A wireless access point can provide multiple SSIDs.

Example:

```
Guest Wi-Fi

↓

VLAN 30
```

```
Corporate Wi-Fi

↓

VLAN 20
```

Both VLANs travel across the same trunk.

The access point itself still needs a management connection.

Administrators use this connection to:

- Configure the AP
- Upgrade firmware
- Monitor performance
- Troubleshoot issues

This management traffic is commonly carried through the Native VLAN.

---

# Management VLAN

A **Management VLAN** is a dedicated VLAN used only for managing network devices.

Typical devices include:

- Switches
- Routers
- Firewalls
- Wireless Access Points
- Wireless Controllers
- Hypervisors
- Monitoring Servers

Example:

```
VLAN 10 → Sales

VLAN 20 → HR

VLAN 30 → Guest

VLAN 99 → Management
```

Only administrators should have access to the Management VLAN.

---

# Native VLAN vs Management VLAN

These two terms are often confused.

## Native VLAN

Purpose:

- Carries untagged traffic on trunk ports.

## Management VLAN

Purpose:

- Carries administrative traffic used to manage infrastructure devices.

They are **not the same thing**, but they are often configured to use the same VLAN ID.

Example:

```
Native VLAN = VLAN 99

Management VLAN = VLAN 99
```

This is a common enterprise design.

They can also be different.

Example:

```
Native VLAN = VLAN 999

Management VLAN = VLAN 99
```

Both configurations are valid.

---

# Why Use a Separate Management VLAN?

Separating management traffic provides several benefits.

### Better Security

Users cannot directly access network devices.

Instead of:

```
Employee PCs

↓

Switch Management
```

Use:

```
Administrator PC

↓

Management VLAN

↓

Switch
```

---

### Easier Administration

All infrastructure devices are located in one network.

This simplifies:

- SSH
- SNMP
- Syslog
- Configuration Backups
- Device Monitoring

---

### Smaller Attack Surface

If an employee computer becomes infected,

```
Virus

↓

User VLAN

↓

Cannot Reach

↓

Management VLAN
```

The attacker cannot directly access switches and routers.

---

# Default Native VLAN

Cisco switches use:

```
VLAN 1
```

as the default Native VLAN.

Although functional, VLAN 1 is not recommended for production because:

- It is the default on every Cisco switch.
- Many Cisco protocols use VLAN 1.
- Attackers commonly target VLAN 1.

Best practice:

Move the Native VLAN to another VLAN.

Example:

```
VLAN 99
```

or

```
VLAN 999
```

---

# Native VLAN Mismatch

One of the most common Layer 2 configuration problems.

Example:

Switch A

```
Native VLAN = 99
```

Switch B

```
Native VLAN = 1
```

Connected by a trunk:

```
Switch A

Native VLAN 99

====================

Native VLAN 1

Switch B
```

An untagged frame leaves Switch A.

Switch A believes:

```
VLAN 99
```

Switch B believes:

```
VLAN 1
```

The same frame is placed into different VLANs.

---

# Problems Caused by Native VLAN Mismatch

- Incorrect VLAN assignment
- Broadcast traffic leakage
- VLAN isolation failure
- Security vulnerabilities
- Management communication failures
- Difficult troubleshooting

---

# Cisco Warning Message

Cisco switches often detect this problem automatically.

Typical message:

```text
%CDP-4-NATIVE_VLAN_MISMATCH
```

This indicates that both sides of the trunk are configured with different Native VLANs.

---

# VLAN and Subnet Relationship

Enterprise networks typically follow:

```
One VLAN

↓

One IP Subnet
```

Example:

| VLAN | Network | Purpose |
|------|------------------|----------------|
| 10 | 192.168.10.0/24 | Sales |
| 20 | 192.168.20.0/24 | HR |
| 30 | 192.168.30.0/24 | Guest |
| 99 | 192.168.99.0/24 | Management |

This simplifies routing, security, and troubleshooting.

---

# Cisco Configuration

## Configure Trunk

```cisco
interface GigabitEthernet0/1
 switchport mode trunk
```

---

## Configure Native VLAN

```cisco
interface GigabitEthernet0/1
 switchport trunk native vlan 99
```

---

## Create Management VLAN

```cisco
vlan 99
 name MANAGEMENT
```

---

## Configure Switch Management Interface

```cisco
interface vlan 99
 ip address 192.168.99.2 255.255.255.0
 no shutdown
```

---

## Configure Default Gateway

```cisco
ip default-gateway 192.168.99.1
```

---

## Verify VLANs

```cisco
show vlan brief
```

---

## Verify Trunks

```cisco
show interfaces trunk
```

Displays:

- Trunk Ports
- Native VLAN
- Allowed VLANs
- Trunk Status

---

# Enterprise Best Practices

- Configure the same Native VLAN on both sides of every trunk.
- Avoid using VLAN 1 as the Native VLAN.
- Use a dedicated Management VLAN.
- Restrict Management VLAN access using ACLs and firewalls.
- Document Native VLAN assignments.
- Verify trunk configuration after changes.
- Use `show interfaces trunk` regularly.

---

# CCNA Exam Points

- Native VLAN carries **untagged traffic** on an **802.1Q trunk**.
- Default Native VLAN is **VLAN 1**.
- Native VLAN only applies to **trunk ports**, not access ports.
- Tagged frames keep their VLAN ID.
- Untagged frames are assigned to the Native VLAN.
- Native VLAN is commonly used for infrastructure management.
- Native VLAN and Management VLAN are different concepts but may use the same VLAN ID.
- Native VLAN mismatches occur when each end of a trunk uses a different Native VLAN.
- Configure the Native VLAN using:

```cisco
switchport trunk native vlan <vlan-id>
```

- Verify trunk configuration using:

```cisco
show interfaces trunk
```

---

# Key Takeaways

- The Native VLAN is the default VLAN for all untagged traffic received on an 802.1Q trunk.
- It was originally introduced to support legacy devices such as Ethernet hubs but remains important in modern networks.
- Many enterprise environments use the Native VLAN for infrastructure management, although the Native VLAN and Management VLAN are separate concepts.
- Native VLAN mismatches can lead to VLAN leakage, incorrect traffic forwarding, and security issues.
- Best practice is to use a dedicated Native VLAN (not VLAN 1), configure the same Native VLAN on both ends of every trunk, and protect the Management VLAN with appropriate security controls.