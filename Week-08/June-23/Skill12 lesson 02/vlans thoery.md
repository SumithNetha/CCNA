# Skill 12 - VLANs

## Lesson 01 - Why We Need VLANs

### What is a VLAN?

A VLAN (Virtual Local Area Network) is a logical segmentation of a Layer 2 network that divides a single physical switch into multiple independent broadcast domains.

- One physical switch can function as multiple logical switches.
- Devices inside one VLAN communicate at Layer 2.
- Devices in different VLANs require Layer 3 routing.

---

## Why VLANs Exist

Without VLANs:

- One broadcast domain
- One IP subnet
- Large amount of broadcast traffic
- Poor scalability
- Weak security

VLANs solve these problems by logically separating devices.

---

## Broadcast Domain

A Broadcast Domain is the area where broadcast traffic can travel.

Without VLANs

```
One Switch

PC
Laptop
Printer
Camera
Server
Phone
```

All devices receive broadcasts.

With VLANs

```
Switch

VLAN 10
PC
Printer

----------------

VLAN 20
Laptop
Phone
```

Broadcast traffic stays inside its VLAN.

---

## Benefits of VLANs

### Scalability

- Create new logical networks without buying new switches.
- Easy expansion for departments or services.

### Security

Separate:

- Employee PCs
- Guest Wi-Fi
- Cameras
- IP Phones
- Servers
- Network Management

Compromise in one VLAN does not automatically affect another.

---

## VLAN Across Multiple Switches

VLANs can span multiple switches using trunk links.

Trunk links carry traffic for multiple VLANs using IEEE 802.1Q tagging.

---

## Inter-VLAN Routing

Different VLANs cannot communicate directly.

A Layer 3 device is required.

Examples:

- Router
- Layer 3 Switch

One common CCNA solution is Router-on-a-Stick.

---

# Lesson 02 - Creating and Naming VLANs

## Default VLAN

Every Cisco switch contains VLAN 1 by default.

Initially:

- All switch ports belong to VLAN 1.
- VLAN support is already enabled.

---

## Switch Virtual Interface (SVI)

An SVI is a virtual interface associated with a VLAN.

Used for:

- Switch management
- Remote administration

Example

```
interface vlan 1
ip address 10.0.18.2 255.255.255.192
```

---

## Creating VLANs

```
vlan 10
name ADMIN-DEVICES

vlan 20
name PATRON-DEVICES
```

Creating a VLAN does NOT move interfaces into it.

The VLAN exists but contains no ports until assigned.

---

## Naming VLANs

Use meaningful names instead of only numbers.

Example

```
VLAN 10 ADMIN-DEVICES

VLAN 20 PATRON-DEVICES
```

Benefits

- Easier troubleshooting
- Better documentation
- Easier administration

---

## Access Ports

Configure interfaces connected to end devices.

```
switchport mode access
```

Assign VLAN

```
switchport access vlan 20
```

---

## Interface Range

Configure multiple interfaces simultaneously.

Example

```
interface range fa0/10-15
```

Useful when configuring many access ports.

---

## Dynamic Ports

Older Cisco switches use Dynamic Auto or Dynamic Desirable.

Avoid using them because they may negotiate trunk links automatically.

Best Practice:

- User devices → Access Mode
- Switches → Trunk Mode

---

## Verification Commands

```
show vlan
show vlan brief
show ip interface brief
show interfaces switchport
show interfaces trunk
show running-config
```

---

## Important Cisco Commands

Create VLAN

```
vlan <vlan-id>
```

Name VLAN

```
name <vlan-name>
```

Configure Interface

```
interface fa0/10
```

Configure Multiple Interfaces

```
interface range fa0/10-15
```

Configure Access Mode

```
switchport mode access
```

Assign VLAN

```
switchport access vlan 20
```

Save Configuration

```
write memory
```

---

## CCNA Exam Points

- VLAN = Virtual Local Area Network.
- VLAN 1 is the default VLAN.
- One VLAN equals one Broadcast Domain.
- One VLAN generally equals one IP subnet.
- VLANs improve scalability and security.
- Access Ports carry one VLAN.
- Trunk Ports carry multiple VLANs.
- IEEE 802.1Q is the VLAN tagging standard.
- Inter-VLAN communication requires Layer 3 routing.
- Avoid Dynamic switchport modes on production access ports.