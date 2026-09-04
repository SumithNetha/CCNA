# Week 14 — August 4

# Skill 20 — Layer 2 Security

## Lesson 03: Configuring Dynamic ARP Inspection (DAI)

> **Core idea:** Dynamic ARP Inspection (DAI) protects a Layer 2 network from **ARP spoofing/poisoning** by inspecting ARP messages and comparing their IP/MAC information against the **DHCP Snooping Binding Table**.

---

# 1. Why Do We Need DAI?

ARP is used on an IPv4 LAN to discover the MAC address associated with an IP address.

For example:

```text
Client
  |
  | ARP Request:
  | "Who has 192.168.10.1?"
  |
  +--------------------+
                       |
              Server / Gateway
              "192.168.10.1
               is at AA:AA:AA:AA:AA:AA"
```

The problem is that **ARP inherently trusts ARP replies**.

An attacker can exploit this by sending a forged ARP reply:

```text
Real Gateway:
192.168.10.1 → AA:AA:AA:AA:AA:AA

Attacker:
192.168.10.1 → BB:BB:BB:BB:BB:BB
```

The victim may accept:

```text
192.168.10.1 → BB:BB:BB:BB:BB:BB
```

Now traffic intended for the legitimate gateway/server can be redirected toward the attacker.

---

# 2. ARP Spoofing / ARP Poisoning

### Normal ARP

```text
Client
   |
   | "Who has Server IP?"
   |
   ▼
Server
   |
   | "I have it"
   ▼
Client
```

### ARP Spoofing

```text
Client
   |
   | "Who has Server IP?"
   |
   +----------------------+
                          |
                    Attacker
                    "That's me!"
                          |
                          ▼
                    Client's ARP
                       Cache
```

The attacker effectively **pretends to be another device**.

This can lead to a:

### Man-in-the-Middle (MITM) Attack

The attacker positions themselves between two communicating devices:

```text
Client  <──────>  Attacker  <──────>  Server
                    |
                    └─ Can potentially inspect/modify/forward traffic
```

The attacker may be able to:

* Intercept traffic
* Inspect traffic
* Modify traffic
* Forward traffic so communication appears normal

The important point is that **ARP itself does not authenticate the identity being advertised**.

---

# 3. What Does DAI Do?

**Dynamic ARP Inspection** provides a mechanism for determining whether ARP information is legitimate.

DAI inspects ARP traffic arriving on **untrusted interfaces**.

It then checks the claimed:

```text
IP Address
     +
MAC Address
```

against information already learned by the switch.

The key source of that information is the:

> **DHCP Snooping Binding Table**

---

# 4. DAI Depends on DHCP Snooping

This is one of the most important concepts from this lesson.

```text
DHCP Snooping
      │
      ▼
DHCP Snooping Binding Table
      │
      │ IP ↔ MAC mapping
      ▼
Dynamic ARP Inspection
      │
      ▼
Validate ARP messages
```

### DHCP Snooping

DHCP Snooping observes DHCP communication and records legitimate IP/MAC bindings.

For example:

```text
IP Address              MAC Address
------------------------------------------------
192.168.10.10            AA:AA:AA:AA:AA:AA
192.168.10.11            BB:BB:BB:BB:BB:BB
192.168.10.12            CC:CC:CC:CC:CC:CC
```

DAI can then use these mappings as its **source of truth**.

If a device claims:

```text
192.168.10.10 → DD:DD:DD:DD:DD:DD
```

but the binding table says:

```text
192.168.10.10 → AA:AA:AA:AA:AA:AA
```

the switch can recognize the inconsistency.

### Result:

```text
ARP claim doesn't match binding
              ↓
        ARP packet dropped
```

This is why the lesson emphasizes:

> **No DHCP Snooping, no DAI** — DAI depends on the DHCP Snooping binding table for its normal validation mechanism.

---

# 5. DAI Configuration Process

The lesson gives three fundamental configuration steps:

### Step 1 — Enable DHCP Snooping

DHCP Snooping must be operating so the switch can build its binding table.

```text
DHCP Snooping
      ↓
Binding Table
```

### Step 2 — Enable DAI for the VLANs

DAI is enabled for the VLANs that need ARP protection.

```text
VLAN 10 → DAI enabled
VLAN 20 → DAI enabled
VLAN 30 → DAI enabled
```

### Step 3 — Identify Trusted Interfaces

Interfaces are normally treated as **untrusted** for ARP inspection.

You selectively mark appropriate interfaces as trusted.

```text
                    Switch
                      |
          +-----------+-----------+
          |                       |
       Trusted                Untrusted
       Uplink                 User PC
          |                       |
       Switch                  Client
```

