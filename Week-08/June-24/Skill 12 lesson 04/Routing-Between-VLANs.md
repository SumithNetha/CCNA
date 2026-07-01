# Skill 12 Lesson 04 – Routing Between VLANs

## Objective

Learn why devices in different VLANs cannot communicate directly and configure Inter-VLAN Routing using the Router-on-a-Stick (ROAS) method.

---

# Why VLANs Cannot Communicate

A VLAN creates its own:

- Layer 2 Broadcast Domain
- IP Subnet
- Logical Network

Devices inside the same VLAN communicate directly using switching.

Devices in different VLANs require a Layer 3 device because they belong to different IP networks.

Example

```
VLAN 10
10.0.18.0/27

↓

Router Required

↓

VLAN 20
10.0.18.32/27
```

Without a router:

- Same VLAN → Communication works
- Different VLAN → Communication fails

---

# Inter-VLAN Routing

Inter-VLAN Routing is the process of routing traffic between different VLANs using a Layer 3 device.

A router or Layer 3 switch performs this routing.

---

# Two Methods of Inter-VLAN Routing

## Method 1 – Legacy Routing

Each VLAN connects to a separate physical router interface.

Example

```
Router

G0/0 → VLAN10

G0/1 → VLAN20

G0/2 → VLAN30
```

### Advantages

- Easy to understand

### Disadvantages

- Requires one physical interface per VLAN
- Wastes router interfaces
- Not scalable

---

## Method 2 – Router-on-a-Stick (ROAS)

One physical router interface is connected to a switch trunk.

Multiple logical subinterfaces are created.

Each subinterface belongs to one VLAN.

Example

```
Router

G0/0.10 → VLAN10

G0/0.20 → VLAN20

G0/0.30 → VLAN30
```

Only one physical interface is required.

This is the preferred CCNA solution.

---

# Router-on-a-Stick Architecture

```
            Router
         Gig0/0
            |
      802.1Q Trunk
            |
        Cisco Switch
      ---------------
      VLAN10 VLAN20
```

Traffic reaches the router over one trunk.

The router routes packets between VLANs.

---

# Subinterfaces

A subinterface is a logical interface created from one physical interface.

Example

```
GigabitEthernet0/0.10

GigabitEthernet0/0.20
```

Best practice:

```
Subinterface Number = VLAN Number
```

Although not required, this greatly simplifies troubleshooting.

---

# 802.1Q Encapsulation

Each subinterface must identify which VLAN it belongs to.

Command

```cisco
encapsulation dot1Q <VLAN-ID>
```

Example

```cisco
interface GigabitEthernet0/0.10

encapsulation dot1Q 10
```

Example

```cisco
interface GigabitEthernet0/0.20

encapsulation dot1Q 20
```

Without encapsulation, the router cannot identify tagged VLAN traffic.

---

# Router Configuration

## Remove IP Address from Physical Interface

```cisco
interface GigabitEthernet0/0

no ip address

no shutdown
```

The physical interface carries VLAN traffic only.

IP addresses belong to the subinterfaces.

---

## Configure VLAN 10

```cisco
interface GigabitEthernet0/0.10

encapsulation dot1Q 10

ip address 10.0.18.1 255.255.255.224
```

---

## Configure VLAN 20

```cisco
interface GigabitEthernet0/0.20

encapsulation dot1Q 20

ip address 10.0.18.33 255.255.255.224
```

---

# Default Gateway

Each VLAN requires its own default gateway.

Example

| VLAN | Gateway |
|------|---------|
| VLAN10 | 10.0.18.1 |
| VLAN20 | 10.0.18.33 |

Hosts send traffic destined for other networks to the gateway.

---

# DHCP Configuration

Each VLAN needs its own DHCP pool.

Example

```cisco
ip dhcp pool ADMIN-10

network 10.0.18.0 255.255.255.224

default-router 10.0.18.1

dns-server 1.1.1.1
```

Second Pool

```cisco
ip dhcp pool PATRON-20

network 10.0.18.32 255.255.255.224

default-router 10.0.18.33

dns-server 1.1.1.1
```

Each VLAN receives addresses only from its own subnet.

---

# Switch Configuration

The switch port connected to the router must operate as a trunk.

