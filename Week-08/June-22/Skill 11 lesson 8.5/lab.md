# Applying New Castle Rysen Subnet Strategy – Lab

## Lab Objective

Implement the new VLSM addressing scheme for the Castle Rysen District Shop network by updating router interfaces, switch management interfaces, NAT configuration, and DHCP services. Verify that all devices communicate correctly using the new subnet.

---

# Lab Topology

The network consists of:

* ISP Router
* Cafe Router (cafe01-RT01)
* Two Layer 2 Switches
* Two Wireless Access Points
* One Server
* Two Client PCs

The District Shop LAN uses the following subnet:

| Network         | Value           |
| --------------- | --------------- |
| Network Address | 10.0.18.0/26    |
| Subnet Mask     | 255.255.255.192 |
| Gateway         | 10.0.18.1       |

---

# IP Addressing Plan

| Device            | IP Address |
| ----------------- | ---------- |
| Router (Gi0/0)    | 10.0.18.1  |
| Switch 1 (VLAN 1) | 10.0.18.2  |
| Switch 2 (VLAN 1) | 10.0.18.3  |
| Access Point 1    | 10.0.18.4  |
| Access Point 2    | 10.0.18.5  |
| Server            | 10.0.18.6  |

### Address Allocation

| Address Range           | Purpose                 |
| ----------------------- | ----------------------- |
| 10.0.18.1 – 10.0.18.10  | Infrastructure Devices  |
| 10.0.18.11 – 10.0.18.50 | DHCP Clients            |
| 10.0.18.51 – 10.0.18.62 | Reserved Static Devices |

---

# Part 1 – Router Configuration

Updated the LAN interface with the new subnet.

* Assigned the router interface **10.0.18.1/26**.
* Removed obsolete interface addressing.
* Verified interface status.

### Verification

```bash
show ip interface brief
```

Confirmed:

* Interface is Up/Up
* Correct IP address assigned
* WAN interface remains unchanged

---

# Part 2 – Routing Verification

Verified that Cisco IOS automatically updated the routing table after assigning the new interface address.

### Verification

```bash
show ip route
```

Confirmed:

* Connected Route (C)
* Local Route (L)
* Default Route (S*)

---

# Part 3 – NAT Configuration

Verified the existing NAT configuration.

Removed the old access list that referenced the previous subnet.

Created a new ACL matching the new network:

* Network: **10.0.18.0/26**
* Wildcard Mask: **0.0.0.63**

Verified that NAT continued using PAT (NAT Overload) through the WAN interface.

### Verification

```bash
show ip nat statistics
```

```bash
show access-lists
```

```bash
show running-config | include nat
```

---

# Part 4 – DHCP Configuration

Configured the router as the DHCP server.

### Excluded Addresses

Infrastructure devices:

* 10.0.18.1 – 10.0.18.10

Reserved future devices:

* 10.0.18.51 – 10.0.18.63

### DHCP Pool Configuration

Configured:

* Network
* Subnet Mask
* Default Gateway
* DNS Server

Clients were configured to obtain IP addresses automatically.

Expected client address range:

* 10.0.18.11 – 10.0.18.50

---

# Part 5 – Switch 1 Configuration

Updated the management interface (VLAN 1).

Previous Address:

* 192.168.0.2

New Address:

* 10.0.18.2

Saved the configuration.

Verified connectivity to the router.

### Verification

```bash
ping 10.0.18.1
```

The first ping experienced one timeout due to ARP resolution. Subsequent pings were successful.

---

# Part 6 – Switch 2 Configuration

Updated the management interface (VLAN 1).

Previous Address:

* 192.168.0.3

New Address:

* 10.0.18.3

Saved the configuration.

Verified connectivity to the router.

### Verification

```bash
ping 10.0.18.1
```

After ARP resolution, all ping requests completed successfully.

---

# Part 7 – MAC Address Verification

Verified that both switches dynamically learned connected device MAC addresses.

### Verification

```bash
show mac-address-table
```

Confirmed:

* Dynamic MAC learning
* Correct port associations
* Normal Layer 2 forwarding operation

---

# Part 8 – Interface Verification

Verified interface status on the router and switches.

### Commands Used

```bash
show ip interface brief
```

Confirmed:

* Correct interface IP addresses
* Interfaces in Up/Up state
* VLAN management interfaces reachable

---

# Verification Commands

## Router

```bash
show ip interface brief
```

```bash
show ip route
```

```bash
show ip nat statistics
```

```bash
show access-lists
```

```bash
show running-config | include nat
```

---

## Switches

```bash
show ip interface brief
```

```bash
show mac-address-table
```

```bash
ping 10.0.18.1
```

---

# Expected Results

* Router LAN interface uses **10.0.18.1/26**.
* Switch management interfaces use **10.0.18.2** and **10.0.18.3**.
* NAT translates traffic from the new subnet.
* DHCP assigns addresses from **10.0.18.11 – 10.0.18.50**.
* Infrastructure devices use static addresses.
* Clients communicate successfully with the default gateway.
* Switches learn MAC addresses dynamically.
* The network successfully operates using the new VLSM addressing plan.

---

# Lab Outcome

Successfully migrated the District Shop network from the previous addressing scheme to the new **10.0.18.0/26** subnet. Updated router interfaces, switch management interfaces, NAT, and DHCP configuration while preserving connectivity. Verification confirmed that routing, address assignment, NAT translation, and Layer 2 communication all function correctly under the new subnet strategy.
