# Tuning EtherChannel Load Balancing

## Overview

EtherChannel provides **aggregate bandwidth** by combining multiple physical links into a single logical interface (Port-Channel). However, it **does not increase the bandwidth available to a single conversation**.

- One network flow uses **one physical link**.
- Multiple independent flows are distributed across the available links.
- Traffic distribution is performed using a **hash-based load-balancing algorithm**.

> **Key Concept:** EtherChannel increases total network capacity, not the speed of an individual connection.

---

# One Conversation vs Multiple Conversations

## Single Conversation

Two 1 Gbps interfaces in an EtherChannel do **not** provide 2 Gbps for one transfer.

```
PC ─────────► Server

EtherChannel
├── Gi0/1 (1 Gbps)  ← Used
└── Gi0/2 (1 Gbps)  ← Idle
```

Maximum throughput:

- **1 Gbps**

---

## Multiple Conversations

When several devices communicate simultaneously, different traffic flows can use different links.

```
PC1 ─────► Server
PC2 ─────► Server
Camera ──► NVR
POS ─────► Database

          EtherChannel

Flow 1 → Gi0/1
Flow 2 → Gi0/2
Flow 3 → Gi0/1
Flow 4 → Gi0/2
```

Benefits:

- Better bandwidth utilization
- Increased aggregate throughput
- Redundancy

---

# How EtherChannel Selects a Link

EtherChannel uses a **hashing algorithm** to determine which physical interface carries a frame.

The hash may be calculated using:

- Source MAC address
- Destination MAC address
- Source IP address
- Destination IP address

Because hashing is deterministic:

- Same inputs
- Same hash value
- Same physical link

This keeps packets in order and prevents packet reordering.

---

# Checking the Current Load-Balancing Method

Display the active hashing algorithm:

```bash
show etherchannel load-balance
```

Example output:

```text
Operational State:
src-mac
```

Meaning:

- Traffic is distributed based on the **Source MAC Address**.

---

# Why Change the Load-Balancing Algorithm?

The default algorithm may not distribute traffic efficiently for every network.

Example:

Many users communicating with the same server.

```
PC1 ─┐
PC2 ─┤
PC3 ─┤──► Server
PC4 ─┘
```

Using only the source MAC may create uneven link utilization.

Choosing a more suitable algorithm provides better traffic distribution.

Examples:

- `src-dst-mac`
- `src-dst-ip`

The best choice depends on which packet fields vary between different conversations.

---

# Common Load-Balancing Algorithms

| Algorithm | Uses | Layer |
|-----------|------|-------|
| src-mac | Source MAC | Layer 2 |
| dst-mac | Destination MAC | Layer 2 |
| src-dst-mac | Source + Destination MAC | Layer 2 |
| src-ip | Source IP | Layer 3 |
| dst-ip | Destination IP | Layer 3 |
| src-dst-ip | Source + Destination IP | Layer 3 |

---

# Configuring Load Balancing

Enter Global Configuration Mode:

```bash
configure terminal
```

Configure the desired algorithm:

```bash
port-channel load-balance src-dst-mac
```

Example options:

```bash
port-channel load-balance src-mac
port-channel load-balance dst-mac
port-channel load-balance src-dst-mac
port-channel load-balance src-ip
port-channel load-balance dst-ip
port-channel load-balance src-dst-ip
```

> Cisco uses the term **Port-Channel** because EtherChannel is represented as a logical Port-Channel interface.

---

# Verify the Configuration

```bash
show etherchannel load-balance
```

Example:

```text
Operational State:
src-dst-mac
```

Always follow this workflow:

1. Configure
2. Verify

Never assume the configuration has been applied correctly.

---

# Configure Both Ends

EtherChannel operates between two switches.

If the load-balancing algorithm is changed on one switch, configure the same algorithm on the peer switch.

```
Switch A
↓

src-dst-mac

↓

Switch B

src-dst-mac
```

Benefits:

- Consistent traffic distribution
- Easier troubleshooting
- Predictable link utilization

---

# Selecting the Best Algorithm

Choose the algorithm based on traffic characteristics.

### Many Clients → One Server

Recommended:

- `src-dst-ip`
- `src-dst-mac`

---

### Layer 2 Networks

Recommended:

- `src-dst-mac`

---

### Routed Layer 3 Networks

Recommended:

- `src-dst-ip`

---

### Server-to-Server Traffic

Recommended:

- `src-dst-ip`

---

# Useful Verification Commands

## View EtherChannel Summary

```bash
show etherchannel summary
```

Displays:

- Port-Channel status
- Member interfaces
- LACP/PAgP mode

---

## View Port-Channel Information

```bash
show etherchannel port-channel
```

Displays:

- Logical interface details
- Port-Channel configuration

---

## View Current Load-Balancing Algorithm

```bash
show etherchannel load-balance
```

---

# Best Practices

- Understand that EtherChannel increases **aggregate bandwidth**, not single-flow bandwidth.
- Expect one conversation to remain on one physical interface.
- Use multiple traffic flows to utilize all bundled links.
- Select a hashing algorithm that matches actual traffic patterns.
- Configure identical load-balancing methods on both switches.
- Always verify configurations after making changes.
- Base tuning decisions on network traffic rather than relying on default settings.

---

# Key Takeaways

- EtherChannel is a logical interface composed of multiple physical links.
- One traffic flow always uses one physical link.
- Multiple independent flows are distributed across available links.
- Traffic distribution is determined using a deterministic hash algorithm.
- Hash inputs can include MAC addresses or IP addresses.
- The default algorithm may not be optimal for every environment.
- Use `port-channel load-balance` to change the hashing method.
- Verify the operational algorithm with `show etherchannel load-balance`.
- Configure both ends of the EtherChannel consistently.
- Proper load-balancing selection improves bandwidth utilization and network performance.