---

# 6. Trusted vs Untrusted Interfaces

This is a major DAI concept.

## Untrusted Interface

An untrusted interface is subject to ARP inspection.

Typical example:

```text
Switch
  |
  |
Access Port
  |
User PC
```

The switch examines ARP packets arriving from the user-facing port.

This is desirable because an ordinary user device should not be freely able to inject arbitrary ARP information.

---

## Trusted Interface

A trusted interface is exempted from the same ARP inspection behavior.

The lesson specifically identifies interfaces that may need to be trusted.

Examples:

* Switch uplinks
* Trunk links
* Routers
* Firewalls
* Some servers
* Wireless infrastructure
* Other infrastructure devices

The exact trust decision depends on the network design.

---

# 7. Why Uplinks Need Special Attention

Consider:

```text
Switch A
   |
   | Uplink
   |
Switch B
   |
Users
```

An uplink carries traffic originating from devices connected to another switch.

Therefore, ARP traffic arriving through the uplink may represent devices that aren't directly connected to that physical interface.

If DAI treats the uplink incorrectly as an ordinary untrusted access port, legitimate ARP traffic can be rejected.

This can cause connectivity problems.

### General rule from the lesson:

> Identify your uplinks and trunks before deploying DAI.

---

# 8. Static IP Addresses — Important Edge Case

This is one of the most important practical limitations discussed in the lesson.

DHCP Snooping learns bindings from **DHCP activity**.

Therefore, a device configured with a static IP may not appear in the DHCP Snooping Binding Table.

Example:

```text
Router

IP: 192.168.10.1
MAC: AA:AA:AA:AA:AA:AA

Configured manually
```

The switch may have no DHCP Snooping binding for:

```text
192.168.10.1 ↔ AA:AA:AA:AA:AA:AA
```

If that router sends an ARP reply through an interface being inspected by DAI, the switch may consider the traffic invalid.

### Potential result:

```text
Legitimate ARP
      ↓
No matching DHCP binding
      ↓
DAI rejects it
      ↓
Connectivity failure
```

Therefore:

> **Static-IP devices must be considered when designing DAI.**

---

# 9. Infrastructure Devices That Need Attention

Before enabling DAI, identify devices such as:

| Device              | Why it matters                                     |
| ------------------- | -------------------------------------------------- |
| Router              | Often uses static addressing                       |
| Firewall            | Often uses static addressing                       |
| Server              | May use static addressing                          |
| Switch              | Management interfaces may use static addressing    |
| Wireless controller | Often infrastructure/static addressing             |
| WAP                 | May use static addressing or special DHCP behavior |
| Switch uplinks      | Carry traffic for many downstream devices          |
| Trunks              | Carry multiple VLANs                               |

The lesson's key message is:

> **Security controls must be designed around the actual network, not blindly enabled everywhere.**

---

# 10. Wireless Access Points and DAI

A WAP creates another consideration.

Suppose:

```text
                    Switch
                      |
                      | One physical port
                      |
                     WAP
              /       |       \
           Client   Client   Client
```

One switch interface can represent many wireless clients.

Therefore, that interface can potentially carry a much larger amount of ARP traffic than a normal wired workstation port.

DAI applies an **ARP packet-per-second rate limit** to ARP packets on untrusted ports.

This provides another protection mechanism.

However:

```text
Normal PC
   ↓
Low ARP volume
   ↓
Default rate may be sufficient

High-density WAP
   ↓
Many clients
   ↓
Potentially much higher ARP volume
   ↓
Rate limit may need review
```

So DAI configuration must consider the **density and behavior of the connected device**.

---

# 11. ARP Rate Limiting

DAI doesn't only validate ARP identity.

It can also apply a rate limit to ARP packets arriving on untrusted interfaces.

Conceptually:

```text
Untrusted Port
      |
      | ARP packets
      ▼
+------------------+
| DAI Rate Limiter |
+------------------+
      |
      ▼
ARP inspection
```

This can help protect against excessive ARP traffic.

However, a rate that works for a normal workstation may not necessarily be appropriate for:

* High-density wireless
* Shared infrastructure links
* Special-purpose devices

Therefore:

> **Review ARP rate limits for unusual/high-density interfaces.**

---

# 12. DAI Validation

DAI can perform additional validation of ARP packets.

The lesson identifies three important fields:

### 1. Source MAC

Checks the source MAC information in the ARP packet.

### 2. Destination MAC

Checks the destination MAC information.

### 3. IP Address

Checks the IP address information contained in the ARP packet.

