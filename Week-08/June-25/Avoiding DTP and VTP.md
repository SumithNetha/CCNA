# Skill 12 Lesson 05 – Avoiding DTP and VTP

## Overview

DTP (Dynamic Trunking Protocol) and VTP (VLAN Trunking Protocol) are Cisco proprietary Layer 2 protocols designed to automate switch configuration.

- **DTP** automates trunk link negotiation.
- **VTP** automates VLAN database synchronization.

Although these features reduce manual configuration, they introduce security risks and unpredictable behavior. Modern enterprise networks generally disable both protocols and use manual configuration.

---

# Dynamic Trunking Protocol (DTP)

## Definition

Dynamic Trunking Protocol (DTP) is a Cisco proprietary protocol that allows two Cisco switches to automatically negotiate whether a switch port should become a trunk.

Instead of manually configuring trunk ports, switches exchange DTP messages to determine if trunking should be established.

---

## Purpose

- Automatically negotiates trunk links.
- Reduces manual configuration.
- Works only between Cisco devices.

---

## DTP Port Modes

### Access

- Always operates as an access port.
- Carries traffic for only one VLAN.
- Never becomes a trunk.

```cisco
switchport mode access
```

### Trunk

- Always operates as a trunk.
- Carries multiple VLANs.
- No negotiation required.

```cisco
switchport mode trunk
```

### Dynamic Auto

- Passive mode.
- Waits for another device to request trunking.
- Does not initiate negotiation.

### Dynamic Desirable

- Active mode.
- Initiates DTP negotiation.
- Attempts to form a trunk.

---

## DTP Negotiation Results

| Side A | Side B | Result |
|---------|---------|--------|
| Auto | Auto | No trunk |
| Auto | Desirable | Trunk |
| Desirable | Auto | Trunk |
| Desirable | Desirable | Trunk |
| Trunk | Auto | Trunk |
| Trunk | Desirable | Trunk |
| Trunk | Trunk | Trunk |
| Access | Any | Access |

---

## Easy Memory Trick

- Access → Never trunk
- Trunk → Always trunk
- Auto → Passive
- Desirable → Active

---

## Security Risk

Leaving DTP enabled allows another device to negotiate a trunk connection.

Example:

```
Attacker Switch
       │
Negotiates DTP
       │
Trunk Created
       │
Access to Multiple VLANs
```

This can lead to a **VLAN Hopping Attack**, where an attacker gains access to VLANs they should never reach.

---

## Disable DTP

Disable DTP negotiation while keeping the interface manually configured.

```cisco
interface GigabitEthernet0/1
 switchport mode trunk
 switchport nonegotiate
```

### Benefits

- Prevents automatic negotiation.
- Improves security.
- Makes network behavior predictable.
- Simplifies troubleshooting.

---

# VLAN Trunking Protocol (VTP)

## Definition

VLAN Trunking Protocol (VTP) is a Cisco proprietary protocol used to distribute VLAN information between Cisco switches.

> **Important:** VTP does **not** create trunk links.

802.1Q performs trunking, while VTP only synchronizes VLAN information.

---

## Purpose

Without VTP:

Every switch must have VLANs created manually.

With VTP:

Create VLAN once, and every switch in the VTP domain automatically learns it.

---

## How VTP Works

```
Create VLAN 20

↓

VTP Advertisement

↓

Other Switches

↓

Automatically Create VLAN 20
```

---

# Risks of VTP

Although convenient, VTP can also replicate mistakes.

If a VLAN is deleted on a VTP Server:

```
Delete VLAN

↓

VTP Advertisement

↓

Every Switch Deletes VLAN
```

One incorrect configuration can affect the entire network.

---

## Common VTP Problems

### 1. VLAN Deletion

Deleting VLANs on a VTP Server propagates those deletions to every switch.

Result:

- VLANs disappear.
- Users lose connectivity.
- Large network outages.

---

### 2. Old Lab Switch

An old switch with an outdated VLAN database is connected to production.

```
Old Switch

↓

VTP Synchronization

↓

Wrong VLAN Database

↓

Production Network Issues
```

---

# VTP Modes

## Server

- Can create VLANs.
- Can delete VLANs.
- Advertises VLAN updates.
- Stores VLAN database.

---

## Client

- Cannot create VLANs.
- Cannot delete VLANs.
- Learns VLANs from Server.
- Automatically synchronizes.

---

## Transparent

- Maintains its own VLAN database.
- Does not apply VTP updates locally.
- VLANs are managed manually.

Recommended mode for most enterprise networks.

```cisco
vtp mode transparent
```

---

# VTP Domain

A VTP Domain identifies which switches exchange VLAN information.

Example:

```
COMPANY
```

Only switches in the same domain synchronize VLAN information.

### Case Sensitive

```
COMPANY

≠

company
```

---

# DTP vs VTP

| DTP | VTP |
|------|------|
| Dynamic Trunking Protocol | VLAN Trunking Protocol |
| Negotiates trunk links | Synchronizes VLAN database |
| Interface configuration | VLAN configuration |
| Cisco proprietary | Cisco proprietary |

---

# Important Cisco Commands

## Configure Access Port

```cisco
interface GigabitEthernet0/1
 switchport mode access
```

---

## Configure Trunk Port

```cisco
interface GigabitEthernet0/1
 switchport mode trunk
```

---

## Disable DTP

```cisco
interface GigabitEthernet0/1
 switchport nonegotiate
```

---

## Configure VTP Transparent Mode

```cisco
Switch(config)#vtp mode transparent
```

---

## Verify VTP Status

```cisco
show vtp status
```

Displays:

- VTP Mode
- Domain Name
- Version
- Configuration Revision
- VLAN Count

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

### DTP

- Configure trunk ports manually.
- Configure access ports manually.
- Disable DTP using `switchport nonegotiate`.
- Avoid Dynamic Auto and Dynamic Desirable.

### VTP

- Use `vtp mode transparent`.
- Create VLANs manually.
- Avoid VTP Server Mode unless required.
- Verify VTP domain before connecting switches.
- Never connect unknown lab switches to production.

---

# CCNA Exam Points

- DTP negotiates trunk links.
- VTP synchronizes VLAN databases.
- DTP and VTP are Cisco proprietary.
- Auto + Auto = No trunk.
- Auto + Desirable = Trunk.
- Desirable + Desirable = Trunk.
- `switchport nonegotiate` disables DTP.
- `vtp mode transparent` is the recommended enterprise mode.
- VTP Server creates and advertises VLANs.
- VTP Client receives VLAN updates only.
- VTP Transparent manages VLANs locally.
- VTP domains are case-sensitive.
- IEEE 802.1Q performs VLAN tagging, not VTP.

---

# Key Takeaways

- DTP automatically negotiates trunk links but is rarely used in production because of security concerns.
- Manual trunk configuration with `switchport mode trunk` and `switchport nonegotiate` provides predictable and secure operation.
- VTP automates VLAN database synchronization but can propagate configuration mistakes across the network.
- Modern enterprise networks typically disable VTP participation by using `vtp mode transparent`.
- Manual configuration is preferred over automation for better security, easier troubleshooting, and more reliable network behavior.