# Week 14 — August 5, 2026

# Skill 20 — Castle Rysen Layer 2 Security Deployment

> **Core idea:** This lesson is not really about memorizing Layer 2 security commands. It is about converting a vague requirement such as *“secure the network”* into a **repeatable, documented Layer 2 security policy**.

Castle Rysen's RFP explicitly requires **Port Security, DHCP Snooping, and Dynamic ARP Inspection (DAI)** as core Layer 2 security controls. 

---

## 1. Layer 2 Security Policy

In a production network, security features should not be enabled randomly.

A proper deployment should answer:

* Which ports connect to end devices?
* Which ports connect to infrastructure?
* Which ports are trusted?
* Which ports are untrusted?
* How many MAC addresses should each access port allow?
* What happens when a violation occurs?
* Which VLANs require DHCP Snooping?
* Which interfaces should DHCP Snooping trust?
* Which interfaces should DAI trust?
* What rate limits should be applied?

### Why write the policy first?

A written policy provides:

* **Consistency** — every cafe can use the same standard.
* **Repeatability** — another administrator can deploy it.
* **Scalability** — configurations can be standardized across locations.
* **Troubleshooting** — expected behavior is documented.
* **Accountability** — there is a record of why security controls were configured.

The Castle Rysen RFP specifically calls for hardware and configuration standards that support turnkey rollouts with minimal error. 

---

# 2. Port Security

## What is Port Security?

**Port Security** restricts which MAC addresses are allowed to use a switchport.

Conceptually:

```text
                 Switch
                   |
             Fa0/10
                   |
              Employee PC
             MAC AAAAA
```

The policy can effectively say:

> Fa0/10 is allowed to communicate with MAC AAAAA.

If another unauthorized MAC address appears:

```text
Attacker
MAC BBBBB
   |
   X
Fa0/10
```

the switch can treat it as a security violation.

---

## 3. Where Should Port Security Be Used?

Port Security makes sense primarily on **end-device/access ports**.

Examples:

```text
PC ───── Access Port
Printer ───── Access Port
IP Phone ───── Access Port
Camera ───── Access Port
```

It generally should **not** simply be applied everywhere.

### Avoid blindly applying it to:

* Switch uplinks
* Router connections
* Infrastructure links
* Ports connected to devices that legitimately represent multiple MAC addresses
* Access-point connections where multiple wireless clients may appear behind the port

Why?

Because these connections can legitimately carry traffic from **multiple MAC addresses**.

---

# 4. Maximum MAC Address

For a simple dedicated end-device port, Castle Rysen's policy uses:

```text
Maximum MAC addresses = 1
```

Conceptually:

```text
Fa0/10
   |
   └── PC
       MAC A
```

Only one MAC is expected.

If a second unexpected MAC appears:

```text
Fa0/10
   |
   ├── PC       MAC A
   └── Attacker MAC B
```

→ **Port Security violation**

---

# 5. Violation Mode — Shutdown

The lesson uses the default **shutdown** behavior.

The security philosophy is:

> If something violates the policy, disable the port and investigate.

This is intentionally strict.

Instead of allowing suspicious traffic to continue:

```text
Unexpected MAC
      ↓
Violation
      ↓
Port shutdown
      ↓
Administrator investigates
```

This is useful for fixed administrative devices where an unexpected device should be treated as suspicious.

---

# 6. Sticky MAC Addresses

## What is Sticky MAC?

With **sticky MAC**, the switch dynamically learns a MAC address and associates it with the port as a secure MAC address.

Conceptually:

```text
PC connects
     ↓
Switch learns MAC A
     ↓
MAC A becomes secure/sticky
     ↓
Future MAC B → violation
```

This is convenient because the administrator doesn't necessarily need to manually type every MAC address.

---

## Sticky MAC and Patron Devices

This is where **policy matters**.

### Administrative device

Example:

```text
Admin PC
   ↓
Fa0/10
   ↓
Switch
```

The device is expected to remain there.

Sticky MAC makes sense.

### Patron device

A cafe patron network is different:

```text
Customer 1
Customer 2
Customer 3
Customer 4
...
```

Devices constantly come and go.

Therefore, blindly applying a sticky-MAC strategy to patron access ports can conflict with the intended operational model.

### Key lesson

> **Security controls must match the behavior of the endpoint.**

Don't implement a security feature simply because it is considered a "best practice."

---

# 7. DHCP Snooping

## What Problem Does It Solve?

DHCP Snooping protects against **rogue DHCP servers** and helps build a trusted DHCP binding database.

Normally, DHCP clients expect DHCP server responses.

An attacker could introduce:

```text
              Network
             /       \
       Legit DHCP   Rogue DHCP
        Server       Server
                       |
                  Bad gateway
                  Bad DNS
                  Bad IP
```

The rogue server could give clients incorrect network information.

DHCP Snooping allows the switch to distinguish between:

* **Trusted ports**
* **Untrusted ports**

---

