# Skill 12 Lesson 03 - Using Trunks to Extend VLANs

## What is a Trunk?

A trunk is a switch interface that carries traffic for multiple VLANs over a single physical link.

Unlike an access port, which belongs to only one VLAN, a trunk transports frames from multiple VLANs by adding VLAN tags using IEEE 802.1Q.

---

# Why Trunks Are Needed

Without trunk links:

- VLANs exist only on individual switches.
- Devices on different switches cannot communicate even if they belong to the same VLAN.

With trunk links:

- VLANs extend across multiple switches.
- Devices in the same VLAN communicate as if connected to one switch.

---

# VLAN = Broadcast Domain

Every VLAN creates its own broadcast domain.

Each VLAN requires:

- Separate Layer 2 domain
- Separate IP subnet

Example:

VLAN 10

192.168.10.0/27

VLAN 20

192.168.10.32/27

---

# Access Port

Characteristics

- One VLAN
- Untagged traffic
- Connects PCs, printers, IP phones

Configuration

switchport mode access

switchport access vlan 10

---

# Trunk Port

Characteristics

- Multiple VLANs
- Tagged traffic
- Connects switches
- Uses IEEE 802.1Q

Configuration

switchport mode trunk

---

# IEEE 802.1Q

Industry standard VLAN tagging protocol.

Adds a 4-byte VLAN tag inside Ethernet frames.

Allows multiple VLANs to travel across one cable.

---

# Native VLAN

Default Native VLAN

VLAN 1

Native VLAN traffic is transmitted without VLAN tags.

---

# Allowed VLANs

Default

1-1005

Restricting VLANs reduces:

- Broadcast traffic
- Security risks

---

# Dangerous Command

switchport trunk allowed vlan 10

This replaces every allowed VLAN.

Only VLAN 10 is permitted.

Everything else is blocked.

---

# Safe Commands

switchport trunk allowed vlan add 20

switchport trunk allowed vlan remove 30

---

# Verify Trunks

show interfaces trunk

Verify:

- Status
- Encapsulation
- Native VLAN
- Allowed VLANs
- Active VLANs

---

# Communication Rules

Same VLAN + Same Subnet

Communication succeeds.

Different VLAN

Communication fails.

Inter-VLAN communication requires:

- Router
- Layer 3 Switch

---

# Important CCNA Notes

- VLAN = Broadcast Domain
- One VLAN = One Subnet
- Access ports carry one VLAN
- Trunk ports carry multiple VLANs
- IEEE 802.1Q is the standard
- Switches cannot route between VLANs