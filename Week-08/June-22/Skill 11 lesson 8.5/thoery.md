# Applying New Castle Rysen Subnet Strategy

## Overview

Designing a subnetting scheme is only the planning phase of network deployment. A subnet plan becomes useful only after it is implemented on actual network devices such as routers, switches, servers, wireless access points, and client computers.

In this lesson, the previously designed Variable Length Subnet Masking (VLSM) addressing plan is applied to the Castle Rysen District Shop network. The goal is to migrate the network from its old addressing scheme to a new subnet while maintaining connectivity and ensuring that all network services continue to function correctly.

---

# Learning Objectives

After completing this lesson, you should be able to:

* Apply a subnetting design to a real network topology.
* Assign logical IP addresses to infrastructure devices.
* Understand the importance of consistent IP addressing.
* Update network services after changing a subnet.
* Verify that the network operates correctly after migration.

---

# Moving from Design to Deployment

Subnetting calculations alone do not create a working network. Every device in the network must be configured to use the new addressing scheme.

A successful subnet migration requires updating:

* Router interfaces
* Switch management interfaces
* Servers
* Wireless access points
* Client devices
* NAT configuration
* DHCP configuration

Only after these components are updated does the subnet design become an operational network.

---

# Visualizing the Network

Before configuring devices, it is important to understand the network topology.

Drawing the network helps administrators identify:

* Device connections
* Network boundaries
* Default gateways
* Infrastructure devices
* Possible configuration errors

Network engineers frequently redraw network diagrams during deployment and troubleshooting because visualizing the network makes it easier to understand traffic flow and identify problems.

---

# Understanding the Address Range

Rather than viewing an address only in slash notation, administrators should immediately recognize the entire subnet.

Example:

**Network:** `10.0.18.0/26`

| Item              | Value                  |
| ----------------- | ---------------------- |
| Network Address   | 10.0.18.0              |
| Subnet Mask       | 255.255.255.192        |
| Usable Host Range | 10.0.18.1 – 10.0.18.62 |
| Broadcast Address | 10.0.18.63             |

Developing the ability to recognize address ranges without performing calculations every time is an essential networking skill.

---

# Logical IP Address Assignment

Infrastructure devices should follow a consistent addressing pattern throughout the network.

A common practice is assigning the first usable addresses to core network devices.

Example:

| Device         | Address   |
| -------------- | --------- |
| Router         | 10.0.18.1 |
| Switch 1       | 10.0.18.2 |
| Switch 2       | 10.0.18.3 |
| Access Point 1 | 10.0.18.4 |
| Access Point 2 | 10.0.18.5 |
| Server         | 10.0.18.6 |

Using predictable addresses simplifies network documentation, troubleshooting, and future expansion.

---

# Organizing Address Space

Instead of assigning addresses randomly, administrators reserve portions of the subnet for different purposes.

Example allocation:

| Address Range           | Purpose                |
| ----------------------- | ---------------------- |
| 10.0.18.1 – 10.0.18.10  | Infrastructure devices |
| 10.0.18.11 – 10.0.18.50 | DHCP clients           |
| 10.0.18.51 – 10.0.18.62 | Future static devices  |

A structured allocation strategy makes it easier to add new devices without reorganizing the network.

---

# Updating Network Services

Changing the IP addressing scheme affects more than just device interfaces. Supporting network services must also be updated.

## Network Address Translation (NAT)

NAT translates private IP addresses into public IP addresses so internal devices can communicate with external networks.

When the internal subnet changes, NAT rules and access control lists must also be updated to match the new address range. If these rules are not updated, Internet connectivity will fail because internal traffic will not be translated correctly.

---

## Dynamic Host Configuration Protocol (DHCP)

DHCP automatically assigns IP configuration to client devices.

A DHCP server typically provides:

* IP address
* Subnet mask
* Default gateway
* DNS server

Infrastructure devices that require static addresses should be excluded from the DHCP pool to prevent address conflicts.

---

# Importance of Consistent Addressing

Using a standardized addressing convention provides several advantages:

* Easier troubleshooting
* Simpler documentation
* Faster device identification
* Better scalability
* Reduced configuration errors

Consistency becomes increasingly important as networks grow larger and support more devices.

---

# Network Verification

After implementing the new subnet strategy, the network should be verified to ensure that all devices operate correctly.

Typical verification includes:

* Confirming interface status
* Verifying routing information
* Checking NAT operation
* Confirming DHCP address assignment
* Testing connectivity between devices
* Verifying switch MAC address learning

Successful verification confirms that the new addressing plan has been implemented correctly.

---

# Best Practices

* Plan the addressing scheme before deployment.
* Assign infrastructure devices predictable IP addresses.
* Reserve address ranges for future expansion.
* Update dependent services whenever the subnet changes.
* Remove obsolete configurations to avoid conflicts.
* Verify network functionality after every major configuration change.
* Maintain accurate network documentation throughout the deployment process.

---

# Summary

Applying a subnet strategy transforms a theoretical addressing plan into a functioning network. Every device must be configured to use the new subnet, and related services such as NAT and DHCP must be updated to support the new addressing scheme.

A successful deployment depends not only on correct subnet calculations but also on organized address assignment, proper configuration of supporting services, and thorough verification after implementation. Following a structured deployment process ensures that the network remains scalable, maintainable, and reliable as it grows.
![alt text](image.png)