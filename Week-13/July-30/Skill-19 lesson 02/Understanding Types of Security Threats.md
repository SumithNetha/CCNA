# Week 13 — July 30

## Skill 19 — Understanding Types of Security Threats

> **Git-ready CCNA Study Notes**
> Source: NetworkChuck Academy — Skill 19 Lesson 02. 

---

# 1. Security Threats: The Big Picture

A **security threat** is something that can potentially cause harm to a system, network, device, application, or data.

The important approach is to recognize **patterns of attacker behavior**, rather than trying to memorize hundreds of individual attack names.

The major threat categories covered in this lesson are:

1. **Denial of Service (DoS)**
2. **Spoofing**
3. **Man-in-the-Middle (MITM)**
4. **Reflection and Amplification**
5. **Distributed Denial of Service (DDoS)**
6. **Reconnaissance**
7. **Malware**

A single real-world attack can combine multiple categories.

For example:

```text
Spoofing
   ↓
Fake DHCP server
   ↓
Victim receives malicious network configuration
   ↓
Traffic redirected through attacker
   ↓
Man-in-the-Middle
```

---

# 2. Denial of Service — DoS

## Definition

A **Denial of Service (DoS)** attack attempts to make a service, device, or network **unavailable to legitimate users**.

The attacker's primary objective is generally:

> **Consume or exhaust the target's resources so legitimate users cannot access the service.**

DoS does not necessarily involve stealing information.

### Common resources that can be exhausted

* CPU
* Memory
* Connection tables
* Bandwidth
* Buffers
* Processing capacity
* Application resources

---

## TCP SYN Flood

A classic DoS attack is a **TCP SYN flood**.

Normally, TCP establishes a connection using the three-way handshake:

```text
Client                  Server
  |                       |
  |-------- SYN --------->|
  |<------ SYN-ACK -------|
  |-------- ACK --------->|
  |                       |
      Connection established
```

With a SYN flood:

```text
Attacker                 Server
   |                       |
   |-------- SYN --------->|
   |<------ SYN-ACK -------|
   |                       |
   |-------- SYN --------->|
   |<------ SYN-ACK -------|
   |                       |
   |-------- SYN --------->|
   |<------ SYN-ACK -------|
   |                       |
        No final ACK
```

The attacker repeatedly starts connections but does not complete them.

The server maintains many **half-open connections**, consuming resources.

Eventually:

```text
Fake requests
      ↓
Connection resources consumed
      ↓
Server overwhelmed
      ↓
Legitimate users cannot connect
```

### Key idea

**DoS = make the service unavailable.**

---

# 3. Spoofing

## Definition

**Spoofing** occurs when an attacker **pretends to be something they are not**.

The attacker falsifies an identity or address to deceive another device or system.

Examples include:

* MAC address spoofing
* IP address spoofing
* DHCP-related spoofing
* Identity spoofing

---

## MAC Address Spoofing

A device can change or manipulate the MAC address it presents to the network.

An attacker may also attempt to appear as multiple different devices.

Example:

```text
Real device
MAC: AA:AA:AA:AA:AA:01

Attacker pretends to be:
AA:AA:AA:AA:AA:01
AA:AA:AA:AA:AA:02
AA:AA:AA:AA:AA:03
...
```

This can create network confusion or contribute to resource exhaustion.

---

# 4. Spoofing Can Cause DoS

Threat categories are **not isolated**.

One attack technique can enable another attack.

For example:

```text
MAC spoofing
     ↓
Attacker creates many fake identities
     ↓
DHCP server receives many requests
     ↓
DHCP address pool becomes exhausted
     ↓
Legitimate clients cannot obtain IP addresses
     ↓
DoS
```

### Important CCNA connection

This concept leads directly into **DHCP Snooping**, which is covered later in your CCNA course.

---

# 5. Rogue DHCP Server

A **rogue DHCP server** is an unauthorized DHCP server operating on the network.

Normally:

```text
Client
  ↓
Legitimate DHCP Server
  ↓
Correct IP configuration
```

With a rogue DHCP server:

```text
Client
  ↓
Rogue DHCP Server
  ↓
Malicious configuration
```

The attacker can potentially provide victims with:

* Malicious default gateway
* Malicious DNS server
* Incorrect IP configuration

This can redirect traffic through attacker-controlled infrastructure.

---

# 6. Man-in-the-Middle — MITM

## Definition

A **Man-in-the-Middle (MITM)** attack occurs when an attacker positions themselves between two communicating systems.

Instead of:

```text
Client ─────────────── Server
```

the communication becomes:

```text
Client ───── Attacker ───── Server
```

The attacker may be able to:

* Observe traffic
* Capture information
* Modify traffic
* Forward traffic
* Redirect communication

The dangerous characteristic is that communication can continue normally.

### Why MITM is dangerous

A DoS is often obvious:

```text
Service → DOWN
```

A MITM can be much harder to detect:

```text
Service → Appears to work normally
              ↓
        Attacker observes
        or modifies traffic
```

---

# 7. Rogue DHCP → MITM Example

Consider the Castle Rysen Coffee network.

