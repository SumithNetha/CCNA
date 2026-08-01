# Lab Walkthrough: OSPF Verification and Troubleshooting (Castle Rysen - FO-RT01)

## 🗺️ Lab Topology Overview
This lab focuses on configuring and verifying OSPF on **FO-RT01** (Fallout Shelter #1 Router) within the Castle Rysen enterprise network. 

From the topology, FO-RT01 is the gateway for a localized network that consists of four VLANs:
*   **VLAN 10 (MGMT):** `10.0.16.0/25`
*   **VLAN 20 (INTERNAL):** `10.0.16.128/25`
*   **VLAN 30 (VIDEO):** `10.0.17.0/25`
*   **VLAN 40 (GUEST):** `10.0.17.128/25`

FO-RT01 routes traffic from these VLANs via a "Router-on-a-Stick" configuration (using subinterfaces `Fa0/0.10`, `.20`, `.30`, `.40`). It connects upstream via a **Serial Point-to-Point link** (`172.16.0.0/30`) to **cafe01-RT01** (Router ID `216.0.5.2`).

---

## 🛠️ Step-by-Step Execution and Results

This section documents the exact chronological steps followed in the lab, including the commands issued and the expected outputs.

### Phase 1: Initial OSPF Verification
Before making changes, you verified that OSPF was running and neighbors were established.

**Step 1: Check the overall OSPF process configuration**
*   **Command:** `show ip protocols`
*   **Result:** Confirmed that `ospf 1` is running. The automatically elected Router ID was `172.16.0.1`. It showed routing enabled for the Serial link network (`172.16.0.1 0.0.0.0`) and the VLANs (`10.0.16.0 0.0.1.255`) in Area 0.

**Step 2: Check interface participation and states**
*   **Command:** `show ip ospf interface brief`
*   **Result:** 
    *   Subinterfaces `Fa0/0.10` through `.40` are all active in Area 0. Their state is **DR** (Cost 1) because Ethernet is a broadcast multi-access network type.
    *   Interface `Se0/0/0` is active in Area 0. Its state is **POINT** (Point-to-Point) with a Cost of 64. No DR/BDR election happens here.

**Step 3: Verify OSPF-learned routes**
*   **Command:** `show ip route ospf`
*   **Result:** The routing table showed a route to `10.0.18.0` (District Shop #1) learned via OSPF (`O`) with an administrative distance of `110` and metric `65`, reachable via neighbor `172.16.0.2` out of `Serial0/0/0`.

**Step 4: View the Link-State Database (LSDB)**
*   **Command:** `show ip ospf database`
*   **Result:** Verified that FO-RT01 has LSAs from itself (`172.16.0.1`) and from its neighbor (`216.0.5.2`) in Area 0.

**Step 5: Inspect detailed OSPF interface settings**
*   **Command:** `show ip ospf interface`
*   **Result:** Confirmed network types (`BROADCAST` vs `POINT-TO-POINT`) and verified that the Hello/Dead timers were set to the defaults of **10** and **40** seconds across all interfaces.

**Step 6: Confirm Neighbor Adjacency**
*   **Command:** `show ip ospf neighbor`
*   **Result:** Confirmed a `FULL` adjacency with Router ID `216.0.5.2` on `Serial0/0/0`. The state was `FULL/ -` (the dash indicates no DR/BDR role on this point-to-point link).

---

### Phase 2: Manually Configuring a Custom Router ID
Because the Router ID was automatically selected based on interface IPs, you manually set it to a more recognizable value.

**Step 7: Configure the new Router ID**
*   **Commands:** 
    ```text
    FO-RT01# conf t
    FO-RT01(config)# router ospf 1
    FO-RT01(config-router)# router-id 1.1.1.1
    FO-RT01(config-router)# ^Z (Exit)
    ```
*   **Result:** The router warned: `Reload or use "clear ip ospf process" command, for this to take effect`. OSPF does not apply new Router IDs on the fly to prevent sudden network instability.

**Step 8: Reset the OSPF process**
*   **Command:** `clear ip ospf process` (Type `y` to confirm).
*   **Result:** The syslog immediately reported:
    ```text
    %OSPF-5-ADJCHG: Process 1, Nbr 216.0.5.2 on Serial0/0/0 from FULL to DOWN
    ```
    The neighbor relationship was forced to reset so the new Router ID (`1.1.1.1`) could be used.

---

### Phase 3: Watching OSPF Form Neighbors in Real-Time (Debug)
To see exactly how OSPF transitions through its states, you enabled debugging and reset the process again.

**Step 9: Enable Adjacency Debugging**
*   **Command:** `debug ip ospf adj`
*   **Result:** OSPF adjacency events debugging was turned on.

**Step 10: Trigger the neighbor process again**
*   **Command:** `clear ip ospf process` (Type `y` to confirm).
*   **Result:** The terminal output displayed the entire state machine sequence:

    **1. DR Elections on Ethernet Interfaces:**
    ```text
    OSPF: DR/BDR election on FastEthernet0/0.10
    OSPF: Elect DR 1.1.1.1
    ```
    *Observation:* FO-RT01 immediately elected itself as the Designated Router on all FastEthernet subinterfaces because it is the only OSPF router on those VLANs.

    **2. The ExStart State (Master/Slave Negotiation):**
    ```text
    OSPF: Rcv DBD from 216.0.5.2 ... state EXSTART
    OSPF: NBR Negotiation Done. We are the SLAVE
    ```
    *Observation:* On the Serial link, FO-RT01 and 216.0.5.2 entered ExStart. They compared Router IDs. FO-RT01 (`1.1.1.1`) is lower than `216.0.5.2`, so FO-RT01 became the SLAVE for the exchange process.

    **3. The Exchange State (Trading Summaries):**
    ```text
    OSPF: Rcv DBD from 216.0.5.2 ... state EXCHANGE
    OSPF: Exchange Done with 216.0.5.2
    ```
    *Observation:* The routers traded Database Description (DBD) packets—the "table of contents" of their routing knowledge.

    **4. The Loading State (Requesting Missing Info):**
    ```text
    OSPF: sent LS REQ packet to 224.0.0.5
    ```
    *Observation:* FO-RT01 requested the specific missing Link-State Advertisements it saw in the DBD packet.

    **5. The Full State (Synchronization Complete):**
    ```text
    Synchronized with with 216.0.5.2 on Serial0/0/0, state FULL
    %OSPF-5-ADJCHG: Process 1, Nbr 216.0.5.2 on Serial0/0/0 from LOADING to FULL, Loading Done
    ```
    *Observation:* All databases are identical, and the routers are fully adjacent again!

**Step 11: Turn off debugging**
*   **Command:** `no debug all`
*   **Result:** Disabled debug output to prevent console flooding.


This output is actually **gold** for learning OSPF. Jeremy intentionally had you use the verification commands and then the debug output so you could **watch OSPF think**. Most CCNA students only see the commands—they never understand *why* each line appears.

Let's go through your output exactly as a Cisco engineer would.

---

# Network Topology Overview

From your Packet Tracer screenshots, the topology is:

```text
                    ISP
                     │
              216.0.5.0/24
                     │
                Cafe Router
             Router ID 216.0.5.2
                     │
              172.16.0.0/30
                     │
            FO-RT01 (Router4)
          Router ID 172.16.0.1
                     │
      ------------------------------
      |      |      |             |
   VLAN10 VLAN20 VLAN30       VLAN40
```

There is **only one OSPF neighbor**.

```text
FO-RT01

↓

Cafe Router
```

Everything else are LANs attached directly to FO-RT01.

---

# Step 1

## show ip route ospf

Your output

```text
O 10.0.18.0 [110/65]
via 172.16.0.2
Serial0/0/0
```

Let's decode every field.

---

### O

```text
O
```

Means

```text
Learned via OSPF
```

Cisco route codes

```text
C
Connected

L
Local

S
Static

O
OSPF

D
EIGRP

R
RIP
```

---

### 10.0.18.0

Destination network.

This is the Cafe network.

---

### [110/65]

This confuses almost everyone.

It means

```text
[Administrative Distance / Metric]
```

---

Administrative Distance

```text
110
```

OSPF's default AD.

Cisco uses AD to decide **which routing protocol to trust**.

Example

```text
Static

AD 1

OSPF

AD 110

RIP

AD 120
```

Lower wins.

---

Metric

```text
65
```

OSPF Cost.

Remember

OSPF uses

```text
Cost
```

not hop count.

---

### via 172.16.0.2

Means

"My next hop is"

```text
172.16.0.2
```

Which is

Cafe Router.

---

### Serial0/0/0

Outgoing interface.

So the routing table says

```text
Need 10.0.18.0?

↓

Send packet

↓

Serial0/0/0

↓

172.16.0.2
```

---

# Step 2

## show ip ospf database

Output

```text
Router Link States

172.16.0.1

216.0.5.2
```

Notice

Exactly two Router LSAs.

Why?

Because only two routers exist.

Each router generates

One Type-1 Router LSA.

```text
FO-RT01

↓

Router LSA

Cafe Router

↓

Router LSA
```

That's exactly what you see.

---

# Step 3

## show ip ospf interface

This command is extremely important.

Let's decode one interface.

---

Example

```text
FastEthernet0/0.10

Area 0

RID 172.16.0.1

Network Type BROADCAST

Cost 1
```

---

### Area 0

Means

```text
This interface participates

in

Area 0
```

---

### Router ID

```text
172.16.0.1
```

Notice

This was BEFORE

you changed it

to

```text
1.1.1.1
```

---

### Network Type

```text
Broadcast
```

Because

Ethernet.

---

### Cost

```text
1
```

Ethernet default cost.

---

### State DR

Every FastEthernet interface says

```text
State DR
```

Why?

Because

There are

no other routers

on those VLANs.

Only switches.

Since OSPF sees

only itself

it automatically becomes

Designated Router.

Notice

```text
Neighbor Count

0
```

Exactly.

No routers exist there.

Only PCs.

---

# Serial Interface

Now compare

```text
Serial0/0/0
```

Output

```text
Network Type

POINT-TO-POINT
```

This is different.

Why?

Because

Serial links connect

exactly

two routers.

Therefore

No DR

No BDR

No election.

Instead

```text
POINT-TO-POINT
```

---

Neighbor Count

```text
1
```

Exactly right.

One router.

Cafe Router.

---

Adjacent Neighbor Count

```text
1
```

Meaning

Neighbor relationship reached

```text
FULL
```

---

# show ip protocols

Output

```text
Routing Protocol

OSPF 1
```

Process

```text
1
```

Remember

Process ID

is

locally significant.

Router A

```text
OSPF 1
```

Router B

could run

```text
OSPF 10
```

Still works.

---

### Router ID

Initially

```text
172.16.0.1
```

Later

you changed

to

```text
1.1.1.1
```

Notice

Cisco immediately warned

```text
Reload

or

clear ip ospf process
```

Why?

Because

Router ID

is chosen

when OSPF starts.

Changing configuration

doesn't immediately restart

the protocol.

---

### Routing Networks

```text
172.16.0.1

0.0.0.0
```

This enables

only

that interface.

---

Second network

```text
10.0.16.0

0.0.1.255
```

Interesting.

Wildcard

```text
0.0.1.255
```

covers

```text
10.0.16.x

AND

10.0.17.x
```

So

one command

enabled

all four VLAN subinterfaces.

Very efficient.

---

# show ip ospf interface brief

Output

```text
Nbrs F/C

0/0
```

Many students ask

What is

```text
F/C
```

Meaning

```text
Fully Adjacent

/

Adjacent Count
```

On VLAN interfaces

```text
0/0
```

No routers.

---

Serial

shows

```text
0/0
```

in the brief output of Packet Tracer, but the detailed interface output confirms:

```text
Neighbor Count = 1
Adjacent Neighbor Count = 1
```

Packet Tracer doesn't always display the brief counters exactly like real IOS.

---

# show ip ospf neighbor

Output

```text
Neighbor

216.0.5.2

FULL

172.16.0.2
```

Perfect.

Everything healthy.

Meaning

```text
Hello

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

Completed successfully.

---

# Router ID Change

You entered

```cisco
router ospf 1

router-id 1.1.1.1
```

Cisco replied

```text
Reload

or

clear ip ospf process
```

Exactly correct.

Router ID

changes

only

after

restarting

OSPF.

---

# clear ip ospf process

You typed

```cisco
clear ip ospf process
```

Immediately

Neighbor dropped

```text
FULL

↓

DOWN
```

Makes sense.

You restarted

the protocol.

Everything begins again.

---

# Debug Output (The Best Part)

This is where you literally watch OSPF build the relationship.

---

## DR Election

```text
Elect DR

1.1.1.1
```

This happened

on

every VLAN interface.

Why?

Each VLAN

contains

only

one router.

Therefore

that router

wins

automatically.

---

## Build Router LSA

```text
Build router LSA

RID

1.1.1.1
```

The router creates

its

Type-1 LSA

describing

its interfaces

and links.

---

## Send DBD

```text
Send DBD
```

Database Description.

This is

the

summary

of

the LSDB.

---

## Receive DBD

```text
Rcv DBD
```

Neighbor replies

with

its

summary.

---

## Negotiation Done

```text
We are the SLAVE
```

One router becomes

Master.

One becomes

Slave.

This is

normal.

It is only used

to coordinate

database synchronization.

---

## Exchange

```text
State

EXCHANGE
```

Database summaries

are compared.

---

## Database Request

```text
Database request
```

Router says

"I need

some LSAs."

---

## LS Request

```text
LS REQ
```

This is

the

LSR packet.

Notice

Jeremy simplified this

on the board

as

LSA

↓

LSU

but internally

Cisco actually sends

```text
LSR

↓

LSU
```

---

## Synchronization

```text
Synchronized

state FULL
```

Databases

match.

SPF

can run.

Routes

installed.

OSPF

finished.

---

# Complete OSPF Adjacency Timeline (Matching Your Debug)

```text
OSPF Process Restart
        │
        ▼
DR Election (Ethernet VLANs)
        │
        ▼
Build Router LSA
        │
        ▼
Hello Exchange
        │
        ▼
Neighbor Discovered
        │
        ▼
ExStart
(Master/Slave Negotiation)
        │
        ▼
DBD Exchange
(Database Summaries)
        │
        ▼
LSR
(Request Missing LSAs)
        │
        ▼
LSU
(Send Requested LSAs)
        │
        ▼
LSAck
(Acknowledge Updates)
        │
        ▼
Loading
        │
        ▼
FULL
        │
        ▼
Run SPF
        │
        ▼
Install OSPF Routes
```

## What You Learned from This Lab

This lab exposed nearly every major aspect of OSPF operation:

* **`show ip route ospf`** confirmed that OSPF successfully installed learned routes.
* **`show ip ospf database`** showed the Link-State Database (LSDB), containing one Router LSA for each participating router.
* **`show ip ospf interface`** revealed interface-specific OSPF settings such as Area, Router ID, Network Type, Cost, DR status, timers, and neighbor counts.
* **`show ip protocols`** verified the OSPF process configuration, Router ID, and the network statements that enabled OSPF on your interfaces.
* Changing the **Router ID** required restarting the OSPF process because Router IDs are selected when the process starts.
* **`clear ip ospf process`** demonstrated that restarting OSPF tears down adjacencies and rebuilds them from scratch.
* The **debug output** let you observe the complete adjacency formation sequence: **DR election → Router LSA creation → DBD exchange → LSR → LSU → synchronization → Full adjacency**.

This is exactly the sequence you need to understand for CCNA troubleshooting: first verify neighbors, then understand where the adjacency process stops, and finally use the appropriate verification commands to identify the underlying cause.
