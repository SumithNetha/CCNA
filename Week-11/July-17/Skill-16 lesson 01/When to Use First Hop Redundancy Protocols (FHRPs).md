
# Skill 16 - Lesson 01

# When to Use First Hop Redundancy Protocols (FHRPs)

---

# Why This Lesson Matters

Up to now, we've learned how routers find alternate paths using routing protocols like:

* RIP
* EIGRP
* OSPF

These protocols provide **router redundancy**.

But there is another problem that routing protocols **cannot solve**.

> **How does a PC survive if its default gateway fails?**

That is exactly why **First Hop Redundancy Protocols (FHRPs)** were invented.

---

# The Hidden Problem

Imagine this network.

```text
                Internet
                    |
        -------------------------
        |                       |
     Router 1               Router 2
        |                       |
        +----------SW-----------+
                   |
      -----------------------------
      |            |             |
     PC1          PC2          Server
```

Both routers have Internet access.

Everything looks redundant.

Most people stop here and think:

> "Great. We have two routers."

Actually...

No.

---

# Why?

Every host only has **ONE default gateway**.

Example:

```text
PC1

IP Address      192.168.10.50
Subnet Mask     255.255.255.0
Default Gateway 192.168.10.1
```

That gateway points to:

```text
Router 1
```

If Router 1 dies...

```text
PC
 |
 |
X Router1 (Dead)

Router2 (Alive)
```

The PC does **NOT** automatically start using Router 2.

Instead it keeps trying:

```text
Send packet →

192.168.10.1

No reply.

Retry...

Retry...

Retry...
```

Eventually,

Everything outside the LAN becomes unreachable.

---

# Important Observation

The backup router is alive.

The Internet is alive.

The switch is alive.

The PC is alive.

Yet...

The user has no network connectivity.

---

## This is the key statement of the lesson

> **Router redundancy and Host Gateway Redundancy are NOT the same thing.**

Dynamic routing protects routers.

FHRPs protect hosts.

---

# Why Can't the PC Just Choose Router 2?

Because PCs are not routing devices.

Hosts don't run OSPF.

Hosts don't run EIGRP.

Hosts don't calculate alternate routes.

They simply follow this rule:

```text
Destination outside my subnet?

↓

Send everything to Default Gateway.
```

Only one gateway exists in the host configuration.

---

# What is the "First Hop"?

Imagine a packet leaving the PC.

```text
PC

↓

Default Gateway

↓

Other Routers

↓

Internet
```

The **first Layer 3 device** that receives the packet is called the:

> **First Hop**

Usually,

The first hop equals

> **Default Gateway**

Therefore,

**First Hop Redundancy Protocol**

means

> **Making the default gateway redundant.**

---

# What Problem Does FHRP Solve?

Instead of making a router the gateway...

FHRPs create a **Virtual Gateway**.

Instead of this:

```text
Gateway

192.168.10.1

↓

Router1
```

We get:

```text
Gateway

192.168.10.1

↓

Virtual Gateway

↓

Router1 (Active)

Router2 (Standby)
```

Now the host doesn't know which router is forwarding traffic.

It only knows:

```text
Gateway = 192.168.10.1
```

---

# Virtual IP Address

The gateway IP belongs to **neither router physically**.

Instead,

Both routers share it.

Example

```text
Router1

Physical IP

192.168.10.2
```

```text
Router2

Physical IP

192.168.10.3
```

Virtual Gateway

```text
192.168.10.1
```

Hosts configure:

```text
Default Gateway

192.168.10.1
```

Nobody configures:

```text
192.168.10.2
```

or

```text
192.168.10.3
```

The host only communicates with the **Virtual IP**.

---

# Active and Standby Routers

Initially:

```text
Virtual Gateway

192.168.10.1

↓

Router1
(Active)
```

Router2 waits.

```text
Standby
```

If Router1 fails...

```text
Virtual Gateway

192.168.10.1

↓

Router2
```

Notice what **didn't** change.

The gateway IP.

Still

```text
192.168.10.1
```

So every host continues working.

No DHCP renewal.

No manual changes.

No reconfiguration.

---

# The Three FHRP Protocols

---

## 1. HSRP

**Hot Standby Redundancy Protocol**

Cisco proprietary.

Works as:

```text
Active

↓

Standby
```

Only one router forwards traffic.

---

## 2. VRRP

**Virtual Router Redundancy Protocol**

Industry standard.

Vendor independent.

Very similar to HSRP.

Terminology:

```text
Master

↓

Backup
```

