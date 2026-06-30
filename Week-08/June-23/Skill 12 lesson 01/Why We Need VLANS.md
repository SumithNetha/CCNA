# Skill 12 Lesson 01 – Why We Need VLANs

## What is a VLAN?

A **VLAN (Virtual Local Area Network)** is a logical segmentation of a Layer 2 network that divides a single physical switch into multiple independent broadcast domains.

- Physical separation is not required.
- One switch can function as multiple logical switches.
- Each VLAN is isolated from other VLANs unless Layer 3 routing is configured.

---

# Why VLANs Exist

As networks grow, connecting every device to one large Layer 2 network creates problems:

- Too many broadcast packets
- Poor scalability
- Lack of security
- Difficult network management
- Large IP subnet requirements

VLANs solve these problems by logically separating devices into different networks.

---

# Problems Without VLANs

A default switch has:

- One VLAN (VLAN 1)
- One broadcast domain
- One IP subnet

Every broadcast frame is sent to every connected device.

Example:

PC → ARP Request

↓

Entire switch receives the broadcast.

Consequences:

- Increased unnecessary traffic
- Lower network performance
- Harder troubleshooting
- Higher security risks

---

# Broadcast Domain

A **Broadcast Domain** is the area where broadcast frames are forwarded.

Examples of broadcasts:

- ARP Request
- DHCP Discover
- NetBIOS
- Service Discovery

Without VLANs:

```
One Switch
│
├── PC1
├── PC2
├── Printer
├── Phone
├── Camera
└── Server
```

Broadcasts reach every device.

With VLANs:

```
Switch

├── VLAN 10
│   ├── PC1
│   └── PC2
│
└── VLAN 20
    ├── PC3
    └── PC4
```

Broadcasts remain inside their own VLAN.

---

# VLAN Isolation

Devices inside the same VLAN communicate directly.

Devices in different VLANs **cannot communicate at Layer 2**.

Example:

```
VLAN 10 (Sales)

PC1
PC2

-----------------

VLAN 20 (HR)

PC3
PC4
```

Sales devices cannot directly communicate with HR devices.

Inter-VLAN communication requires routing.

---

# VLAN = Logical Switches

Instead of purchasing multiple physical switches,

One switch can logically become:

- Sales Switch
- HR Switch
- Finance Switch
- Guest Network

All inside the same hardware.

---

# VLANs Across Multiple Switches

VLANs are not limited to a single switch.

They can extend across multiple switches using **Trunk Links**.

Example:

```
Switch 1
     │
   Trunk
     │
Switch 2
```

A trunk carries traffic for multiple VLANs simultaneously.

---

# Access Port

An **Access Port** belongs to only one VLAN.

Characteristics:

- Connects end devices
- Carries untagged traffic
- One VLAN only

Examples:

- PC
- Printer
- IP Phone
- Camera

---

# Trunk Port

A **Trunk Port** carries traffic for multiple VLANs.

Used between:

- Switch ↔ Switch
- Switch ↔ Router
- Switch ↔ Layer 3 Switch

Traffic is identified using VLAN tags.

---

# IEEE 802.1Q

802.1Q is the IEEE standard used for VLAN tagging.

Functions:

- Inserts VLAN ID into Ethernet frames
- Allows multiple VLANs over one physical cable
- Enables interoperability between different vendors

Supported by:

- Cisco
- Dell
- Aruba
- HP
- Ubiquiti
- Juniper

---

# VLAN IDs

Every VLAN has a unique numerical identifier.

Example:

| VLAN ID | Purpose |
|----------|----------|
| 10 | Sales |
| 20 | HR |
| 30 | Voice |
| 40 | Guest |
| 50 | Servers |
| 99 | Management |

---

# One VLAN = One IP Subnet

One of the most important CCNA concepts.

```
One VLAN
      ↓
One Broadcast Domain
      ↓
One IP Subnet
```

Example:

| VLAN | Network |
|-------|----------------|
| VLAN 10 | 192.168.10.0/24 |
| VLAN 20 | 192.168.20.0/24 |
| VLAN 30 | 192.168.30.0/24 |

Each VLAN requires its own subnet and default gateway.

---

# Why VLANs Improve Scalability

Without VLANs:

Adding new device types often requires:

- New switches
- New cabling
- New infrastructure

With VLANs:

Simply create another logical network.

Example:

Before

```
VLAN 10
Employees
```

After expansion

```
VLAN 10 Employees
VLAN 20 Voice
VLAN 30 Guest
VLAN 40 Cameras
```

No new switch required.

---

# Why VLANs Improve Security

Segmentation reduces the attack surface.

Example:

Separate:

- Employee PCs
- Guest Wi-Fi
- IP Phones
- CCTV Cameras
- Servers
- Management Devices

Benefits:

- Broadcast containment
- Traffic isolation
- Easier policy enforcement
- Reduced malware spread
- Better access control

---

# Inter-VLAN Routing

Different VLANs cannot communicate directly.

A Layer 3 device is required.

Options:

- Router
- Layer 3 Switch

Traffic Flow:

```
PC
 ↓
Switch
 ↓
Router
 ↓
Switch
 ↓
Destination VLAN
```

The router decides whether communication is allowed.

---

# Router-on-a-Stick

Common CCNA topology for Inter-VLAN Routing.

```
        Router
           │
        Trunk Link
           │
        Cisco Switch
```

Router creates logical subinterfaces:

- G0/0.10 → VLAN 10
- G0/0.20 → VLAN 20
- G0/0.30 → VLAN 30

Each subinterface acts as the **Default Gateway** for its VLAN.

---

# Real-World VLAN Design

Typical enterprise VLAN layout:

| VLAN | Purpose |
|-------|----------|
| VLAN 10 | Employees |
| VLAN 20 | Voice/IP Phones |
| VLAN 30 | Guest Wi-Fi |
| VLAN 40 | Servers |
| VLAN 50 | CCTV Cameras |
| VLAN 60 | Printers |
| VLAN 99 | Network Management |

This design improves scalability, troubleshooting, and security.

---

# Advantages of VLANs

- Reduces broadcast traffic
- Creates multiple broadcast domains
- Improves network scalability
- Enhances security through segmentation
- Simplifies network management
- Supports logical departmental separation
- Enables efficient IP address planning
- Allows multiple logical networks on one physical switch
- Reduces hardware requirements
- Supports enterprise network design

---

# Important CCNA Exam Points

- VLAN = Virtual Local Area Network.
- Default Cisco switch places all ports in VLAN 1.
- One VLAN equals one Broadcast Domain.
- One VLAN typically equals one IP Subnet.
- Devices in different VLANs require Layer 3 routing.
- Access ports carry one VLAN.
- Trunk ports carry multiple VLANs.
- IEEE 802.1Q is the VLAN tagging standard.
- Router-on-a-Stick uses subinterfaces for Inter-VLAN Routing.
- VLANs improve both scalability and security.

---

# Key Terms

- VLAN
- Broadcast Domain
- Access Port
- Trunk Port
- IEEE 802.1Q
- VLAN Tagging
- VLAN ID
- Inter-VLAN Routing
- Router-on-a-Stick
- Default Gateway
- Layer 2 Segmentation
- Layer 3 Routing