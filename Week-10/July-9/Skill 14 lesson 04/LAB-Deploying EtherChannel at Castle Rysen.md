# Deploying EtherChannel at Castle Rysen Coffee - Lab

## Objective

Configure LACP EtherChannels between:

- Cafe SW1 ↔ Cafe SW2
- FO-SW01 ↔ FO-SW02

Verify:

- EtherChannel operation
- STP behavior
- Load balancing

---

# Part 1 – Verify Trunk Interfaces

Verify the interfaces intended for EtherChannel.

```
show interfaces trunk
```

Verified:

```
Fa0/1
Fa0/2
```

Both interfaces had:

- Trunk mode
- Native VLAN 1
- Allowed VLANs 1-1005
- Active VLANs 1,10,20

---

# Part 2 – Select LACP

Configure the EtherChannel protocol.

```
interface range fa0/1-2

channel-protocol lacp
```

Verification:

```
show etherchannel summary
```

Output:

```
Number of channel-groups in use: 0
```

Selecting the protocol alone does not create an EtherChannel.

---

# Part 3 – Create EtherChannel

Configure:

```
interface range fa0/1-2

channel-group 1 mode active
```

Initial output:

```
%EC-5-L3DONTBNDL2

LACP currently not enabled on the remote port.
```

Reason:

The remote switch had not yet been configured for LACP.

Cisco temporarily suspended the interfaces until both sides matched.

---

# Part 4 – Configure Remote Switch

Repeat the configuration.

```
interface range fa0/1-2

channel-group 1 mode active
```

The Port-Channel interface was created automatically.

```
Creating Port-channel1
```

---

# Part 5 – Verify EtherChannel

```
show etherchannel summary
```

Output:

```
Po1(SU)

Fa0/1(P)

Fa0/2(P)
```

Meaning:

- Layer 2 EtherChannel
- Port-channel operational
- Both interfaces participating

---

# EtherChannel Flag Interpretation

```
Po1(SU)
```

S

- Layer 2 EtherChannel

U

- Operational

```
Fa0/1(P)

Fa0/2(P)
```

P

- Interface actively participating

---

# Negotiation Process

During LACP negotiation:

```
Po1(SD)

Fa0/1(I)

Fa0/2(I)
```

Meaning:

- Port-channel exists
- Down
- Interfaces operating independently

After successful negotiation:

```
Po1(SU)

Fa0/1(P)

Fa0/2(P)
```

The EtherChannel becomes operational.

---

# Fallout Shelter Deployment

Interfaces:

```
Fa0/6
Fa0/7
```

Configuration:

```
interface range fa0/6-7

channel-group 1 mode active
```

Verification:

```
show etherchannel summary
```

Output:

```
Po1(SU)

Fa0/6(P)

Fa0/7(P)
```

Both interfaces successfully joined the bundle.

---

# STP Verification

Command:

```
show spanning-tree
```

Output:

```
Po1

Role: Designated

State: Forwarding

Cost: 12
```

Observation:

Before EtherChannel:

```
Fa0/6

Fa0/7

↓

One interface blocked
```

After EtherChannel:

```
Port-channel1

↓

Forwarding
```

STP now treats the EtherChannel as a single logical path.

---

# Load Balancing Verification

Check current algorithm.

```
show etherchannel load-balance
```

Default:

```
src-mac
```

Meaning:

Traffic is distributed using only the source MAC address.

---

# Configure Better Load Balancing

```
configure terminal

port-channel load-balance src-dst-mac
```

Verify:

```
show etherchannel load-balance
```

Output:

```
src-dst-mac
```

Meaning:

Traffic is distributed using:

- Source MAC
- Destination MAC

This provides more even traffic distribution across member interfaces.

---

# Commands Used

Verify trunk:

```
show interfaces trunk
```

Configure LACP:

```
channel-protocol lacp
```

Create EtherChannel:

```
channel-group 1 mode active
```

Verify EtherChannel:

```
show etherchannel summary
```

Verify STP:

```
show spanning-tree
```

Verify load balancing:

```
show etherchannel load-balance
```

Configure load balancing:

```
port-channel load-balance src-dst-mac
```

---

# Lab Outcome

Successfully completed:

- Verified trunk interface consistency.
- Configured LACP on both switches.
- Created Layer 2 EtherChannels.
- Verified operational Port-Channels.
- Observed STP recognizing the Port-Channel instead of individual interfaces.
- Verified and modified the EtherChannel load-balancing algorithm.
- Confirmed successful EtherChannel deployment in both the Cafe and Fallout Shelter topologies.