A legitimate DHCP server provides:

```text
IP Address
Subnet Mask
Default Gateway
DNS Server
```

An attacker introduces a rogue DHCP server.

The victim receives:

```text
IP Address       → legitimate-looking
Subnet Mask      → legitimate-looking
Default Gateway  → attacker
DNS Server       → attacker
```

The victim believes everything is working.

Traffic can now potentially travel through attacker-controlled infrastructure.

```text
Victim
   ↓
Attacker
   ↓
Internet / Server
```

This is an example of how **spoofing can facilitate a MITM attack**.

---

# 8. Reflection and Amplification

These attacks increase the impact of malicious traffic.

## Reflection

The attacker causes other systems to send traffic toward the victim.

Conceptually:

```text
Attacker
   ↓
Third-party systems
   ↓
Victim
```

This can also make it harder to identify the original attacker.

---

## Amplification

An attacker sends a relatively small amount of traffic/request and causes a much larger amount of traffic to reach the victim.

Conceptually:

```text
Small attacker request
        ↓
Reflection service
        ↓
Large response
        ↓
Victim
```

### Key distinction

**Reflection** → traffic is bounced through other systems.

**Amplification** → the resulting traffic is significantly larger than the attacker's original traffic.

---

# 9. Distributed Denial of Service — DDoS

## Definition

A **Distributed Denial of Service (DDoS)** attack is essentially a DoS attack launched from **many systems**.

Instead of:

```text
Attacker
   ↓
Target
```

you have:

```text
Compromised Device ─┐
Compromised Device ─┤
Compromised Device ─┤
Compromised Device ─┤
                    ↓
                  Target
```

The attacking systems could include:

* Compromised computers
* IoT devices
* Cameras
* Other infected systems

### DoS vs DDoS

| DoS                                  | DDoS                                   |
| ------------------------------------ | -------------------------------------- |
| Typically one source                 | Multiple distributed sources           |
| Easier to identify source            | More difficult to trace                |
| Limited attacking resources          | Potentially massive combined resources |
| Attempts to exhaust target resources | Same objective                         |

### Remember

> **DoS = one attacker/source conceptually**
> **DDoS = distributed sources attacking together**

---

# 10. Reconnaissance

## Definition

**Reconnaissance** is the process of **gathering information about a target before launching or developing an attack**.

Think of it as:

> **Learning about the target before deciding how to attack it.**

Reconnaissance can be passive or active.

---

## Examples

### WHOIS

Attackers can investigate publicly available domain-registration information.

They may learn information about:

* Domains
* Organizations
* Registration information
* Infrastructure

---

## Port Scanning

A **port scan** checks which TCP/UDP ports are accessible on a device.

For example:

```text
Target
 ├── TCP 22   → Open
 ├── TCP 80   → Open
 ├── TCP 443  → Open
 ├── TCP 23   → Closed
 └── TCP 3389 → Open
```

An attacker can use this information to build a profile of the target.

### Why open ports matter

An open port isn't automatically a vulnerability.

However:

```text
Open Port
   ↓
Running Service
   ↓
Potential Attack Surface
```

Therefore, unnecessary services should not be exposed.

---

# 11. Reconnaissance Attack Flow

A typical attack can begin long before the victim realizes anything happened:

```text
Public information
       ↓
WHOIS / DNS information
       ↓
Identify infrastructure
       ↓
Port scanning
       ↓
Identify services
       ↓
Identify software
       ↓
Look for vulnerabilities
       ↓
Launch attack
```

### Security principle

> **Visibility and basic security hygiene matter.**

Reduce unnecessary exposure by:

* Limiting publicly exposed information
* Closing unnecessary ports
* Disabling unnecessary services
* Inventorying reachable services
* Monitoring the network

---

# 12. Malware

## Definition

**Malware** = **malicious software**.

It is an umbrella category rather than one specific type of attack.

The lesson identifies examples including:

* Viruses
* Worms
* Trojan horses
* Ransomware
* Backdoors

---

# 13. Virus

A **virus** is malicious software that can attach itself to other files or programs and spread when the infected host is executed or shared.

Conceptually:

```text
Malicious code
      ↓
Attaches to legitimate file
      ↓
File executed/shared
      ↓
Infection spreads
```

---

# 14. Worm

A **worm** is malware capable of spreading between systems, often without requiring the same type of user action as a traditional virus.

Conceptually:

```text
Infected Host
     ↓
Network
 ┌───┼───┐
 ↓   ↓   ↓
PC  PC  Server
```

This makes worms particularly relevant to network security.

---

# 15. Trojan Horse

A **Trojan horse** is malicious software that masquerades as something legitimate or useful.

The victim believes:

```text
"Useful application"
```

but actually receives:

```text
"Malicious software"
```

The key idea is **deception**.

---

# 16. Ransomware

**Ransomware** is malware that typically encrypts data or systems and demands payment from the victim.

Conceptually:

```text
Initial compromise
       ↓
Malware execution
       ↓
Files encrypted
       ↓
Systems disrupted
       ↓
Ransom demand
```

