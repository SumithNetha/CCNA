# Castle Rysen – Implementing OSPF Routing Protocol

> **Skill 15 – Lesson 05 | Week 11 – July 14**

---

# Objective

Implement **OSPF (Open Shortest Path First)** to connect the **District Shop (Cafe)** and **Fallout Shelter** through a WAN link, allowing routers to **automatically exchange routing information** instead of relying on static routes.

---

# Prerequisites

Before this lesson, the following had already been completed:

* IPv4 Addressing
* Subnetting
* VLAN Implementation
* Inter-VLAN Routing
* Static Routing
* Default Routing
* Dynamic Routing Concepts
* OSPF Theory
* OSPF Network Command

This lesson combines all previous topics into one real-world implementation.

---

# Existing Network

## District Shop (Cafe)

| VLAN    | Purpose | Network       |
| ------- | ------- | ------------- |
| VLAN 10 | Admin   | 10.0.18.0/27  |
| VLAN 20 | Patron  | 10.0.18.32/27 |

---

## Fallout Shelter

| VLAN    | Purpose    | Network        |
| ------- | ---------- | -------------- |
| VLAN 10 | Management | 10.0.16.0/25   |
| VLAN 20 | Internal   | 10.0.16.128/25 |
| VLAN 30 | Video      | 10.0.17.0/25   |
| VLAN 40 | Guest      | 10.0.17.128/25 |

Initially, these two locations operate independently and have **no dynamic routing** between them.

---

# Lesson Flow

```text
Existing Networks
        │
        ▼
Create WAN Link
        │
        ▼
Verify WAN Connectivity
        │
        ▼
Remove Old Routing Protocol
        │
        ▼
Enable OSPF
        │
        ▼
Advertise Required Networks
        │
        ▼
Configure Passive Interface
        │
        ▼
Form OSPF Neighbor
        │
        ▼
Verify Routes
        │
        ▼
Troubleshoot Configuration Mistake
        │
        ▼
Successful Dynamic Routing
```

---
# Step 0 – Prepare the Routers for the WAN Connection

Before configuring OSPF, the routers must have a **Serial interface** available. The default Packet Tracer routers used in this lab do not include one, so an expansion module must be installed.

## Why is this required?

The Cafe Router and Fallout Router are located at different sites.

To simulate a **WAN (Wide Area Network)** between them, Jeremy uses a **Serial connection**.

However, the routers initially only have Ethernet interfaces.

Therefore, we must install a Serial HWIC module.

---

## Install the HWIC-2T Module

Perform the following steps on **both routers**.

### 1. Open the Router

Click the router in Packet Tracer.

Go to

```text
Physical Tab
```

---

### 2. Turn Off the Router

Move the power switch to **OFF**.

> **Important**
>
> Packet Tracer does not allow hardware modules to be installed while the router is powered on.

---

### 3. Install the Serial Module

Drag the following module into an empty HWIC slot.

```text
HWIC-2T
```

**HWIC** stands for:

> **High-Speed WAN Interface Card**

The **2T** means the module provides:

* 2 Serial Interfaces

After installation, the router gains:

```text
Serial0/1/0

Serial0/1/1
```

(or similar numbering depending on the router model.)

---

### 4. Turn the Router Back On

Move the power switch back to **ON**.

Wait for the IOS to boot.

The router now recognizes the new hardware.

---

### 5. Verify the Interfaces

Run:

```cisco
show ip interface brief
```

Expected output:

```text
Serial0/1/0

Serial0/1/1
```

Initially they appear as:

```text
unassigned

down

down
```

This is expected because:

* No IP address has been configured.
* The interfaces are not yet connected or enabled.

---

## Connect Both Routers

Use a **Serial DCE cable** to connect:

```text
Cafe Router
Serial0/1/0
        │
        │
Serial DCE Cable
        │
        │
Fallout Router
Serial0/0/0
```

> **Note:** In Packet Tracer, the first router you click with the Serial DCE cable becomes the **DCE** side. The DCE end provides clocking. In your lab, the Cafe router showed `clock rate 2000000` configured on its serial interface, indicating it was the DCE side.

---

## Why Use a Serial Connection?

Jeremy uses a Serial link because it is a simple way to simulate a leased WAN circuit in Packet Tracer.

Real-world WAN technologies today often use:

* MPLS
* Metro Ethernet
* Fiber
* SD-WAN
* Ethernet handoffs from ISPs

Serial links are still taught in CCNA because they clearly demonstrate point-to-point WAN concepts.

---

## Lab Flow