# 8. Trusted vs Untrusted DHCP Ports

This distinction is critical.

### Trusted port

A trusted interface is allowed to receive legitimate DHCP server-side messages.

Typical examples:

```text
Router
   |
Trusted port
   |
Switch
```

or:

```text
DHCP Server
     |
Trusted uplink
     |
Switch
```

### Untrusted port

Client-facing interfaces are normally untrusted:

```text
PC
 |
Untrusted access port
 |
Switch
```

An untrusted client should not be able to behave like a DHCP server.

---

# 9. Castle Rysen DHCP Snooping Policy

The lesson's policy:

1. Enable DHCP Snooping on the cafe VLANs.
2. Trust the router/uplink interfaces.
3. Keep client-facing interfaces untrusted.
4. Rate-limit DHCP traffic on untrusted ports.

Conceptually:

```text
                    Router/DHCP
                         |
                    TRUSTED PORT
                         |
                    +----+----+
                    | Switch  |
                    +----+----+
                         |
              -----------------------
              |          |          |
           UNTRUSTED   UNTRUSTED  UNTRUSTED
              |          |          |
             PC       Printer     Camera
```

---

# 10. Why DHCP Rate Limiting?

DHCP Snooping also helps defend against **DHCP starvation**.

### DHCP starvation

An attacker generates many DHCP requests, potentially consuming the available DHCP address pool.

Conceptually:

```text
Attacker
   |
   | DHCP requests
   | DHCP requests
   | DHCP requests
   | DHCP requests
   ↓
Switch
   ↓
DHCP Server
   ↓
Address pool exhausted
```

Legitimate clients may then be unable to obtain an address.

Rate limiting on untrusted ports restricts how much DHCP traffic a client-facing port can generate.

---

# 11. Port Security + DHCP Snooping

These controls can complement each other.

### Port Security

Controls:

> **Who can use this physical port?**

### DHCP Snooping

Controls:

> **Who is allowed to send DHCP server-style responses?**

Together:

```text
              Layer 2 Security
                     |
          +----------+----------+
          |                     |
   Port Security          DHCP Snooping
          |                     |
   MAC restrictions       DHCP protection
          |                     |
          +----------+----------+
                     |
              Stronger defense
```

Port Security can also make DHCP starvation more difficult by limiting the number of MAC addresses that can appear on a port, although it is **not a complete defense against DHCP starvation**.

---

# 12. Dynamic ARP Inspection — DAI

## What is DAI?

**Dynamic ARP Inspection (DAI)** protects against malicious or invalid ARP messages.

The primary attack being addressed is:

### ARP spoofing

ARP normally allows hosts to associate:

```text
IP address ↔ MAC address
```

An attacker can lie about this association.

For example:

```text
Legitimate PC
IP = 192.168.10.20
MAC = AAAA

Attacker
MAC = BBBB
```

The attacker could attempt to convince other devices:

```text
192.168.10.20 → BBBB
```

instead of:

```text
192.168.10.20 → AAAA
```

This can facilitate **man-in-the-middle attacks**.

---

# 13. How DAI Defends Against ARP Spoofing

DAI examines ARP messages arriving on untrusted interfaces.

It can compare the information against trusted bindings.

The important relationship is:

```text
DHCP Snooping
      ↓
Binding Database
      ↓
IP ↔ MAC ↔ VLAN ↔ Interface
      ↓
DAI
      ↓
Validate ARP
```

If an ARP message doesn't match the expected information:

```text
ARP packet
    ↓
DAI inspection
    ↓
Does it match trusted information?
       /       \
     YES        NO
      ↓          ↓
    Allow      Drop
```

---

# 14. Why DHCP Snooping Comes Before DAI

This is one of the most important concepts from the lesson.

DAI depends on information generated by DHCP Snooping.

### DHCP Snooping

Builds the binding information.

### DAI

Uses that information to validate ARP.

So:

```text
              DHCP
               ↓
        DHCP Snooping
               ↓
       Binding Database
               ↓
              DAI
               ↓
        ARP validation
```

This is why these features should be viewed as a **security system**, rather than three unrelated commands.

---

# 15. Trusted Interfaces with DAI

Infrastructure interfaces are typically trusted according to the network design.

Client-facing interfaces remain untrusted.

Example:

```text
                Router
                  |
               TRUSTED
                  |
              +-------+
              |Switch |
              +-------+
              /   |   \
             /    |    \
        UNTRUSTED UNTRUSTED UNTRUSTED
           PC      PC      Camera
```

DAI can then inspect ARP traffic arriving from untrusted endpoints.

---

# 16. ARP Rate Limiting

DAI can also use **ARP inspection rate limits**.

The objective is to prevent excessive ARP traffic from an untrusted port from overwhelming the network or becoming part of an attack.

Again, this is a policy decision:

> The rate should be appropriate for the expected behavior of the endpoint.

Don't blindly copy a value without understanding the traffic profile.

---

