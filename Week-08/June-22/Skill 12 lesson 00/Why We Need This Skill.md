# theory.md

# Why We Need This Skill

## Overview

As networks grow, connecting every device to the same network becomes inefficient, difficult to manage, and less secure. Enterprise networks require logical separation so that different groups of users and devices can communicate efficiently while remaining isolated when necessary.

Virtual LANs (VLANs) provide this separation by allowing multiple logical networks to exist on the same physical switching infrastructure.

---

# The Problem with Flat Networks

A **flat network** is a network where all devices belong to the same broadcast domain.

Characteristics of a flat network include:

* Every device shares the same Layer 2 network.
* Broadcast traffic reaches every device.
* Security boundaries are minimal.
* Network management becomes more difficult as the network grows.

Although a flat network may be suitable for small environments, it does not scale well for business networks.

---

# Broadcast Domains

A **broadcast domain** is a group of devices that receive Layer 2 broadcast frames.

In a flat network:

* Every device receives broadcast traffic.
* More devices generate more broadcasts.
* Network efficiency decreases as the broadcast domain grows.

Reducing broadcast domains improves network performance and manageability.

---

# What is a VLAN?

A **Virtual Local Area Network (VLAN)** is a logical grouping of devices within a switched network.

Unlike physical network separation, VLANs create separate logical networks while using the same physical switches and cabling.

Each VLAN operates as an independent broadcast domain.

---

# Why VLANs Are Needed

VLANs solve several common networking challenges.

## Network Segmentation

Devices can be grouped according to their function instead of their physical location.

Examples include:

* Employee devices
* Guest Wi-Fi
* IP phones
* Security cameras
* Servers
* Management devices

Each group operates independently within its own VLAN.

---

## Improved Security

Sensitive business devices should not share the same network as guest devices.

Examples:

* Guest Wi-Fi should not access company servers.
* Security cameras should remain isolated from employee computers.
* Payment systems should be separated from public networks.

VLANs establish logical security boundaries between these groups.

---

## Reduced Broadcast Traffic

Because each VLAN forms its own broadcast domain:

* Broadcast traffic remains inside the VLAN.
* Unnecessary broadcasts are reduced.
* Network performance improves.

---

## Better Network Management

Organizing devices by department or function simplifies:

* IP address planning
* Troubleshooting
* Policy implementation
* Network expansion
* Device administration

Logical organization is easier to maintain than physical separation.

---

## Cost Efficiency

Without VLANs, separating departments would require additional switches and cabling.

VLANs allow multiple logical networks to share the same switching hardware, reducing equipment costs while maintaining separation.

---

# VLANs and IP Addressing

Each VLAN is typically assigned its own IP subnet.

Example:

| VLAN    | Purpose    | Example Subnet  |
| ------- | ---------- | --------------- |
| VLAN 10 | Employees  | 192.168.10.0/24 |
| VLAN 20 | Guests     | 192.168.20.0/24 |
| VLAN 30 | Cameras    | 192.168.30.0/24 |
| VLAN 40 | Management | 192.168.40.0/24 |

This relationship between VLANs and subnets simplifies routing and network design.

---

# Enterprise Example

A coffee shop network may include:

* Point-of-sale terminals
* Employee laptops
* Guest Wi-Fi
* Security cameras
* Wireless access points
* Printers

Instead of placing every device in one network, each category can be assigned to its own VLAN, improving security, performance, and management.

---

# Benefits of VLANs

* Logical network segmentation
* Smaller broadcast domains
* Improved security
* Better scalability
* Simplified IP addressing
* Easier troubleshooting
* Reduced infrastructure costs
* Better control over network communication

---

# Preparing for the Next Lessons

This lesson introduces the purpose of VLANs. The following lessons will cover:

* VLAN creation
* VLAN assignment
* Trunk links
* Inter-VLAN routing
* VLAN security
* Enterprise VLAN implementation

---

# Key Takeaways

* Flat networks do not scale efficiently.
* VLANs create multiple logical networks on the same physical infrastructure.
* Each VLAN forms its own broadcast domain.
* VLANs improve security, reduce broadcast traffic, and simplify management.
* Most enterprise networks use VLANs to separate users, departments, and services.
* VLANs provide the foundation for advanced switching and inter-VLAN communication.