```text
Open Router
      ↓
Power Off
      ↓
Install HWIC-2T
      ↓
Power On
      ↓
Verify Serial Interfaces
      ↓
Connect Serial Cable
      ↓
Assign IP Addresses
      ↓
Verify Connectivity
      ↓
Configure OSPF
      ↓
Verify Neighbor
      ↓
Verify Routes
```

---

# Step 1 – Decide What Should Be Advertised

Jeremy intentionally **does not advertise every network**.

## Cafe

Advertise

* Admin VLAN (10.0.18.0/27)

Do Not Advertise

* Patron VLAN

### Why?

Patron devices only require:

* Internet
* Guest Services

They do **not** need direct communication with the Fallout Shelter.

> **Real World**
>
> Never advertise networks simply because they exist. Only advertise networks that must be reachable from other sites. This follows the **Principle of Least Privilege** and reduces unnecessary routing information.

---

# Step 2 – Build the WAN Link

A point-to-point WAN is created between the two routers.

Network:

```text
172.16.0.0/30
```

| Device    | IP Address |
| --------- | ---------- |
| FO-RT01   | 172.16.0.1 |
| Cafe-RT01 | 172.16.0.2 |

---

## Why /30?

A point-to-point connection needs only **two usable addresses**.

```text
Network      172.16.0.0
Router A     172.16.0.1
Router B     172.16.0.2
Broadcast    172.16.0.3
```

No IP addresses are wasted.

### CCNA Note

Traditionally, Cisco uses **/30** for WAN links.

Modern networks may also use **/31** on point-to-point links (RFC 3021), but CCNA primarily teaches **/30**.

---

# Step 3 – Configure the WAN Interfaces

### Cafe Router

```cisco
interface Serial0/1/0
 ip address 172.16.0.2 255.255.255.252
 no shutdown
```

### Fallout Router

```cisco
interface Serial0/0/0
 ip address 172.16.0.1 255.255.255.252
 no shutdown
```

---

## Verify

```cisco
show ip interface brief
```

Expected:

```text
Serial Interface

Status    up
Protocol  up
```

### Understanding Interface States

| Status                | Meaning                       |
| --------------------- | ----------------------------- |
| down/down             | Physical issue                |
| administratively down | Interface disabled (shutdown) |
| up/down               | Layer 2 problem               |
| up/up                 | Interface fully operational ✅ |

---

# Step 4 – Verify Layer 3 Connectivity

Before configuring OSPF, verify IP connectivity.

```cisco
ping 172.16.0.1
```

or

```cisco
ping 172.16.0.2
```

Expected:

```text
!!!!!

Success rate 100%
```

Each **!** represents one successful ICMP reply.

### Best Practice

Always verify connectivity **before enabling any routing protocol**.

```text
Physical Link
      ↓
IP Connectivity
      ↓
Ping Success
      ↓
Configure OSPF
```

---

# Step 5 – Remove Previous Routing Protocol

Earlier lessons used EIGRP.

Remove it:

```cisco
no router eigrp 1
```

### Why?

Using multiple routing protocols during learning makes troubleshooting difficult.

For this lab:

```text
Only OSPF
```

should be responsible for dynamic routing.

---

# Step 6 – Start the OSPF Process

```cisco
router ospf 1
```

This creates **OSPF Process ID 1**.

### Important

The Process ID is **locally significant**.

Example:

Router A

```text
router ospf 1
```

Router B

```text
router ospf 100
```

Still works.

Neighboring routers **do not need matching Process IDs**.

---

# Step 7 – Advertise the Cafe Admin Network

```cisco
network 10.0.18.1 0.0.0.0 area 0
```

This command performs **two jobs**.

### Job 1

Find the interface with IP

```text
10.0.18.1
```

Enable OSPF on that interface.

---

### Job 2

Advertise its connected network

```text
10.0.18.0/27
```

to OSPF neighbors.

---

## Understanding the Wildcard

```text
0.0.0.0
```

means

> Match every bit exactly.

OSPF searches all interfaces.

```text
Does interface IP equal 10.0.18.1?

YES

↓

Enable OSPF
```

---

### Important Clarification

The **network command does NOT advertise the IP address**.

It advertises the **connected network** of the matched interface.

Example

```text
Matched Interface

10.0.18.1/27

↓

Advertised

10.0.18.0/27
```

---

# Step 8 – Enable OSPF on the WAN

```cisco
network 172.16.0.2 0.0.0.0 area 0
```

Purpose:

* Enable OSPF on the Serial interface
* Send Hello packets
* Form neighbor relationship

Without enabling OSPF on the WAN interface, routers can never discover each other.

---

# Step 9 – Configure Passive Interface

```cisco
passive-interface GigabitEthernet0/0.10
```

