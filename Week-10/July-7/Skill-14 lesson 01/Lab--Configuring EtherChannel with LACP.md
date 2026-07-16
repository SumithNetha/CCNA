# Lab: Configuring EtherChannel with LACP

## Lab Objective

In this lab, two Cisco switches are connected using two FastEthernet links. Initially, **Rapid PVST (RSTP)** blocks one of the redundant links to prevent a Layer 2 loop. The goal is to configure **EtherChannel** so that both physical links become a **single logical Port-Channel**, allowing both links to forward traffic while maintaining loop prevention.

---

# Network Topology

```text
                 cafe01-sw1 (Root Bridge)
              +-------------------------+
              |                         |
         Fa0/1|=========================|Fa0/1
              |                         |
         Fa0/2|=========================|Fa0/2
              |                         |
              +-------------------------+
                    cafe01-sw02

           Two FastEthernet trunk links
```

---

# Initial STP State

## cafe01-sw1 (Root Bridge)

```text
Fa0/1   Designated   Forwarding
Fa0/2   Designated   Forwarding
```

Since `cafe01-sw1` is the Root Bridge, all its switch-to-switch ports are **Designated Ports** and remain in the **Forwarding** state.

---

## cafe01-sw02 (Non-Root Bridge)

```text
Fa0/1   Root Port        Forwarding
Fa0/2   Alternate Port   Blocking
```

RSTP blocks one redundant path to prevent a switching loop.

---

# Verify STP Before EtherChannel

```cisco
show spanning-tree
```

Expected Output:

```text
cafe01-sw02

Fa0/1  Root FWD

Fa0/2  Altn BLK
```

Only one physical link forwards traffic.

---

# Configure EtherChannel Using LACP

## Step 1: Configure First Switch

```cisco
conf t

interface range fa0/1-2

switchport mode trunk

channel-group 1 mode active

end
```

---

## Step 2: Configure Second Switch

```cisco
conf t

interface range fa0/1-2

switchport mode trunk

channel-group 1 mode active

end
```

---

# Optional Enterprise Best Practice

Disable Dynamic Trunking Protocol (DTP).

```cisco
interface range fa0/1-2

switchport nonegotiate
```

### Why?

- Prevents unnecessary DTP frames.
- Increases security.
- Common enterprise practice.
- Does **not** affect LACP negotiation.

> **Note:** `switchport nonegotiate` disables **DTP only**. It does **not** disable or interfere with LACP.

---

# What Happens During Configuration?

When the EtherChannel is created, Cisco automatically:

1. Removes the interfaces from STP.
2. Creates a logical Port-Channel interface.
3. Adds member interfaces to the bundle.
4. Recalculates STP.
5. Brings the bundle online.

Typical console messages:

```text
Interface FastEthernet0/1 changed state to down

Interface FastEthernet0/1 changed state to up

Interface FastEthernet0/2 changed state to down

Interface FastEthernet0/2 changed state to up

Interface Port-channel1 changed state to up
```

This temporary interface flap is expected.

---

# Verify Running Configuration

```cisco
show running-config
```

Expected configuration:

```cisco
interface Port-channel1
 switchport mode trunk

interface FastEthernet0/1
 switchport mode trunk
 switchport nonegotiate
 channel-group 1 mode active

interface FastEthernet0/2
 switchport mode trunk
 switchport nonegotiate
 channel-group 1 mode active
```

---

# Verify EtherChannel

```cisco
show etherchannel summary
```

Expected (Real Cisco IOS):

```text
Group  Port-channel  Protocol   Ports

1      Po1(SU)       LACP       Fa0/1(P) Fa0/2(P)
```

Meaning:

| Symbol | Meaning |
|---------|---------|
| **Po1** | Port-channel 1 |
| **S** | Layer 2 EtherChannel |
| **U** | In use |
| **P** | Successfully bundled |

---

# Packet Tracer Observation

Packet Tracer may incorrectly display:

```text
Protocol : PAgP
```

even after configuring:

```cisco
channel-group 1 mode active
```

This is a **Packet Tracer display bug**.

The running configuration correctly shows:

```cisco
channel-group 1 mode active
```

which confirms the EtherChannel is configured for **LACP**.

On real Cisco hardware, the protocol would display as:

```text
Protocol : LACP
```

---

# Verify STP After EtherChannel

```cisco
show spanning-tree
```

Before EtherChannel:

```text
Fa0/1   Root FWD

Fa0/2   Alternate BLK
```

After EtherChannel:

```text
Po1   Root FWD
```

STP now sees one logical interface instead of two physical interfaces.

---

# STP Cost Comparison

Before EtherChannel:

```text
Cost = 19
```

After EtherChannel:

```text
Cost = 12
```

### Why?

Bundling two FastEthernet interfaces doubles the available bandwidth, resulting in a lower STP path cost.

Lower STP cost indicates a more preferred path.

---

# Understanding the Port-Channel

Internally:

```text
Fa0/1
Fa0/2

      │
      ▼

Port-channel1
```

The Port-Channel is a logical interface.

STP, VLANs, and trunking operate on the Port-Channel rather than individual member interfaces.

---

# Configuration Best Practices

- Use **LACP** (`mode active`) instead of PAgP whenever possible.
- Configure the same EtherChannel mode on both ends.
- Configure all member interfaces with identical settings.
- Apply VLAN and trunk configurations to the **Port-Channel**.
- Verify the EtherChannel after configuration.
- Perform EtherChannel changes during maintenance windows because member interfaces briefly go down and come back up.

---

# Troubleshooting

## EtherChannel Does Not Form

Possible causes:

- Different channel-group numbers
- LACP on one side and PAgP on the other
- Different trunk/access modes
- VLAN mismatch
- Speed mismatch
- Duplex mismatch
- Different native VLANs
- Different allowed VLAN lists

---

## Error Example

```text
Fa0/1 suspended:

LACP currently not enabled on the remote port.
```

Meaning:

The remote switch is not configured for LACP or the EtherChannel protocols do not match.

---

# Commands Used

```cisco
show spanning-tree

show etherchannel summary

show etherchannel

show running-config

interface range fa0/1-2

channel-group 1 mode active

switchport mode trunk

switchport nonegotiate
```

---

# Lab Summary

- Verified that **RSTP blocked one redundant link** before EtherChannel.
- Configured **LACP EtherChannel** using `channel-group 1 mode active`.
- Observed Cisco automatically create **Port-channel1**.
- Verified that **STP replaced the physical interfaces with Po1**, treating the EtherChannel as a single logical path.
- Confirmed the STP path cost decreased from **19** to **12**, reflecting increased aggregate bandwidth.
- Verified the running configuration showed `mode active` on both switches.
- Noted a **Packet Tracer limitation** where `show etherchannel` incorrectly reported **PAgP** instead of **LACP**, even though the configuration was correct.
- Learned that `switchport nonegotiate` disables **DTP only** and does not affect LACP negotiation.

---

# CCNA Exam Points

- **LACP** is the IEEE standard EtherChannel negotiation protocol (IEEE 802.1AX).
- `channel-group <number> mode active` configures **LACP Active mode**.
- STP treats the entire EtherChannel as **one logical interface**.
- Configure VLANs and trunking on the **Port-Channel**.
- All member interfaces must have identical configurations.
- `switchport nonegotiate` disables **DTP**, not LACP.
- EtherChannel improves bandwidth utilization while maintaining redundancy and loop prevention.