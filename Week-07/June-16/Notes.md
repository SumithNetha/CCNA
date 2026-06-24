# What is Subnetting?

## Definition

Subnetting is the process of dividing one larger IP network into multiple smaller networks (subnets).

Purpose:

* Reduce broadcast traffic
* Use IP addresses efficiently
* Create organized and scalable networks



# Why Subnetting Exists

Without subnetting, networks become:

* Large
* Noisy
* Hard to manage
* Wasteful of IP addresses

Example:

Network:
192.168.1.0/24

Usable Hosts:
254

If all devices share this network:

* Guest WiFi
* POS systems
* Cameras
* Office devices
* Servers

→ Broadcast traffic grows.

Subnetting solves this by creating separate networks.

---

# Reason 1 — Reduce Broadcast Domains

Broadcast = Traffic sent to every device in the subnet.

Examples:

* DHCP Discover
* ARP Requests

Example:

Who has 192.168.1.1?

Every device receives it.

Large networks create excessive broadcast traffic.

Subnetting reduces this problem.

Example:

192.168.1.0/24

↓

Guest WiFi:
192.168.1.0/25

Office:
192.168.1.128/27

POS:
192.168.1.160/28

Camera:
192.168.1.176/28

Result:
Broadcast stays inside each subnet.

---

# Reason 2 — Avoid Wasting IP Addresses

Example:

Router ---- Router

Only 2 devices.

Bad Design:

192.168.10.0/24

Available:
254 hosts

Used:
2 hosts

Waste:
252 addresses

Better:

192.168.10.0/30

Addresses:

192.168.10.0 → Network

192.168.10.1 → R1

192.168.10.2 → R2

192.168.10.3 → Broadcast

Perfect fit.



# Understanding Subnet Masks

Subnet Mask determines:

* Network portion
* Host portion

Example:

192.168.1.0/24

Mask:

255.255.255.0

Binary:

11111111.11111111.11111111.00000000

Meaning:

24 bits → Network

8 bits → Hosts

Hosts:

2^8 − 2 = 254



# Reserved Addresses

Example:

192.168.1.0/24

Reserved:

192.168.1.0 → Network Address

192.168.1.255 → Broadcast Address

Usable:

192.168.1.1 – 192.168.1.254



# Real World Thinking

Do not ask:

"What subnet mask should I use?"

Ask:

"How many devices need addresses?"

Examples:

Guest WiFi → Large subnet

Office → Small subnet

Point-to-point → Tiny subnet

Subnet size should match business requirements.

---

# Key Takeaways

Subnetting = Breaking large networks into smaller networks.

Benefits:

* Less broadcast traffic
* Efficient IP allocation
* Better security
* Easier troubleshooting
* Scalable design

Subnetting is the foundation for:

* VLANs
* Routing
* DHCP
* ACL
* NAT
* OSPF

Next Topic:
Converting Decimal to Binary
