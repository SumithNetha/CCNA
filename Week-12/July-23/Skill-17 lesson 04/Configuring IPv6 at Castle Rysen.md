# Week 12 — July 23

## Skill 17 — Lesson 04: Configuring IPv6 at Castle Rysen

This lesson takes IPv6 from **theory into a working multi-site network**. The main objective is to extend the existing IPv6 configuration from the Castle Rysen Cafe to the **Fallout Shelter**, connect the sites over a WAN link, and use **static IPv6 routing** so the two locations can communicate.

---

## 1. IPv6 at Castle Rysen

The implementation has three major parts:

1. Configure IPv6 addressing on the Fallout Shelter VLANs.
2. Configure the IPv6 WAN link between the Cafe and Fallout Shelter.
3. Configure static IPv6 routes between the two locations.

The result is an **IPv6 overlay connecting the Cafe and Fallout Shelter networks**.

---

## 2. Global Unicast Addresses

A **Global Unicast Address (GUA)** is an IPv6 address that is globally routable.

The lesson contrasts IPv6 with traditional IPv4 addressing:

### IPv4 mindset

```text
Limited public IPv4 addresses
        ↓
Private IPv4 addresses
        ↓
NAT
        ↓
Internet
```

### IPv6 mindset

```text
Large IPv6 address space
        ↓
Global Unicast Addresses
        ↓
Directly routable addressing
        ↓
Control access with security mechanisms
```

The key shift is:

> IPv4 often focuses on conserving addresses, while IPv6 focuses more on organizing the enormous available address space.

At Castle Rysen, separate IPv6 subnets are assigned to the different VLANs.

For example:

```text
Cafe
├── Admin VLAN
├── Internal VLAN
├── Guest VLAN
└── Other VLANs

Fallout Shelter
├── Management VLAN
├── Internal VLAN
├── Video VLAN
├── Guest VLAN
└── Other VLANs
```

A separate IPv6 subnet is also used for the **WAN link**.

---

## 3. IPv6 Address Types

Three address categories are particularly important in this lesson.

| Address Type   | Range                        | Purpose                                |
| -------------- | ---------------------------- | -------------------------------------- |
| Global Unicast | Globally routable IPv6 space | Communication across networks/Internet |
| Unique Local   | `FD00::/8`                   | Internal/private-style addressing      |
| Link-Local     | `FE80::/10`                  | Local-link communication               |

### Global Unicast

Used for routable IPv6 communication.

```text
Global Unicast
     ↓
Can be routed between networks
```

### Unique Local Address

The lesson emphasizes:

```text
FD00::/8
```

These are intended for internal communication and are not routed over the public Internet.

### Link-Local

```text
FE80::/10
```

Link-local addresses are automatically associated with IPv6 interfaces and are used for communication on the local network segment.

They are important for things such as:

* Neighbor communication
* Router discovery
* Local IPv6 operations

---

## 4. EUI-64

**EUI-64** can automatically generate the interface ID portion of an IPv6 address using the device's MAC address.

The basic concept presented in the lesson is:

```text
MAC Address
     ↓
Split into two halves
     ↓
Insert FFFE
     ↓
Generate Interface ID
```

Conceptually:

```text
MAC
AA:BB:CC:DD:EE:FF

       ↓

AA:BB:CC:FF:FE:DD:EE:FF
```

The important idea is that the **network prefix is supplied**, while EUI-64 can generate the host/interface portion.

### Practical takeaway

For the Castle Rysen interfaces, EUI-64 was used so the routers could automatically generate the interface ID from the MAC address.

---

## 5. IPv6 WAN Link

After addressing the Fallout Shelter interfaces, the next step was connecting the **Cafe and Fallout Shelter**.

The WAN link is a point-to-point connection.

For this particular link, the lesson manually assigns IPv6 addresses rather than relying on EUI-64.

### Why?

Because manually assigned addresses make the **next-hop addresses predictable and easy to reference**.

Conceptually:

```text
Cafe Router
     |
     | IPv6 WAN
     |
Fallout Shelter Router
```

---

## 6. IPv6 Static Routing

The routing concept remains essentially the same as IPv4.

### IPv4

```text
ip route
```

### IPv6

```text
ipv6 route
```

A static route tells the router:

> "To reach this remote network, send the traffic through this path."

---

## 7. Routing Between Castle Rysen Sites

The Cafe router needs routes toward the Fallout Shelter networks.

```text
Cafe Router
    |
    | Static IPv6 routes
    ↓
Fallout Shelter VLANs
```

The Fallout Shelter router needs the reverse routes:

```text
Fallout Shelter Router
    |
    | Static IPv6 routes
    ↓
Cafe VLANs
```

So the routing must be **bidirectional**.

### Important concept

It isn't enough for the Cafe to know how to reach the Fallout Shelter.

The Fallout Shelter must also know how to return traffic to the Cafe.

```text
Cafe Network
    ↓
Cafe Router
    ↓
WAN
    ↓
Shelter Router
    ↓
Shelter Network
```

And the return path:

```text
Shelter Network
    ↓
Shelter Router
    ↓
WAN
    ↓
Cafe Router
    ↓
Cafe Network
```

---

## 8. Verification

Once addressing and static routes were configured, connectivity was tested using **ping across the WAN** to a remote interface.

The successful ping demonstrated that:

```text
IPv6 Addressing
       +
IPv6 WAN Connectivity
       +
Static IPv6 Routing
       ↓
Working IPv6 Network
```

This is the key transition from simply configuring IPv6 addresses to actually building a functioning IPv6 network.

---

## 9. Commands to Remember

The lesson specifically contrasts the IPv4 and IPv6 static-route commands:

```text
IPv4:
ip route

IPv6:
ipv6 route
```

For IPv6 troubleshooting and verification, the concepts you should be comfortable with are:

```text
IPv6 interface addressing
Link-local addresses
Global unicast addresses
EUI-64
IPv6 static routes
IPv6 ping/connectivity testing
```

---

## 10. Key Takeaways

### IPv6 address types

```text
GUA       → Globally routable
FD00::/8  → Unique Local/internal
FE80::/10 → Link-local
```

### EUI-64

```text
MAC address
    ↓
Interface ID generation
```

### Static routing

```text
IPv4 → ip route
IPv6 → ipv6 route
```

### Castle Rysen implementation

```text
             IPv6 WAN
       ┌─────────────────┐
       │                 │
   Cafe Router      Shelter Router
       │                 │
   Cafe VLANs        Shelter VLANs
```

The central lesson is **IPv6 becomes much easier once you start configuring and troubleshooting it instead of only memorizing address formats**. The Castle Rysen implementation combines **addressing + EUI-64 + WAN connectivity + static routing** into one functioning IPv6 network.
