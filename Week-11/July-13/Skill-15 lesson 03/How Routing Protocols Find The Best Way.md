# How Routing Protocols Find The Best Way

## Overview

A router does **not automatically know** the best path to a destination.

It learns routes from multiple sources, compares them using a defined decision process, and installs **only the best route** into the routing table.

The routing table contains the **winning routes**, not every route the router has learned.

> **The routing table is the router's final answer, not its entire knowledge.**

---

# Router's Two Primary Jobs

## 1. Forward Packets

Move packets from one network to another using the routing table.

## 2. Determine the Best Path

Before forwarding traffic, the router decides:

- Which path is best?
- Which interface should be used?
- Which next-hop router should receive the packet?

---

# The Routing Table

The routing table contains only the **best routes** selected after route comparison.

A router may learn multiple routes to the same destination, but only one is installed in the routing table.

Example:

```
Static Route
OSPF Route
RIP Route

↓

Router selects one best route

↓

Installed in Routing Table
```

Backup routes remain available but are not active unless the preferred route fails.

---

# Route Selection Process

When multiple routes exist, the router compares them in the following order:

```
1. Longest Prefix Match (Most Specific Route)

↓

2. Administrative Distance (Most Trusted Source)

↓

3. Metric (Best Path Within Same Routing Protocol)

↓

Install Best Route
```

---

# Step 1 – Longest Prefix Match

The router first checks which route is **more specific**.

A route with a **longer subnet mask** always wins.

Example:

```
192.168.1.0/24
192.168.1.128/25
```

Destination:

```
192.168.1.200
```

Both routes match, but:

```
/25
```

is more specific than:

```
/24
```

Therefore, the router selects:

```
192.168.1.128/25
```

---

# Default Route

Default Route:

```
0.0.0.0/0
```

Meaning:

"If no specific route exists, send the packet here."

The default route is always the **last choice** because it is the **least specific** route.

---

# Step 2 – Administrative Distance (AD)

If two routes have the same prefix length, the router compares **Administrative Distance (AD).**

Administrative Distance measures **how trustworthy the source of a route is.**

**Lower AD = More Trusted**

### Common Administrative Distances

| Route Source | AD |
|--------------|---:|
| Directly Connected | 0 |
| Static Route | 1 |
| OSPF | 110 |
| RIP | 120 |

Example:

```
Network learned via:

OSPF  (AD 110)

RIP   (AD 120)
```

Result:

```
OSPF wins
```

because:

```
110 < 120
```

---

# Static Routes

Static routes are manually configured.

Example:

```bash
ip route 10.10.10.0 255.255.255.0 192.168.1.2
```

Advantages:

- Highly trusted
- Predictable routing
- Administrator-controlled

Disadvantages:

- Configuration errors can override dynamic routes
- Require manual maintenance

---

# Floating Static Route

A **Floating Static Route** is a static route configured with a **higher Administrative Distance** so it functions as a backup path.

Example:

Primary Route:

```
OSPF
AD = 110
```

Backup Route:

```
Static Route
AD = 121
```

Normal Operation:

```
OSPF is used
```

If OSPF fails:

```
Floating Static Route becomes active
```

Common use case:

- Backup WAN connection
- LTE / 5G backup link
- Secondary ISP

---

# Step 3 – Metric

If two routes:

- Have the same prefix length
- Come from the same routing protocol

The router compares the **Metric**.

A metric is the routing protocol's method of measuring the best path.

**Lower Metric = Better Path**

---

# RIP Metric

RIP uses:

```
Hop Count
```

Hop Count = Number of routers crossed.

Characteristics:

- Simple
- Ignores bandwidth
- Ignores delay
- Slower convergence

---

# OSPF Metric

OSPF uses:

```
Cost
```

Cost is primarily based on **bandwidth**.

Higher bandwidth:

```
↓

Lower Cost

↓

Preferred Path
```

---

# EIGRP Metric

EIGRP uses a **Composite Metric**.

It considers multiple network characteristics, including:

- Bandwidth
- Delay

This provides more intelligent path selection than simple hop count.

---

# Cisco Route Entry Format

Example:

```
S* 0.0.0.0/0 [1/0] via 216.0.5.1
```

Explanation:

| Field | Meaning |
|--------|---------|
| S | Static Route |
| * | Candidate Default Route |
| 0.0.0.0/0 | Default Route |
| [1/0] | Administrative Distance = 1, Metric = 0 |
| via 216.0.5.1 | Next-Hop Router |

---

# Common Route Codes

| Code | Meaning |
|------|---------|
| L | Local Interface Address |
| C | Connected Network |
| S | Static Route |
| R | RIP |
| D | EIGRP |
| O | OSPF |
| B | BGP |

---

# Connected vs Local Routes

### Connected Route (C)

Represents the directly connected network.

Example:

```
C 192.168.1.0/24
```

Meaning:

The entire subnet is directly connected.

---

### Local Route (L)

Represents the router's own interface IP address.

Example:

```
L 192.168.1.1/32
```

Meaning:

Only the router's interface IP.

A **/32** subnet identifies exactly one IP address.

---

# Variably Subnetted

Example:

```
192.168.1.0/24 is variably subnetted,
2 subnets,
2 masks
```

Meaning:

The router knows routes with different subnet masks within the same major network.

Example:

```
192.168.1.0/24
192.168.1.1/32
```

---

# Route Selection Flow

```
Packet Arrives

↓

Find Matching Routes

↓

Longest Prefix Match
(Most Specific)

↓

Administrative Distance
(Most Trusted Source)

↓

Metric
(Best Path Within Same Protocol)

↓

Install Best Route

↓

Forward Packet
```

---

# Key Takeaways

- Routers compare multiple routes before selecting the best path.
- The routing table stores only the **best (winning) routes**.
- Route selection follows this order:
  1. Longest Prefix Match
  2. Administrative Distance
  3. Metric
- **Longest Prefix Match** always takes priority.
- **Administrative Distance** determines the most trusted routing source.
- **Static Routes** have AD = **1** and are highly trusted.
- **Floating Static Routes** provide automatic backup routing by using a higher AD.
- **Metrics** are protocol-specific:
  - RIP → Hop Count
  - OSPF → Cost (Bandwidth)
  - EIGRP → Composite Metric
- Cisco route entries display:
  - Route source
  - Destination network
  - Administrative Distance
  - Metric
  - Next-hop router
- Common route codes:
  - **C** = Connected
  - **L** = Local
  - **S** = Static
  - **R** = RIP
  - **O** = OSPF
  - **D** = EIGRP
  - **B** = BGP
- Understanding the route selection process makes routing tables predictable and simplifies troubleshooting.