# Skill 20 — Why Is This Important?

## Layer 2 Security

This lesson introduces **Skill 20**, which follows the security fundamentals and Cisco AAA section. Your study plan places this lesson at the start of Skill 20, followed by **Port Security, DHCP Snooping, and Dynamic ARP Inspection (DAI)**. 

> **Core idea:** Cybersecurity is not a separate responsibility from networking. A network engineer is responsible for building basic security protections directly into the network infrastructure.

---

# 1. Why Does a Network Engineer Need Security?

A common misconception is:

```text
Networking Team       → Networking
Security Team         → Security
```

In reality:

```text
Networking + Security
        ↓
Secure Network Infrastructure
```

Network devices are where systems:

* Connect
* Communicate
* Trust each other
* Exchange network information

Therefore, weaknesses in the network itself can become security problems.

### Example

If anyone can plug a device into an unused switch port:

```text
Unknown Device
      │
      ▼
Switch Port
      │
      ▼
Network
```

That device may potentially attempt to:

* Impersonate network services
* Disrupt connectivity
* Manipulate local network traffic
* Intercept traffic

So **network security starts at the infrastructure level**, not only at firewalls or dedicated security appliances.

---

# 2. Cybersecurity Is Part of Good Networking

The lesson makes an important distinction:

You don't need to become a full-time cybersecurity specialist to care about security.

Whether you're a:

* Network engineer
* Systems administrator
* Infrastructure engineer
* IT administrator

security is still part of your responsibility.

Think:

```text
Good Network Design
        +
Security Controls
        ↓
Reliable + Secure Infrastructure
```

---

# 3. Security Is Not Only About Hackers

Security problems don't necessarily require a sophisticated attacker.

A problem could come from:

```text
Malicious device
      OR
Misconfigured device
      OR
Unauthorized device
```

All three can cause network problems.

For example, a device connected to an access switch could accidentally or intentionally behave like a DHCP server.

That can disrupt users even if there is no sophisticated attack.

Therefore:

> **Security controls also protect against configuration mistakes and unexpected devices, not only deliberate attacks.**

---

# 4. The Three Layer 2 Security Technologies

The lesson introduces three technologies that it considers important baseline protections at the access layer:

```text
┌──────────────────────────────┐
│       Layer 2 Security       │
├──────────────────────────────┤
│ Port Security                │
│ DHCP Snooping                │
│ Dynamic ARP Inspection (DAI) │
└──────────────────────────────┘
```

Each protects against a different type of problem.

| Technology        | Primary purpose                                  |
| ----------------- | ------------------------------------------------ |
| **Port Security** | Controls which devices can use a switch port     |
| **DHCP Snooping** | Helps prevent rogue DHCP servers                 |
| **DAI**           | Inspects ARP and helps prevent ARP-based attacks |

These three will be studied individually in the following lessons.

---

# 5. Port Security

## What problem does it solve?

**Port Security controls which devices are allowed to connect through a switch port.**

A switch normally learns MAC addresses dynamically.

For example:

```text
PC
MAC = AAAA.BBBB.CCCC
       │
       ▼
Switch Fa0/1
       │
       ▼
CAM Table
```

Port Security allows you to place restrictions on which MAC addresses can use that port.

---

## Example Scenario

Imagine an employee workstation connected to:

```text
Switch
  |
Fa0/1
  |
Employee PC
```

You can configure the port so that only an expected device/MAC address is permitted.

If an unexpected device is connected:

```text
Unknown Device
      │
      ▼
Fa0/1
      │
      ▼
Port Security
      │
      ▼
Violation
```

The switch can enforce the configured violation behavior.

### Main concept

> **Port Security = Control the devices allowed on a switch port.**

---

# 6. Why Port Security Matters

Consider an unused network jack in a coffee shop:

```text
             Switch
               │
        ┌──────┼──────┐
        │      │      │
       POS    PC    Unused
                     Port
                       │
                       ▼
                Unknown Device
```

Without appropriate protection, someone could potentially connect an unauthorized device.

Port Security provides a mechanism to restrict the devices permitted on the access port.

---

# 7. DHCP Snooping

To understand DHCP Snooping, first remember what DHCP does.

**DHCP automatically provides network configuration information to clients.**

For example:

```text
Client
  │
  │ DHCP
  ▼
DHCP Server
  │
  ├── IP address
  ├── Subnet mask
  ├── Default gateway
  └── Other configuration
```

---

# 8. The Rogue DHCP Problem

The problem occurs when an unauthorized device pretends to be a DHCP server.

Imagine:

```text
             Legitimate DHCP
                   │
                   ▼
                Switch
               /      \
              /        \
        Clients       Rogue Device
                         │
                         ▼
                    Fake DHCP
```

The rogue device can potentially provide incorrect network information.

For example, clients could receive an incorrect:

* IP configuration
* Default gateway
* DNS configuration

That can cause communication failures or facilitate attacks.

---

# 9. What DHCP Snooping Does

**DHCP Snooping allows the switch to distinguish between trusted and untrusted ports for DHCP traffic.**

Conceptually:

```text
Legitimate DHCP Server
        │
        ▼
 Trusted Switch Port
        │
        ▼
      Switch
        │
        ▼
      Clients
```

While:

```text
Rogue DHCP Server
        │
        ▼
 Untrusted Switch Port
        │
        X
        │
      Blocked
```

### Main concept

> **DHCP Snooping = Help prevent unauthorized DHCP servers from serving clients.**

---

# 10. Trusted vs Untrusted Ports

This is an important concept you'll use when configuring DHCP Snooping.

```text
DHCP Server
     │
     ▼
Trusted Port
     │
     ▼
  Switch
     │
     ▼
Untrusted Access Ports
     │
     ▼
  Clients
```

The switch treats DHCP messages differently depending on whether they arrive through a trusted or untrusted interface.

---

# 11. Dynamic ARP Inspection — DAI

Now we reach the third protection.

**DAI = Dynamic ARP Inspection**

First, understand ARP.

### ARP

ARP allows devices on a local IPv4 network to associate:

```text
IP address
     ↕
MAC address
```

For example:

```text
Who has 192.168.1.1?

192.168.1.1 → 00AA.11BB.22CC
```

The device can then send Ethernet frames to the appropriate MAC address.

---

# 12. The ARP Problem

ARP does not inherently provide strong authentication of ARP information.

An attacker on the local network can attempt to send misleading ARP information.

Conceptually:

```text
Victim
   │
   │ "Who is the gateway?"
   ▼
Attacker
   │
   │ Fake ARP information
   ▼
Victim
```

This can contribute to **ARP spoofing/poisoning** and potentially facilitate man-in-the-middle attacks.

---

# 13. What DAI Does

**Dynamic ARP Inspection examines ARP traffic and can block invalid or suspicious ARP messages.**

Conceptually:

```text
ARP Message
     │
     ▼
    DAI
     │
 ┌───┴────┐
 │        │
Valid   Invalid
 │        │
 ▼        X
Allow    Block
```

The lesson describes DAI as building on DHCP Snooping's protection.

### Main concept

> **DAI = Inspect ARP messages and block problematic ARP traffic.**

---

# 14. DHCP Snooping + DAI

These two technologies are particularly important together.

Think of the relationship:

```text
DHCP Snooping
      │
      ▼
Learns/helps establish legitimate
IP-to-MAC information
      │
      ▼
DAI
      │
      ▼
Uses that information when
validating ARP traffic
```

So don't study them as completely unrelated features.

They form part of a broader Layer 2 security strategy.

---

# 15. The Three Technologies Compared

| Feature           | Protects Against                            | Main Question                         |
| ----------------- | ------------------------------------------- | ------------------------------------- |
| **Port Security** | Unauthorized devices/MAC addresses on ports | "Who can connect here?"               |
| **DHCP Snooping** | Rogue DHCP behavior                         | "Who is allowed to provide DHCP?"     |
| **DAI**           | Invalid/forged ARP information              | "Is this ARP information legitimate?" |

### Easy memory

```text
PORT SECURITY
→ Device

DHCP SNOOPING
→ DHCP

DAI
→ ARP
```

---

# 16. Coffee Shop Attack Scenario

Put all three together.

Imagine someone connects an unauthorized device to a coffee shop switch.

```text
                 Switch
                   │
        ┌──────────┼──────────┐
        │          │          │
       POS       Staff      Attacker
                            Device
```

The attacker device attempts:

### Attack 1 — Unauthorized connection