Conceptually:

```text
                ARP Packet
                    |
          +---------+---------+
          |         |         |
          ▼         ▼         ▼
      Source MAC  Dest MAC   IP
          |         |         |
          +---------+---------+
                    |
                    ▼
              Validation
```

The **source MAC + IP relationship** is particularly important for detecting classic spoofing.

---

# 13. Source MAC + IP Validation

Suppose DHCP Snooping learned:

```text
IP: 192.168.20.50
MAC: AA:AA:AA:AA:AA:AA
```

An attacker sends an ARP message claiming:

```text
IP: 192.168.20.50
MAC: BB:BB:BB:BB:BB:BB
```

DAI can identify:

```text
Expected:
192.168.20.50 → AA:AA:AA:AA:AA:AA

Received:
192.168.20.50 → BB:BB:BB:BB:BB:BB

                ↓

             Mismatch
                ↓
          ARP rejected
```

This is the fundamental DAI defense against ARP identity spoofing.

---

# 14. Destination MAC Validation

Destination MAC validation provides another layer of checking.

It becomes particularly interesting with:

### Gratuitous ARP

A **gratuitous ARP** is an unsolicited ARP message rather than a response to a normal ARP request.

These can be legitimate.

For example, **First Hop Redundancy Protocols (FHRPs)** can use gratuitous ARP for legitimate network operations.

But attackers can also use gratuitous ARP to manipulate ARP tables.

Therefore:

```text
Destination MAC validation
           ↓
     More inspection
           ↓
     More security
           +
     More edge cases
           ↓
Need to understand the network
```

This illustrates an important operational principle:

> **More security isn't automatically better if the security control isn't correctly designed for the environment.**

---

# 15. DAI and FHRP

This is especially relevant to your CCNA studies because you already covered **FHRP/HSRP**.

You previously learned that HSRP can use gratuitous ARP.

Therefore, when deploying DAI:

```text
HSRP
 ↓
Gratuitous ARP
 ↓
DAI
 ↓
Validation
```

You must understand that not every unusual ARP packet is malicious.

This is a good example of why Layer 2 security requires understanding **normal network behavior** before enforcing security policies.

---

# 16. DAI in Castle Rysen Coffee

The Castle Rysen RFP explicitly requires **Layer 2 security**, including:

* Port Security
* DHCP Snooping
* Dynamic ARP Inspection (DAI)



The RFP also requires the network to separate different types of traffic using VLANs and security boundaries.

For example, a District Shop separates:

```text
District Shop
│
├── Administrative devices
│   ├── Network equipment
│   ├── Cameras
│   └── Plex server
│
└── Patron devices
```

The Fallout Shelter has additional segmentation:

```text
Fallout Shelter
│
├── Management
├── Internal Communication
├── Video Surveillance
└── Guest
```



DAI becomes one component of the overall defense-in-depth architecture.

---

# 17. Layer 2 Security Stack

At this point, think about the three major Layer 2 protections you've been learning:

```text
                 Layer 2 Security
                       │
        +--------------+--------------+
        │              │              │
        ▼              ▼              ▼
   Port Security  DHCP Snooping      DAI
        │              │              │
        ▼              ▼              ▼
   MAC/device       DHCP binding   ARP validation
     control           table
                       │
                       └──────────────┐
                                      ▼
                                   DAI uses
                                   bindings
```

### Port Security

Controls which MAC addresses are allowed on a switch port.

### DHCP Snooping

Protects DHCP and builds legitimate IP-to-MAC bindings.

### DAI

Uses those bindings to validate ARP.

This creates a layered defense rather than relying on a single security mechanism.

---

# 18. The Complete Attack → Defense Flow

### Without DAI

```text
Attacker
   |
   | Fake ARP
   | "Gateway IP = My MAC"
   ▼
Switch
   |
   ▼
Victim
   |
   ▼
ARP cache poisoned
   |
   ▼
Traffic redirected
   |
   ▼
Potential MITM
```

### With DHCP Snooping + DAI

```text
DHCP
  │
  ▼
DHCP Snooping
  │
  ▼
Binding Table
  │
  │
  │       Attacker sends fake ARP
  │                    │
  │                    ▼
  └──────────────►     DAI
                       │
                       ▼
                 Compare IP/MAC
                       │
                 +-----+-----+
                 │           │
               Match      Mismatch
                 │           │
                 ▼           ▼
               Allow       Drop
```

---

# 19. Production Deployment Strategy

The lesson gives a very practical deployment approach.

### Before enabling DAI:

#### 1. Identify uplinks

```text
Switch ↔ Switch
```

