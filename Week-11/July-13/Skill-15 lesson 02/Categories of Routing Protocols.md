# Categories of Routing Protocols

## Why This Matters

Knowing routing protocol commands is useful, but understanding **how routing protocols work** is what makes troubleshooting possible.

When routing fails:
- Credit card payments stop
- Online orders fail
- Branch connectivity is lost
- Business operations are affected

Understanding the **architecture** behind routing protocols is more valuable than memorizing commands.

---

# Routing Protocol Categories

Routing protocols are broadly divided into two categories:

```
                 Routing Protocols
                        │
          ┌─────────────┴─────────────┐
          │                           │
        IGP                         EGP
 (Inside Organization)      (Between Organizations)
          │                           │
   ┌──────┴──────┐                    │
   │             │                    │
Distance      Link-State             BGP
 Vector
```

---

# Interior Gateway Protocol (IGP)

## Definition

An **Interior Gateway Protocol (IGP)** is used for routing **inside a single organization**.

All routers belong to the same administrative domain.

### Examples

- OSPF
- EIGRP
- RIP
- IS-IS

### Real-World Example

NetworkChuck Coffee internal network:

- Headquarters
- Branch stores
- Warehouses
- POS systems
- Security cameras

All routing between these networks uses an **IGP**.

### Characteristics

- Internal routing
- Same organization
- Smaller network scope
- Faster and simpler to manage

---

# Exterior Gateway Protocol (EGP)

## Definition

An **Exterior Gateway Protocol (EGP)** is used for routing **between different organizations (Autonomous Systems).**

### Primary EGP

- **BGP (Border Gateway Protocol)**

### Real-World Example

NetworkChuck Coffee connects to:

- ISP
- Cloud provider
- Payment gateway
- Business partner

BGP exchanges routing information between these independent networks.

### Characteristics

- External routing
- Internet routing
- Business-to-business connectivity
- Policy-controlled routing

---

# Distance Vector Routing Protocols

## Concept

Distance Vector works like **routing by rumor**.

Each router learns routes only from its **neighbors**.

It does **not** know the complete network topology.

### How It Works

```
Router A ←→ Router B ←→ Router C
```

Router A trusts Router B's information without knowing the full network.

### Advantages

- Simple
- Low CPU usage
- Low memory usage
- Easy to configure

### Disadvantages

- No complete network view
- Slower convergence
- Depends on neighbor updates

### Examples

- RIP
- EIGRP *(as categorized in this lesson)*

### Troubleshooting Focus

If using a Distance Vector protocol, check:

- Neighbor relationships
- Routing updates
- Update timers
- Route advertisements

---

# Link-State Routing Protocols

## Concept

Link-State works like **routing by map**.

Each router builds a complete view of the network and calculates the best path independently.

### How It Works

```
        A
      /   \
     B     C
      \   /
        D
```

Every router understands the network topology.

### Advantages

- Faster convergence
- Better path selection
- Intelligent routing decisions

### Disadvantages

- Higher CPU usage
- Higher memory usage
- More complex than Distance Vector

### Examples

- OSPF
- IS-IS

### OSPF

**Open Shortest Path First (OSPF)** is the primary Link-State routing protocol covered in CCNA.

---

# Path Vector Routing

## Definition

Path Vector is primarily associated with **BGP**.

Instead of building a complete topology, BGP makes routing decisions based on:

- Path information
- Routing policies

### Why BGP?

The Internet is too large for routers to maintain a complete topology.

Instead, BGP exchanges path information and applies administrator-defined policies.

### Policy Examples

BGP allows administrators to control:

- Which routes to advertise
- Which routes to accept
- Preferred paths
- Blocked routes

This makes BGP ideal for:

- Internet routing
- ISP connectivity
- Multi-cloud environments
- Business partner connections

---

# Routing by Analogy

| Routing Type | Analogy |
|--------------|---------|
| Distance Vector | 🗣️ Routing by Rumor |
| Link-State | 🗺️ Routing by Map |
| Path Vector | 🌍 Routing by Policy & Path |

---

# Distance Vector vs Link-State

| Feature | Distance Vector | Link-State |
|----------|----------------|------------|
| Learns From | Neighbors | Entire topology |
| Network View | Partial | Complete |
| CPU Usage | Low | Higher |
| Memory Usage | Low | Higher |
| Convergence | Slower | Faster |
| Complexity | Simple | More advanced |

---

# IGP vs EGP

| IGP | EGP |
|-----|-----|
| Internal routing | External routing |
| Same organization | Different organizations |
| Smaller networks | Internet-scale networks |
| OSPF, RIP, EIGRP | BGP |

---

# Complete Classification

```
Routing Protocols

├── Interior Gateway Protocol (IGP)
│
├── Distance Vector
│   ├── RIP
│   └── EIGRP
│
└── Link-State
    ├── OSPF
    └── IS-IS

Exterior Gateway Protocol (EGP)

└── BGP
      └── Path Vector
```

---

# Key Takeaways

- Routing protocols are categorized based on **where they operate** and **how they learn routes**.
- **IGP** is used for routing within a single organization.
- **EGP** is used for routing between different organizations.
- **BGP** is the primary EGP and uses the **Path Vector** approach.
- **Distance Vector** protocols learn routes from neighbors ("routing by rumor").
- **Link-State** protocols build a complete network map ("routing by map").
- Link-State protocols generally provide **faster convergence** than Distance Vector protocols.
- BGP focuses on **routing policies** and **path selection**, making it ideal for Internet-scale networks.
- Understanding the architecture behind routing protocols makes configuration and troubleshooting much easier.