Instead of

```text
Active

↓

Standby
```

---

## 3. GLBP

**Gateway Load Balancing Protocol**

Cisco proprietary.

Unlike HSRP,

multiple routers can actively forward traffic while still providing redundancy.

Provides:

* Redundancy
* Load Balancing

instead of only failover.

---

# Quick Comparison

| Protocol | Vendor            | Traffic Forwarding      | Main Purpose                        |
| -------- | ----------------- | ----------------------- | ----------------------------------- |
| **HSRP** | Cisco Proprietary | One Active, One Standby | Gateway Redundancy                  |
| **VRRP** | Open Standard     | One Master, One Backup  | Gateway Redundancy                  |
| **GLBP** | Cisco Proprietary | Multiple Active Routers | Gateway Redundancy + Load Balancing |

---

# How Do Routers Detect Failure?

The routers continuously exchange:

> **Hello Messages**

Think of them like periodic health checks.

```text
Router1

Hello

↓

Router2
```

```text
Router2

Hello

↓

Router1
```

As long as Hellos arrive:

Everything is healthy.

If the standby router stops receiving Hellos:

```text
No Hello

↓

Active Router Failed
```

It immediately becomes the active gateway.

---

# The Hidden Problem — ARP

Suppose the PC already learned:

```text
192.168.10.1

↓

AA-AA-AA-AA-AA-AA
```

where `AA-AA-AA-AA-AA-AA` is Router1's **physical MAC address**.

If Router1 dies...

The PC still tries sending frames to that MAC until its ARP cache expires.

That delay would interrupt communication.

---

# The Second Part of the Solution: Virtual MAC Address

FHRPs solve this by creating not only a **Virtual IP**, but also a **Virtual MAC Address**.

Instead of Router1 advertising its own hardware MAC, the active router answers ARP requests with the **shared Virtual MAC**.

```text
Virtual IP

192.168.10.1

↓

Virtual MAC

0000.0C07.AC01
```

*(The exact virtual MAC format depends on the specific FHRP.)*

When Router2 takes over, it also uses **the same Virtual MAC**.

```text
PC ARP Cache

192.168.10.1

↓

Virtual MAC
```

Nothing changes from the PC's perspective.

Traffic continues flowing without waiting for the ARP cache to expire.

---

# Why Virtual IP + Virtual MAC Are Both Required

| Without Virtual IP                                      | Without Virtual MAC                                                                     |
| ------------------------------------------------------- | --------------------------------------------------------------------------------------- |
| Hosts would need a new default gateway after a failure. | Hosts would keep sending frames to the failed router's MAC until the ARP cache expired. |

Both identities must remain constant for seamless failover.

---

# Real-World Example (NetworkChuck Coffee)

Consider a coffee shop network during business hours:

* POS systems process payments.
* Inventory systems sync with headquarters.
* Security cameras upload footage.
* Guests use Wi-Fi.

If the primary router fails and every device points to that router's physical IP, the entire shop loses access to remote resources—even though a second router is available.

With an FHRP:

* The default gateway stays the same.
* The virtual MAC stays the same.
* The standby router immediately takes over.
* Users continue working with little or no noticeable interruption.

---

# Key Takeaways

* Dynamic routing protocols provide **router-to-router redundancy**, not **host gateway redundancy**.
* Hosts can use only **one default gateway**.
* FHRPs make the **first hop (default gateway)** resilient.
* FHRPs use a **Virtual IP Address** as the default gateway.
* FHRPs also use a **Virtual MAC Address** to avoid ARP-related outages.
* Routers detect failures by exchanging **Hello messages**.
* The three major FHRPs are:

  * **HSRP** – Cisco proprietary, Active/Standby
  * **VRRP** – Open standard, Master/Backup
  * **GLBP** – Cisco proprietary, Redundancy with Load Balancing

---

## Interview Questions

1. Why can't dynamic routing protocols alone provide gateway redundancy for end hosts?
2. What is the "first hop" in an IP network?
3. Why is a Virtual IP Address required in FHRPs?
4. Why is a Virtual MAC Address required in addition to the Virtual IP?
5. What happens if only the Virtual IP exists but the MAC address changes during failover?
6. How do FHRP routers detect that the active router has failed?
7. Compare HSRP, VRRP, and GLBP.
8. Which FHRP supports load balancing in addition to redundancy?
9. Why doesn't a host automatically start using another router when its default gateway fails?
10. In a production network, why is default gateway redundancy just as important as router redundancy?