#### 2. Identify trunks

```text
Switch ↔ Router
Switch ↔ Switch
Switch ↔ WAP/Controller infrastructure
```

#### 3. Identify static-IP devices

```text
Routers
Firewalls
Servers
Infrastructure
```

#### 4. Identify special edge cases

```text
WAPs
FHRPs
High-density links
Special applications
```

#### 5. Determine trusted interfaces

Mark appropriate interfaces as trusted.

#### 6. Enable DAI on required VLANs

Only then deploy inspection.

This prevents the classic problem:

```text
Enable security feature
       ↓
Legitimate traffic gets blocked
       ↓
Network outage
       ↓
"Why is the network broken?"
       ↓
"Oh... DAI."
```

---

# 20. Verification

After configuring DAI, don't simply assume it works.

The lesson recommends verifying:

### VLAN inspection status

Determine which VLANs have ARP inspection enabled.

```text
VLAN
 ↓
DAI enabled?
```

### Interface trust state

Determine which interfaces are:

```text
Trusted
   vs
Untrusted
```

### ARP rate limits

Review the configured/default ARP rate limits.

```text
Interface
   ↓
ARP packets/sec
   ↓
Within expected range?
```

Verification is critical because DAI can intentionally drop traffic.

---

# 21. Key Terms

| Term                    | Meaning                                                                                      |
| ----------------------- | -------------------------------------------------------------------------------------------- |
| **ARP**                 | Protocol used to discover the MAC address associated with an IPv4 address on a local network |
| **ARP Spoofing**        | Forging ARP information to impersonate another device                                        |
| **ARP Poisoning**       | Manipulating ARP caches with false IP/MAC mappings                                           |
| **MITM**                | Man-in-the-middle attack where an attacker inserts themselves into communication             |
| **DAI**                 | Dynamic ARP Inspection; validates ARP traffic                                                |
| **DHCP Snooping**       | Layer 2 security feature that observes DHCP traffic and builds bindings                      |
| **Binding Table**       | Table containing learned IP/MAC associations                                                 |
| **Trusted Interface**   | Interface excluded from normal DAI inspection behavior                                       |
| **Untrusted Interface** | Interface where ARP traffic is inspected                                                     |
| **Gratuitous ARP**      | Unsolicited ARP message that can be legitimate or malicious                                  |
| **ARP Rate Limit**      | Restriction on the rate of ARP packets accepted on an interface                              |

---

# 22. Exam-Level Mental Model

Memorize this chain:

```text
DHCP Snooping
      ↓
Learns legitimate
IP ↔ MAC mappings
      ↓
Binding Table
      ↓
DAI
      ↓
Inspects ARP
      ↓
Compare claimed IP/MAC
against known binding
      ↓
Valid → Allow
Invalid → Drop
```

And remember:

> **DHCP Snooping tells the switch who legitimately received an IP address. DAI uses that information to determine whether ARP claims are legitimate.**

---

# 23. Most Important Takeaways

### 1. ARP is inherently trusting

ARP does not inherently authenticate the sender of an ARP reply.

### 2. ARP spoofing enables MITM attacks

An attacker can claim to own another device's IP address.

### 3. DAI protects against ARP spoofing

DAI inspects ARP messages and validates their information.

### 4. DAI normally relies on DHCP Snooping

```text
DHCP Snooping → Binding Table → DAI
```

### 5. User-facing ports are normally untrusted

This allows DAI to inspect ARP traffic from endpoint devices.

### 6. Infrastructure links require careful planning

Pay particular attention to:

* Uplinks
* Trunks
* Routers
* Firewalls
* Servers
* WAPs
* Static-IP devices

### 7. Static IPs are an important edge case

A statically configured device may not have a DHCP Snooping binding.

### 8. DAI can validate multiple fields

The lesson identifies:

```text
Source MAC
Destination MAC
IP Address
```

### 9. Gratuitous ARP isn't automatically malicious

FHRPs can legitimately use gratuitous ARP.

### 10. Always verify after deployment

Check:

```text
DAI-enabled VLANs
Trusted interfaces
Untrusted interfaces
ARP rate limits
```

---

# 🔥 One-Line Revision

**DAI = DHCP Snooping's IP/MAC knowledge + ARP inspection → blocks forged ARP identities.**

And the bigger **Week 14 Layer 2 Security** picture is:

```text
Port Security
      +
DHCP Snooping
      +
Dynamic ARP Inspection
      ↓
Defense-in-Depth at Layer 2
```

The Castle Rysen project specifically calls for these Layer 2 security mechanisms as part of its network-security implementation. 
