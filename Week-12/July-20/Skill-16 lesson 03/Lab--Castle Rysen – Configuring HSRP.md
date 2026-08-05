# Lab: Castle Rysen – Configuring HSRPv2

## Lab Objective

In this lab, the Fallout Shelter network is upgraded from a **single-router default gateway** to a **highly available redundant gateway** using **HSRP Version 2 (HSRPv2)**.

Before configuring HSRP, the network is prepared by configuring:

- Second Edge Router (FO-RT02)
- Second Internet Connection
- Router-on-a-Stick
- VLAN Trunking
- OSPF
- NAT Overload

After the infrastructure is ready, HSRPv2 is configured to provide a virtual default gateway for all VLANs, followed by testing failover and failback.

---

# Network Topology

The lab consists of:

- 2 Edge Routers
  - FO-RT01 (Primary)
  - FO-RT02 (Secondary)

- 2 Distribution Switches

- 4 Access Switches

- Public Internet Router

- Multiple VLANs

| VLAN | Purpose | Network |
|-------|----------|---------------|
| 10 | Management | 10.0.16.0/25 |
| 20 | Internal | 10.0.16.128/25 |
| 30 | Video | 10.0.17.0/25 |
| 40 | Guest | 10.0.17.128/25 |

---

# Initial Network State

Before HSRP:

- FO-RT01 was the only default gateway.
- Clients used FO-RT01's interface IP as their gateway.
- FO-RT02 was added to provide redundancy.
- If FO-RT01 failed, all clients would lose connectivity.

The objective is to eliminate this single point of failure.

---

# Step 1 – Configure the Second Edge Router

Configure FO-RT02 with:

- Internet-facing interface
- Router-on-a-Stick
- Default Route
- OSPF
- NAT

Purpose:

Prepare FO-RT02 so it can immediately take over if FO-RT01 fails.

---

# Step 2 – Configure Router-on-a-Stick

Create subinterfaces for every VLAN.

Configure:

- VLAN encapsulation
- IP addresses
- NAT inside interfaces

Each VLAN receives its own Layer-3 gateway interface.

---

# Step 3 – Enable the Physical Interface

Initially,

the subinterfaces remain down because the parent interface is administratively down.

Bring the parent interface online.

Result:

All VLAN subinterfaces transition to an operational state.

---

# Step 4 – Verify Trunk Links

Verify that trunk ports carry:

- VLAN 10
- VLAN 20
- VLAN 30
- VLAN 40

Purpose:

Router-on-a-Stick requires tagged VLAN traffic from the switches.

Without trunking, inter-VLAN routing cannot occur.

---

# Step 5 – Configure OSPF

Configure OSPF on FO-RT02.

Advertise:

- VLAN Networks
- Internet-facing Network

Verify:

- Neighbor relationships
- Routing table
- Learned routes

Purpose:

Both routers must exchange routing information before implementing HSRP.

---

# Step 6 – Configure NAT

Configure NAT Overload.

Configure:

- Inside Interfaces
- Outside Interface
- ACL
- NAT Overload

Verify:

- NAT translations
- Internet connectivity

Purpose:

During failover, whichever router becomes Active must still translate private addresses.

---

# Step 7 – Prepare for HSRP

Before HSRP:

Each router owns the gateway IP.

Example:

| Device | IP Address |
|----------|------------|
| FO-RT01 | 10.0.16.1 |
| FO-RT02 | 10.0.16.3 |

After HSRP:

| Device | Address |
|----------|-------------|
| Virtual Gateway | 10.0.16.1 |
| FO-RT01 | 10.0.16.2 |
| FO-RT02 | 10.0.16.3 |

The gateway IP no longer belongs to a physical router.

Instead,

it belongs to HSRP.

---

# Step 8 – Configure HSRPv2

Configure HSRP Version 2 on every VLAN subinterface.

Configure:

- HSRP Version
- Group Number
- Virtual IP
- Priority
- Preempt

Design:

- VLAN Number = HSRP Group Number

Example:

| VLAN | HSRP Group |
|------|------------|
| 10 | 10 |
| 20 | 20 |
| 30 | 30 |
| 40 | 40 |

---

# Step 9 – Configure Router Priority

FO-RT01

Priority:

```
105
```

FO-RT02

Priority:

```
100
```

Result:

FO-RT01 becomes the Active Router.

FO-RT02 remains Standby.

---

# Step 10 – Configure Preempt

Enable Preempt.

Purpose:

When FO-RT01 recovers after a failure,

it automatically resumes the Active role.

Without Preempt,

the backup router would continue forwarding traffic.

---

# Step 11 – Verify HSRP

Verify:

- Active Router
- Standby Router
- Group Numbers
- Priority
- Virtual IP

Confirm:

- FO-RT01 is Active.
- FO-RT02 is Standby.

---

# Step 12 – Verify Client Gateway

Clients continue using:

```
10.0.16.1
```

No client configuration changes are required.

This demonstrates one of HSRP's major advantages.

---

# Step 13 – Verify Virtual MAC

Check the client's ARP table.

Observe:

The client learns the HSRP Virtual MAC Address,

