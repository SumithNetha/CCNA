# Re-addressing Castle Rysen Coffee

## Overview

Re-addressing is the process of redesigning an existing IP addressing scheme to improve scalability, efficiency, organization, and future growth.

Instead of creating random IP addresses, every department or VLAN receives a properly planned subnet using Variable Length Subnet Masking (VLSM).

This project simulates how enterprise network engineers redesign production networks.

---

# Objectives

- Reduce IPv4 address wastage
- Improve scalability
- Organize departments into separate networks
- Prepare for future expansion
- Support routing and VLAN implementation
- Simplify troubleshooting

---

# Why Re-addressing is Necessary

As organizations grow, they require:

- More users
- More departments
- More servers
- More network devices
- Better network organization

Poor addressing schemes become difficult to manage and troubleshoot.

Re-addressing creates a structured IP addressing plan.

---

# Re-addressing Process

## 1. Gather Requirements

Determine:

- Number of departments
- Number of hosts
- Servers
- Printers
- Network devices
- Expected future growth

---

## 2. Calculate Host Requirements

Determine the required subnet size for every department.

Example:

| Department | Hosts Needed |
|------------|-------------:|
| Sales |100|
| HR |50|
| Management |20|
| Guest Wi-Fi |40|
| Servers |10|

---

## 3. Apply VLSM

Allocate subnets from largest to smallest.

This minimizes IPv4 address wastage.

---

## 4. Assign Network Addresses

Each department receives its own subnet.

Example:

- Administration
- Sales
- Management
- Servers
- Guest Wi-Fi
- Voice
- Network Management

---

## 5. Assign Default Gateways

Each subnet requires a gateway.

Example:

Network:
192.168.10.0/24

Gateway:
192.168.10.1

The gateway is typically the first or last usable IP address.

---

## 6. Reserve Infrastructure Addresses

Reserve IP addresses for:

- Routers
- Layer 3 Switches
- Firewalls
- Wireless Access Points
- Servers
- Printers

Keeping infrastructure addresses consistent simplifies management.

---

## 7. Document the Addressing Plan

Each subnet should include:

- Network Address
- Prefix Length
- Broadcast Address
- First Host
- Last Host
- Default Gateway
- Department/VLAN Name

Proper documentation is essential for troubleshooting and future expansion.

---

# Benefits

- Efficient IPv4 utilization
- Better scalability
- Easier troubleshooting
- Organized network design
- Improved routing
- Easier VLAN implementation
- Simplified DHCP configuration
- Better security segmentation

---

# Connection to Future CCNA Topics

The addressing plan created during re-addressing becomes the foundation for:

- VLANs
- Inter-VLAN Routing
- DHCP
- NAT
- ACLs
- OSPF
- IPv6
- Layer 2 Security
- Enterprise Network Design

---

# Best Practices

- Design before configuration.
- Allocate large networks first.
- Leave room for future growth.
- Keep addressing consistent.
- Document every subnet.
- Avoid overlapping address spaces.
- Reserve infrastructure addresses.

---

# Key Takeaways

- Re-addressing redesigns an existing IP addressing scheme.
- VLSM is used to allocate efficient subnet sizes.
- Every department should have its own subnet.
- Every subnet requires a valid default gateway.
- Proper documentation is critical for network maintenance.
- A well-designed addressing plan supports future CCNA topics such as VLANs, routing, DHCP, NAT, and security.