```text
Attacker Device
      ↓
Switch Port
      ↓
Port Security
      ↓
Potential violation
```

### Attack 2 — Rogue DHCP

```text
Attacker Device
      ↓
Fake DHCP Server
      ↓
DHCP Snooping
      ↓
Blocked/restricted
```

### Attack 3 — ARP manipulation

```text
Attacker Device
      ↓
Fake ARP information
      ↓
DAI
      ↓
Invalid ARP blocked
```

The important idea is that **different controls address different attack surfaces**.

---

# 17. Defense in Depth

These technologies demonstrate **defense in depth**.

Don't depend on one protection.

Instead:

```text
                Network
                   │
          ┌────────┴────────┐
          ▼                 ▼
     Port Security      DHCP Snooping
                              │
                              ▼
                            DAI
```

Each control addresses a different Layer 2 threat.

If one control doesn't address a particular attack, another control may.

---

# 18. Baseline Hardening

The lesson strongly presents these technologies as **baseline hardening controls**, rather than something you wait to deploy after an attack.

The philosophy is:

```text
Bad approach:
Incident
   ↓
Security configuration

Better approach:
Design
   ↓
Hardening
   ↓
Deployment
   ↓
Normal operation
```

Security should be built into the network **before** something goes wrong.

---

# 19. Security and Availability

Security isn't only about confidentiality.

Poor Layer 2 security can also affect **network availability**.

For example:

```text
Rogue DHCP
    ↓
Wrong gateway
    ↓
Clients lose connectivity
    ↓
Business disruption
```

At a coffee shop this could mean:

* POS problems
* Staff connectivity problems
* Customer connectivity problems
* Service interruptions
* Lost revenue

Therefore:

> **Network security also contributes to network reliability and business continuity.**

---

# 20. How This Fits the Castle Rysen RFP

The Castle Rysen project explicitly requires Layer 2 security measures including:

* Port Security
* DHCP Snooping
* Dynamic ARP Inspection



This lesson therefore introduces a security requirement that you'll actually implement later in the Castle Rysen environment.

The progression is:

```text
Security concepts
       ↓
Threat understanding
       ↓
AAA
       ↓
Layer 2 security
       ↓
Port Security
       ↓
DHCP Snooping
       ↓
DAI
       ↓
Castle Rysen security deployment
```

---

# 21. What You Will Learn Next

According to your study plan, Skill 20 progresses as follows: 

### Lesson 01

**Configuring Switch Port Security**

→ Then its lab.

### Lesson 02

**Configuring DHCP Snooping**

→ Then its lab.

### Lesson 03

**Configuring DAI**

→ Then its lab.

### Lesson 04

**Castle Rysen Layer 2 Security Deployment**

→ Putting the individual protections together in the project environment.

---

# 22. Important CCNA Mental Model

You should now be able to map:

```text
Threat
  │
  ▼
Layer 2 Attack
  │
  ├───────────────┬────────────────┐
  ▼               ▼                ▼
Unauthorized    Rogue DHCP      ARP Spoofing
Device
  │               │                │
  ▼               ▼                ▼
Port Security  DHCP Snooping       DAI
```

That's much more useful than simply memorizing three feature names.

---

# 23. Quick Revision Table

| Technology        | Layer   | Main Function                                             | Think                                 |
| ----------------- | ------- | --------------------------------------------------------- | ------------------------------------- |
| **Port Security** | Layer 2 | Restricts devices/MAC addresses on switch ports           | **Who can connect?**                  |
| **DHCP Snooping** | Layer 2 | Controls DHCP behavior using trusted/untrusted interfaces | **Who can provide DHCP?**             |
| **DAI**           | Layer 2 | Validates ARP traffic                                     | **Can I trust this ARP information?** |

---

# 🧠 Final Takeaway

The lesson's message is bigger than simply learning three Cisco features:

> **Security is part of network engineering.**

When you deploy an access-layer switch, you shouldn't only ask:

```text
"Will devices communicate?"
```

You should also ask:

```text
"Who is allowed to connect?"
"Who is allowed to provide network information?"
"Can someone manipulate ARP?"
"Can one compromised device affect everyone else?"
```

And that's exactly where **Port Security → DHCP Snooping → DAI** come in.

**Next lesson: Port Security — now we move from "why" to the actual configuration.**