not the physical MAC address of either router.

Purpose:

The gateway appears as one logical device.

---

# Step 14 – Test Internet Connectivity

Verify:

- Client can reach Internet.
- NAT translations exist.
- OSPF routes are present.

Everything should operate through FO-RT01.

---

# Step 15 – Test Failover

Simulate failure by disconnecting FO-RT01 from the network.

Observe:

- Small packet loss
- HSRP election
- FO-RT02 becomes Active
- NAT moves to FO-RT02
- Internet connectivity resumes

Users continue using the same default gateway.

---

# Step 16 – Test Failback

Restore FO-RT01.

Observe:

- OSPF neighbors recover.
- Preempt activates.
- FO-RT01 resumes the Active role.
- FO-RT02 returns to Standby.

The network automatically returns to its original design.

---

# Expected Results

After completing the lab:

- Both routers participate in HSRP.
- FO-RT01 operates as Active.
- FO-RT02 operates as Standby.
- Clients always use the Virtual Gateway.
- Internet connectivity survives router failure.
- NAT functions on both routers.
- OSPF exchanges routes correctly.
- Automatic failover occurs.
- Automatic failback occurs after recovery.

---

# Verification Checklist

- Router-on-a-Stick configured
- Trunk links operational
- OSPF neighbors established
- Routing tables populated
- NAT functioning
- HSRP Version 2 configured
- Virtual IP reachable
- Active/Standby roles verified
- Virtual MAC learned by clients
- Internet reachable
- Failover tested successfully
- Failback tested successfully

---

# Key Learning Outcomes

- Prepared an enterprise network for gateway redundancy.
- Configured HSRPv2 on redundant edge routers.
- Understood the relationship between HSRP, OSPF, NAT, and VLANs.
- Verified Active and Standby router operation.
- Observed Virtual IP and Virtual MAC functionality.
- Tested real failover and failback scenarios.
- Demonstrated seamless default gateway redundancy for end devices.





# Step 1 - Verify Existing Network

## Objective

Verify the current network before adding HSRPv2.

### Command

```cisco
show ip interface brief
```

### Expected Result

```
Interface              IP-Address      OK? Method Status Protocol

Fa0/0                  unassigned      YES manual up     up
Fa0/0.10               10.0.16.2       YES manual up     up
Fa0/0.20               10.0.16.130     YES manual up     up
Fa0/0.30               10.0.17.2       YES manual up     up
Fa0/0.40               10.0.17.130     YES manual up     up
Fa0/1                  216.0.5.2       YES manual up     up
```

### Result

All router interfaces are operational.

The router is ready for HSRP configuration.

---

# Step 2 - Verify OSPF

## Command

```cisco
show ip ospf neighbor
```

### Expected Result

```
Neighbor ID     Pri   State     Dead Time    Address

2.2.2.2           1   FULL/-    00:00:36     216.0.5.3
```

### Result

OSPF adjacency is established successfully.

Both routers can exchange routing information.

---

# Step 3 - Verify NAT

## Command

```cisco
show ip nat translations
```

### Expected Result

```
Pro Inside global      Inside local

216.0.5.2              10.0.16.10
216.0.5.2              10.0.16.20
```

### Result

Private addresses are successfully translated to public addresses.

NAT is functioning correctly.

---

# Step 4 - Configure HSRPv2

## Commands

```cisco
interface Fa0/0.10

 standby version 2
 standby 10 ip 10.0.16.1
 standby 10 priority 105
 standby 10 preempt
```

### Result

HSRP Group 10 is created.

Router becomes Active because it has the highest priority.

---

# Step 5 - Verify HSRP

## Command

```cisco
show standby brief
```

### Expected Result

```
Interface   Grp Pri P State

Fa0/0.10    10 105 Active
Fa0/0.20    20 105 Active
Fa0/0.30    30 105 Active
Fa0/0.40    40 105 Active
```

### Result

FO-RT01 is operating as the Active Router.

---

# Step 6 - Verify ARP

## Command (PC)

```bash
arp -a
```

### Expected Result

```
Internet Address      Physical Address

10.0.16.1             0000.0c9f.f00a
```

### Result

The PC learns the HSRP Virtual MAC instead of the router's physical MAC.

---

# Step 7 - Test Failover

## Action

Shutdown the switch port connected to FO-RT01.

```cisco
interface Fa0/1

shutdown
```

### Verify

```cisco
show standby brief
```

### Expected Result

```
FO-RT02

State

Active
```

### Result

Router 2 automatically becomes the Active Router.

Internet connectivity is maintained.

---

# Step 8 - Verify NAT After Failover

## Command

```cisco
show ip nat translations
```

### Expected Result

```
Inside Global

216.0.5.3
```

### Result

NAT translations have moved from FO-RT01 to FO-RT02.

---

# Step 9 - Restore Primary Router

## Action

```cisco
interface Fa0/1

no shutdown
```

### Verify

```cisco
show standby brief
```

### Expected Result

```
FO-RT01

Active

FO-RT02

Standby
```

### Result

Preempt returns FO-RT01 to the Active role automatically.