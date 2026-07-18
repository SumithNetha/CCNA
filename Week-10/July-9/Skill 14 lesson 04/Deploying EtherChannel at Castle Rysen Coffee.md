# Deploying EtherChannel at Castle Rysen Coffee

## Objective

Deploy an EtherChannel using **LACP** to combine multiple physical links into a single logical interface, improving bandwidth utilization and redundancy while allowing STP to maintain loop prevention.

---

# Why EtherChannel Matters

Without EtherChannel:

- STP detects redundant Layer 2 links.
- One redundant link is placed into the Blocking state.
- Only one physical link forwards traffic.
- Available bandwidth is wasted.

With EtherChannel:

- Multiple physical links become one logical interface (Port-Channel).
- STP sees a single path instead of multiple redundant links.
- All member links actively forward traffic.
- Bandwidth and redundancy both improve.

> EtherChannel doesn't replace STP—it changes how STP views redundant links.

---

# EtherChannel Benefits

- Combines multiple physical interfaces into one logical interface.
- Increases aggregate bandwidth.
- Provides link redundancy.
- Eliminates unnecessary STP blocking.
- Simplifies network management.
- Supports Layer 2 and Layer 3 deployments.

---

# Pre-Deployment Checklist

Before configuring an EtherChannel, verify all member interfaces have identical configurations.

Check:

- Interface speed
- Duplex settings
- Access/Trunk mode
- Native VLAN
- Allowed VLAN list
- VLAN assignments
- Switchport negotiation settings

Any mismatch may prevent the EtherChannel from forming.

---

# Troubleshooting Tip

If an EtherChannel fails to establish, verify:

- Trunk configuration
- Allowed VLANs
- Native VLAN
- Speed
- Duplex
- Existing channel-group configuration
- Interface consistency

Most EtherChannel issues are caused by mismatched interface configurations rather than problems with LACP itself.

---

# LACP

LACP (Link Aggregation Control Protocol)

- IEEE 802.3ad / 802.1AX standard
- Vendor independent
- Preferred over Cisco's proprietary PAgP

---

# LACP Modes

## Active

- Initiates LACP negotiation.
- Sends LACP packets.

```
channel-group 1 mode active
```

## Passive

- Waits for LACP packets.
- Does not initiate negotiation.

```
channel-group 1 mode passive
```

### Valid Combinations

| Side A | Side B | Result |
|---------|---------|--------|
| Active | Active | ✅ Forms EtherChannel |
| Active | Passive | ✅ Forms EtherChannel |
| Passive | Passive | ❌ No EtherChannel |

Using **Active** on both sides ensures both switches actively negotiate the bundle.

---

# EtherChannel Deployment Workflow

1. Verify interface configurations.
2. Verify trunk and VLAN configuration.
3. Configure LACP.
4. Create the Port-Channel.
5. Verify EtherChannel status.
6. Verify STP.
7. Configure load balancing if required.

---

# EtherChannel Verification

Primary verification command:

```
show etherchannel summary
```

Displays:

- Port-channel number
- Protocol
- Member interfaces
- Operational status

---

# EtherChannel Status Flags

## Port-Channel Flags

| Flag | Meaning |
|------|---------|
| S | Layer 2 EtherChannel |
| R | Layer 3 EtherChannel |
| U | Port-channel Up and in use |
| D | Down |

---

## Member Interface Flags

| Flag | Meaning |
|------|---------|
| P | Interface is participating in the bundle |
| I | Stand-alone interface |
| s | Suspended |
| w | Waiting to aggregate |
| u | Unsuitable for bundling |

Example:

```
Po1(SU)
Fa0/1(P)
Fa0/2(P)
```

Means:

- Layer 2 EtherChannel
- Operational
- Both interfaces actively forwarding

---

# STP After EtherChannel

Before EtherChannel:

```
Fa0/1
Fa0/2

↓

STP blocks one interface
```

After EtherChannel:

```
Port-channel1

↓

STP sees one logical link
```

Benefits:

- No blocked redundant member interfaces.
- Loop prevention remains intact.
- Aggregate bandwidth becomes available.

---

# EtherChannel Load Balancing

Default:

```
src-mac
```

Traffic is distributed based only on the source MAC address.

Improved option:

```
src-dst-mac
```

Traffic distribution uses both:

- Source MAC
- Destination MAC

This provides more even utilization across member links.

---

# Important Concept

EtherChannel **does not** split one conversation across multiple links.

Instead:

- One conversation remains on one physical link.
- Different conversations are distributed across different member links using a hashing algorithm.

This preserves packet order while improving aggregate bandwidth utilization.

---

# Key Commands

```
show interfaces trunk

show etherchannel summary

show etherchannel load-balance

port-channel load-balance src-dst-mac
```

---

# Deployment Summary

- Verify interface consistency before configuration.
- Use LACP as the preferred EtherChannel protocol.
- Configure member interfaces with identical settings.
- Verify successful bundle creation.
- Confirm STP recognizes the Port-Channel.
- Optimize load balancing when necessary.

EtherChannel transforms redundant backup links into active forwarding paths while maintaining STP loop protection.