Impact can include:

* Data unavailable
* Business operations interrupted
* Financial losses
* Recovery costs
* Potential data exposure

---

# 17. Backdoors

A **backdoor** is a hidden mechanism that allows unauthorized access to a system.

The important security concept is **persistence**.

An attacker may compromise a system and then leave another way to regain access.

```text
Initial compromise
       ↓
Attacker gains access
       ↓
Backdoor installed
       ↓
System restored
       ↓
Attacker returns through backdoor
```

Therefore, simply restoring a compromised system may not be enough if the original compromise or persistence mechanism remains.

---

# 18. Malware Is More Than "A Virus"

The lesson emphasizes that malware can involve several behaviors simultaneously.

A compromise could involve:

```text
Malware
 ├── Initial infection
 ├── Persistence
 ├── Surveillance
 ├── Data theft
 ├── Reinfection
 ├── Sabotage
 └── Extortion
```

This is why malware should be viewed as a broad security category.

---

# 19. Security Threat Comparison

| Threat             | Main idea                                 | Typical objective                      |
| ------------------ | ----------------------------------------- | -------------------------------------- |
| **DoS**            | Overwhelm a service                       | Availability                           |
| **Spoofing**       | Pretend to be another device/identity     | Deception                              |
| **MITM**           | Intercept communication                   | Capture/modify traffic                 |
| **Reflection**     | Bounce traffic through other systems      | Increase/hide attack traffic           |
| **Amplification**  | Generate disproportionately large traffic | Increase attack impact                 |
| **DDoS**           | Distributed attack sources                | Deny availability                      |
| **Reconnaissance** | Gather information                        | Prepare for attack                     |
| **Malware**        | Malicious software                        | Damage, steal, spread, persist, extort |

---

# 20. How the Threats Relate

This is the most important conceptual takeaway from today's lesson.

Security attacks can be chained together.

### Example

```text
RECONNAISSANCE
      ↓
Discover exposed DHCP/network services
      ↓
SPOOFING
      ↓
Deploy/impersonate a rogue service
      ↓
MITM
      ↓
Intercept victim traffic
      ↓
MALWARE
      ↓
Compromise endpoint
      ↓
BACKDOOR
      ↓
Maintain persistence
```

Or:

```text
MAC Spoofing
     ↓
Many fake DHCP clients
     ↓
DHCP pool exhaustion
     ↓
DoS
```

Or:

```text
Compromised IoT devices
       ↓
Many attacking sources
       ↓
DDoS
       ↓
Service unavailable
```

---

# 21. NetworkChuck Coffee / Castle Rysen Application

The Castle Rysen RFP specifically requires security controls because district shops contain administrative devices, cameras, Plex servers, and guest devices. The RFP requires segmentation and security boundaries between patron and administrative traffic. 

The RFP also specifically calls for:

* **ACLs**
* **Port Security**
* **DHCP Snooping**
* **Dynamic ARP Inspection (DAI)**
* Wireless security
* VPNs
* SSH restrictions
* Network monitoring



So today's threat concepts directly connect to later configuration work:

| Threat                 | Relevant defense                                       |
| ---------------------- | ------------------------------------------------------ |
| MAC spoofing           | Port Security                                          |
| Rogue DHCP             | DHCP Snooping                                          |
| ARP-related MITM       | DAI                                                    |
| Unauthorized access    | ACLs / AAA                                             |
| Network reconnaissance | Restrict unnecessary services/ports                    |
| Malware                | Segmentation + access control + endpoint security      |
| DoS                    | Rate limiting / infrastructure protection / redundancy |
| MITM                   | Encryption, segmentation, secure management            |

---

# 22. CCNA Exam Memory Sheet

### DoS

**Goal:** Make a service unavailable.

### DDoS

**Goal:** Make a service unavailable using distributed sources.

### Spoofing

**Think:** **Pretending**

### MITM

**Think:** **Intercepting**

### Reflection

**Think:** **Bouncing traffic**

### Amplification

**Think:** **Making traffic bigger**

### Reconnaissance

**Think:** **Information gathering**

### Malware

**Think:** **Malicious software**

### Rogue DHCP

**Think:** **Unauthorized DHCP server**

### SYN Flood

**Think:**

```text
SYN → SYN-ACK → no ACK
```

Many half-open connections → resource exhaustion.

---

# 23. One-Minute Revision

```text
                 SECURITY THREATS
                        │
       ┌────────────────┼────────────────┐
       │                │                │
   Availability      Deception       Information
       │                │                │
      DoS           Spoofing        Reconnaissance
       │                │
     DDoS             MITM
       │
       └──── Reflection / Amplification

                        │
                     Malware
                        │
        ┌───────────────┼───────────────┐
      Virus           Worm          Trojan
                                      │
                                  Ransomware
                                      │
                                   Backdoor
```

## Core takeaway

> **Don't memorize attacks as isolated names. Identify what the attacker is trying to do: overwhelm, impersonate, intercept, gather information, multiply traffic, or deploy malicious software.**

That mental model will make the security configuration lessons that follow much easier to understand.