Effect:

* Admin network is still advertised.
* No OSPF Hello packets are sent toward PCs.

### Why?

End devices cannot become OSPF neighbors.

Sending Hello packets toward user devices wastes CPU and bandwidth.

---

# Step 10 – Configure OSPF on the Fallout Router

Configure:

```cisco
router ospf 1
```

Advertise:

* WAN Interface
* Local Fallout Networks

Initially, Jeremy advertises only the WAN network to establish the neighbor relationship first.

Later, he advertises all VLANs using a broader network statement.

---

# Step 11 – Verify Neighbor Relationship

```cisco
show ip ospf neighbor
```

Initially

```text
No Neighbor
```

This indicates that OSPF adjacency has not formed.

---

# Step 12 – Troubleshooting

Enable debugging.

```cisco
debug ip ospf adj

debug ip ospf events
```

Debug displays:

* Hello packets
* Neighbor changes
* Database Description (DBD)
* Link-State Advertisements (LSAs)
* State transitions

---

# Step 13 – Configuration Mistake

Jeremy intentionally demonstrates a common mistake.

He configured

```cisco
network 172.16.0.1 0.0.0.0 area 0
```

on the **Cafe Router**.

However,

Cafe Router's Serial interface is

```text
172.16.0.2
```

Since

```text
0.0.0.0
```

requires an **exact IP match**,

OSPF could not find the interface.

Result

* No Hello packets
* No Neighbor
* No Route Exchange

---

# Step 14 – Correct the Mistake

Remove

```cisco
no network 172.16.0.1 0.0.0.0 area 0
```

Add

```cisco
network 172.16.0.2 0.0.0.0 area 0
```

Immediately,

OSPF begins exchanging Hello packets.

---

# Step 15 – OSPF Neighbor Formation

The debug output shows the complete adjacency process.

```text
Down
   ↓
Init
   ↓
2-Way
   ↓
ExStart
   ↓
Exchange
   ↓
Loading
   ↓
FULL
```

### Meaning of Each State

| State    | Description                             |
| -------- | --------------------------------------- |
| Down     | No communication                        |
| Init     | Hello received                          |
| 2-Way    | Bidirectional communication established |
| ExStart  | Master/Slave election                   |
| Exchange | Database Description packets exchanged  |
| Loading  | Missing LSAs requested                  |
| FULL     | Databases synchronized ✅                |

---

# Step 16 – Verify Neighbor

```cisco
show ip ospf neighbor
```

Expected

```text
State

FULL
```

This confirms the routers are fully synchronized.

---

# Step 17 – Verify the Routing Table

```cisco
show ip route
```

Before OSPF

Cafe knew only:

```text
10.0.18.0/27
10.0.18.32/27
```

After OSPF

```text
O 10.0.16.0/25
O 10.0.16.128/25
O 10.0.17.0/25
O 10.0.17.128/25
```

The letter

```text
O
```

means

> Learned through OSPF.

Similarly,

Fallout learns

```text
O 10.0.18.0/27
```

---

# Step 18 – Advertise Multiple Networks

Instead of advertising each subnet separately,

Jeremy demonstrates

```cisco
network 10.0.16.0 0.0.1.255 area 0
```

This single command matches

* 10.0.16.0/25
* 10.0.16.128/25
* 10.0.17.0/25
* 10.0.17.128/25

### Advantage

* Simpler configuration

### Disadvantage

May advertise more networks than intended.

Always advertise only the networks that other routers actually need.

---

# Verification Commands

```cisco
show ip interface brief

show ip ospf neighbor

show ip route

show running-config

show ip ospf interface

debug ip ospf adj

debug ip ospf events
```

---

# Common Mistakes

❌ Wrong IP in `network` statement

❌ Forgetting `no shutdown`

❌ OSPF not enabled on WAN interface

❌ Wrong wildcard mask

❌ Wrong Area number

❌ Assuming the Process ID must match

❌ Forgetting to verify with `show ip ospf neighbor`

---

# Key Takeaways

* OSPF dynamically exchanges routes between routers.
* A `/30` subnet is ideal for point-to-point WAN links.
* The `network` command both **enables OSPF on an interface** and **advertises its connected network**.
* `0.0.0.0` is an **exact-match wildcard mask**.
* `passive-interface` advertises the network while suppressing Hello packets on that interface.
* OSPF neighbors must reach the **FULL** state before routes are exchanged.
* The routing table marks OSPF-learned routes with the code **O**.
* Troubleshooting is a core networking skill—Jeremy intentionally included a configuration mistake to demonstrate how OSPF debugging and verification commands help identify and fix real-world issues.