# 17. The Three-Layer Defense

This is the most important takeaway from today's lesson.

| Feature           | Primary protection         | Main question                         |
| ----------------- | -------------------------- | ------------------------------------- |
| **Port Security** | Unauthorized MAC addresses | **Who can use this port?**            |
| **DHCP Snooping** | Rogue DHCP / DHCP abuse    | **Who can provide DHCP information?** |
| **DAI**           | ARP spoofing               | **Is this ARP claim legitimate?**     |

Together:

```text
             END DEVICE
                 |
                 ↓
         +---------------+
         | Port Security |
         +---------------+
                 |
                 ↓
        Who is allowed here?
                 |
                 ↓
         +---------------+
         | DHCP Snooping  |
         +---------------+
                 |
                 ↓
        Who can provide DHCP?
                 |
                 ↓
        Binding Database
                 |
                 ↓
         +---------------+
         |      DAI      |
         +---------------+
                 |
                 ↓
         Is ARP legitimate?
```

---

# 18. Castle Rysen Real-World Policy

The RFP requires the Castle Rysen network to implement these Layer 2 security controls. It also requires segmentation and restricted administrative access, particularly in the Fallout Shelter. 

A practical policy therefore looks like:

```text
                     CASTLE RYSEN
                          |
                    Layer 2 Security
                          |
        +-----------------+-----------------+
        |                 |                 |
        ↓                 ↓                 ↓
 Port Security      DHCP Snooping          DAI
        |                 |                 |
 End-device MACs     DHCP trust         ARP validation
        |                 |                 |
        +-----------------+-----------------+
                          |
                    VLAN Segmentation
                          |
        +-----------------+-----------------+
        |                 |                 |
      Admin             Internal          Guest
```

And the RFP's security requirements go beyond Layer 2 controls: at district shops, patron devices must be separated from administrative devices, while Fallout Shelters require separate management, internal, surveillance, and guest segments. 

---

# 19. Production Deployment Mindset

One of the strongest points in this lesson is **don't configure blindly on production equipment**.

A safer workflow is:

```text
1. Map interfaces
       ↓
2. Identify endpoint types
       ↓
3. Identify trusted interfaces
       ↓
4. Identify untrusted interfaces
       ↓
5. Define security policy
       ↓
6. Build configuration
       ↓
7. Review configuration
       ↓
8. Schedule maintenance window
       ↓
9. Deploy
       ↓
10. Verify
       ↓
11. Document
```

This matters because a mistake with Layer 2 security can create an outage.

For example:

```text
Wrong DHCP trust
       ↓
DHCP responses blocked
       ↓
Clients can't obtain IP addresses
       ↓
NETWORK OUTAGE
```

Or:

```text
Incorrect DAI configuration
       ↓
Legitimate ARP blocked
       ↓
Connectivity problems
```

---

# 20. Verification Is Part of the Configuration

A configuration isn't finished simply because the commands were accepted.

The real workflow is:

```text
Configure
   ↓
Verify
   ↓
Test
   ↓
Observe behavior
   ↓
Correct if necessary
   ↓
Document
```

Useful verification should answer:

### Port Security

* Is port security enabled?
* What MAC addresses are secure?
* How many MAC addresses are allowed?
* Has a violation occurred?
* Is the port still operational?

### DHCP Snooping

* Is DHCP Snooping enabled?
* Which VLANs are protected?
* Which ports are trusted?
* Are DHCP bindings being learned?
* Are rate limits configured?

### DAI

* Is DAI enabled for the correct VLANs?
* Which interfaces are trusted?
* Are ARP packets being inspected?
* Are legitimate ARP messages passing?
* Are violations being detected?

---

# 21. Important Real-World Lesson

The biggest lesson isn't:

> `show command X`

or:

> `configure command Y`

It's this:

## **Security is a policy before it is a configuration.**

You first decide:

```text
What should be allowed?
        ↓
What should be prohibited?
        ↓
Where should it apply?
        ↓
What happens when policy is violated?
        ↓
How will I verify it?
```

Only then do you translate that policy into Cisco IOS configuration.

---

# 22. CCNA Mental Model

Remember the three technologies like this:

### 🔒 Port Security

**MAC-level control**

> "I don't expect this device on this port."

### 🛡️ DHCP Snooping

**DHCP-level control**

> "I don't trust this port to act like a DHCP server."

### 👁️ DAI

**ARP-level control**

> "I don't trust this device's ARP claim unless it can be validated."

And their relationship:

```text
        PORT SECURITY
             │
       MAC protection
             │
             ▼
       DHCP SNOOPING
             │
      Builds bindings
             │
             ▼
            DAI
             │
       ARP protection
             │
             ▼
     Layer 2 defense
```

## The one sentence to remember

> **Port Security controls who can connect, DHCP Snooping controls who can provide DHCP information, and DAI controls who can make trusted ARP claims.**

That is the core of **Castle Rysen Layer 2 Security Deployment**.