Example

```cisco
interface FastEthernet0/24

switchport mode trunk
```

Without trunking:

- Router receives only one VLAN
- DHCP fails
- Inter-VLAN routing fails

---

# Verification Commands

## Router

```cisco
show ip interface brief
```

Verify subinterfaces.

---

```cisco
show running-config
```

Verify:

- Subinterfaces
- DHCP Pools
- IP Addresses

---

```cisco
show ip dhcp binding
```

Verify DHCP leases.

Example

```
10.0.18.2

10.0.18.34
```

---

## Switch

```cisco
show interfaces trunk
```

Verify:

- Trunk Status
- Allowed VLANs
- Active VLANs

---

```cisco
show vlan brief
```

Verify VLAN membership.

---

# Packet Tracer Verification

## DHCP

Initially

```
0.0.0.0
```

After DHCP failure

```
169.254.x.x
```

Meaning

- DHCP server unreachable
- Trunk missing
- Wrong VLAN
- Wrong gateway

After fixing trunk

```
VLAN10

10.0.18.2

Gateway

10.0.18.1
```

```
VLAN20

10.0.18.34

Gateway

10.0.18.33
```

---

# Ping Verification

Same VLAN

```
Success
```

Different VLAN after ROAS

```
Success
```

Without ROAS

```
Failure
```

---

# Traceroute Verification

Example

```
PC

↓

10.0.18.1

↓

10.0.18.34
```

The router appears as the first hop.

This confirms Layer 3 routing is occurring.

---

# Common Troubleshooting

## Client Receives 169.254.x.x

Possible Causes

- DHCP server unreachable
- Router trunk missing
- Wrong VLAN
- Wrong switch port
- Incorrect default gateway

---

## Devices Cannot Communicate

Check

```cisco
show interfaces trunk
```

Verify trunk is operational.

---

Check

```cisco
show vlan brief
```

Verify VLAN assignments.

---

Check

```cisco
show ip interface brief
```

Verify router subinterfaces are up.

---

Check

```cisco
show ip dhcp binding
```

Confirm clients received addresses.

---

# Common Mistakes

- Leaving IP address on physical router interface.
- Forgetting encapsulation dot1Q.
- Forgetting to configure the switch port as a trunk.
- Wrong VLAN ID on subinterface.
- Wrong default gateway.
- DHCP pool configured with incorrect network.
- Assigning hosts to the wrong VLAN.

---

# Best Practices

- Match subinterface numbers to VLAN IDs.
- Configure one DHCP pool per VLAN.
- Use meaningful DHCP pool names.
- Always verify trunk status after configuration.
- Test with ping and traceroute.
- Verify DHCP before troubleshooting routing.

---

# Key Commands

```cisco
interface GigabitEthernet0/0

no ip address
```

```cisco
interface GigabitEthernet0/0.10

encapsulation dot1Q 10

ip address 10.0.18.1 255.255.255.224
```

```cisco
interface GigabitEthernet0/0.20

encapsulation dot1Q 20

ip address 10.0.18.33 255.255.255.224
```

```cisco
switchport mode trunk
```

```cisco
show interfaces trunk
```

```cisco
show vlan brief
```

```cisco
show ip interface brief
```

```cisco
show ip dhcp binding
```

---

# Real-World Tips

- A router is required for communication between VLANs.
- Router-on-a-Stick is widely used in small and medium-sized networks.
- Layer 3 switches are preferred in large enterprise environments for higher performance.
- A 169.254.x.x address almost always indicates a DHCP issue.
- Traceroute is an excellent tool for verifying that packets are routed through the default gateway.

---

# Summary

- Every VLAN is a separate Layer 2 broadcast domain and Layer 3 subnet.
- Devices in different VLANs require Inter-VLAN Routing.
- Router-on-a-Stick uses one physical router interface with multiple subinterfaces.
- Each subinterface requires an IP address and `encapsulation dot1Q`.
- The switch-to-router connection must be configured as a trunk.
- Configure separate DHCP pools and default gateways for each VLAN.
- Verify operation using `show interfaces trunk`, `show vlan brief`, `show ip interface brief`, `show ip dhcp binding`, `ping`, and `traceroute`.