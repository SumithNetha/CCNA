# Skill 23 — Lesson 02: Configuring SNMPv2c and SNMPv3

This lesson moves from **understanding SNMP** to actually **enabling a Cisco device to be monitored**.

The most important idea is:

> **Cisco provides the SNMP interface on the network device; the NMS such as PRTG uses that interface to collect information.**

Your Week 16 study plan places this lesson immediately after “What is SNMP?” and before the SNMP/Syslog labs. 

---

# 1. What Are We Actually Configuring?

When you configure SNMP on a Cisco router or switch, you're essentially allowing an external **Network Management System (NMS)** to retrieve management information from the device.

Think about the architecture:

```text
                   NMS
              (PRTG/SolarWinds)
                     |
                     | SNMP
                     |
          +----------+----------+
          |                     |
       Router                 Switch
          |                     |
       OIDs/MIB              OIDs/MIB
          |                     |
     CPU/Interface          Interface/Uptime
     Traffic/Uptime         Traffic/Errors
```

The Cisco device contains the information.

The NMS asks for that information.

SNMP provides the communication mechanism.

---

# 2. The Three Main Components

You should be able to identify these three pieces immediately.

## 2.1 Managed Device

The **managed device** is the network device being monitored.

Examples:

* Cisco router
* Cisco switch
* Firewall
* Wireless access point
* Server
* Other SNMP-capable devices

Example:

```text
Cisco Router
     |
     +-- Interface information
     +-- Uptime
     +-- Traffic counters
     +-- CPU information
     +-- Memory information
```

---

## 2.2 SNMP Agent

The managed device runs an **SNMP agent**.

The agent is responsible for responding to SNMP requests and providing management information.

Conceptually:

```text
NMS
 |
 | "Give me interface traffic."
 ↓
SNMP Agent
 |
 | accesses management information
 ↓
Device data
 |
 ↓
Response
```

Think of the agent as the **SNMP component on the device that communicates with the NMS**.

---

## 2.3 NMS

The **Network Management System** is the system performing monitoring.

Examples mentioned in the lesson include:

* PRTG
* SolarWinds
* Other network-management platforms

The NMS:

1. Identifies the device.
2. Connects using SNMP.
3. Requests management information.
4. Receives values.
5. Stores the values.
6. Creates graphs/trends.
7. Generates alerts.

```text
Device
   ↓
SNMP
   ↓
NMS
   ↓
Database
   ↓
Graphs
   ↓
Alerts
   ↓
Network Engineer
```

---

# 3. What Does Configuring SNMP Actually Expose?

Configuring SNMP doesn't mean:

> "The router is now broadcasting all its information."

Instead, you're enabling an SNMP management interface through which authorized monitoring systems can request supported management information.

That information can include things such as:

* Interface status
* Interface traffic
* Packet counters
* Error counters
* Device uptime
* CPU utilization
* Memory utilization
* Other supported management objects

The exact information available depends on the device/platform and its supported MIBs.

---

# 4. OIDs — The Information Being Requested

From the previous lesson:

**OID = Object Identifier**

An OID identifies a specific management object/data point.

For example:

```text
Device
 |
 +-- OID → Uptime
 |
 +-- OID → Interface status
 |
 +-- OID → Interface traffic
 |
 +-- OID → Packet counters
 |
 +-- OID → Error counters
 |
 +-- OID → CPU
 |
 +-- OID → Memory
```

When the NMS asks the Cisco device for a particular OID, the device returns the corresponding value if it supports it and permits the request.

So:

```text
PRTG
 |
 | "Give me this OID"
 ↓
Cisco Router
 |
 | "Here is its current value"
 ↓
PRTG
```

The monitoring platform then converts the values into something useful.

---

# 5. Cisco Configuration: SNMPv2c

SNMPv2c is comparatively simple to configure.

The lesson describes the fundamental configuration as:

```text
snmp-server community <community-string>
```

The **community string** acts as the shared credential used by the NMS to access SNMP information.

For example, conceptually:

```text
Router(config)# snmp-server community <community-string> ro
```

The important part is:

```text
snmp-server community
```

This establishes the SNMP community.

---

# 6. What Is a Community String?

A community string is essentially a shared secret between the NMS and the SNMP device.

Think:

```text
PRTG
 |
 | "My SNMP credential is X"
 ↓
Cisco Router
 |
 | "X is permitted."
 ↓
SNMP data
```

The NMS needs to use the appropriate community string when communicating with the device.

### Example

Suppose you configure:

```text
snmp-server community MONITOR ro
```

Then the monitoring system would be configured to use:

```text
Community String:
MONITOR
```

The two sides need to agree.

```text
Cisco Router              PRTG
     |                      |
     | community = MONITOR  |
     +----------------------+
```

If the credentials don't match, SNMP monitoring won't work as expected.

---

# 7. Read-Only — `ro`

The lesson emphasizes **read-only monitoring**.

Conceptually:

```text
snmp-server community MONITOR ro
```

Here:

```text
ro = read-only
```

This means the SNMP relationship is intended for retrieving information rather than changing device values.

Think:

```text
NMS
 |
 | GET
 ↓
Router
 |
 | Information
 ↓
NMS
```

The monitoring system can ask:

> "What's your interface traffic?"

> "What's your uptime?"

> "What's the status of this interface?"

But it doesn't receive permission to modify the device through the SNMP relationship.

---

# 8. Read-Write — `rw`

The lesson also introduces read-write access.

Conceptually:

```text
snmp-server community <community-string> rw
```

Here:

```text
rw = read-write
```

This provides more access.

Instead of merely:

```text
READ
```

you potentially have:

```text
READ + WRITE
```

This is a significant security distinction.

### Read-only

```text
"Tell me what's happening."
```

### Read-write

```text
"Tell me what's happening
 AND allow me to change things."
```

Because write access increases the consequences of compromise or misuse, it should not be enabled casually.

---

# 9. Different Community Strings for Different Permissions

The lesson points out that different community strings can be used for different permission levels.

Conceptually:

```text
Community A
     ↓
Read-only

Community B
     ↓
Read-write
```

For example:

```text
NMS Monitoring
      |
      | READ_ONLY
      ↓
   Router

Management system
      |
      | READ_WRITE
      ↓
   Router
```

This allows you to distinguish the access level associated with different SNMP credentials.

For ordinary monitoring, **read-only is the preferred model** from the lesson.

---

# 10. Why SNMPv2c Is Easy

The configuration is relatively small.

Conceptually:

```text
Router(config)#
    snmp-server community <community-string> ro
```

Then the NMS is configured with the same community string.

That's basically the fundamental relationship:

```text
Cisco Device
      |
      | SNMPv2c
      |
      | Community String
      ↓
     NMS
```

This simplicity is one reason SNMPv2c remains common in existing environments.

---

# 11. But Simple Does NOT Mean Secure

This is the biggest weakness of SNMPv2c.

The community string is not protected with the kind of modern authentication/encryption mechanisms provided by SNMPv3.

The lesson characterizes the community string as being sent in **clear text**.

Therefore:

```text
SNMPv2c
    |
    ↓
Community String
    |
    ↓
Clear text
    |
    ↓
Can potentially be captured
```

That creates a security concern.

---

# 12. Why This Matters

Imagine your monitoring network is carrying SNMPv2c traffic.

An attacker capable of capturing that traffic could potentially obtain the community string.

That creates a problem because the community string is functioning as an SNMP credential.

So:

```text
SNMPv2c
   +
Weak credential protection
   ↓
Security risk
```

This is why SNMPv2c isn't the preferred option when stronger security is required.

---

# 13. SNMPv3 — The Secure Alternative

SNMPv3 is more sophisticated.

Instead of simply configuring:

```text
Community String
```

you configure security relationships involving:

1. **Group**
2. **User**
3. **Authentication**
4. **Privacy/encryption**

The lesson's configuration workflow is:

```text
Create group
     ↓
Create user
     ↓
Assign user to group
     ↓
Configure authentication
     ↓
Configure privacy/encryption
```

This is more configuration than SNMPv2c, but it provides significantly stronger security.

---

# 14. SNMPv3 Security Model

The lesson introduces two particularly important concepts:

### Authentication

Authentication answers:

> **"Who are you?"**

It verifies that the SNMP user is legitimate.

---

### Privacy

Privacy is SNMP terminology for protecting the communication through **encryption**.

It answers:

> **"Can someone else read the information while it is traveling across the network?"**

So:

```text
Authentication
      ↓
Verify identity

Privacy
      ↓
Protect information
```

---

# 15. SNMPv3 Security Levels

The lesson explains that an SNMPv3 group can define whether authentication is required and whether authentication plus encryption is required.

The standard SNMPv3 security-level terminology is:

```text
noAuthNoPriv
authNoPriv
authPriv
```

### `noAuthNoPriv`

No authentication and no privacy.

```text
Authentication → No
Encryption     → No
```

---

### `authNoPriv`

Authentication is enabled, but encryption is not.

```text
Authentication → Yes
Encryption     → No
```

You can verify who is communicating, but the data itself isn't encrypted.

---

### `authPriv`

Authentication **and** privacy are enabled.

```text
Authentication → Yes
Encryption     → Yes
```

This is the strongest of these three SNMPv3 security levels.

For a secure deployment, this is generally the level you're aiming for when supported and appropriate.

---

# 16. Authentication Algorithm: SHA

The lesson specifically describes using:

**SHA**

for authentication.

Conceptually:

```text
SNMPv3
 |
 +-- Authentication
       |
       +-- SHA
```

Authentication provides a mechanism to verify the identity/integrity expectations of the SNMP communication.

---

# 17. Privacy Algorithm: AES 128

The lesson also describes using:

**AES 128-bit encryption**

for privacy.

Conceptually:

```text
SNMPv3
 |
 +-- Authentication
 |      |
 |      +-- SHA
 |
 +-- Privacy
        |
        +-- AES 128
```

Therefore, the configuration described in the lesson is essentially:

```text
SNMPv3
   |
   +-- User
   |
   +-- Group
   |
   +-- SHA authentication
   |
   +-- AES-128 privacy
```

---

# 18. SNMPv3 Group

The lesson says you typically create a **group first**.

The group defines the security model/level that members of the group will use.

Think of it like:

```text
SNMPv3 Group
      |
      +-- Security requirements
      |
      +-- Access permissions
      |
      +-- Security level
```

Then users are associated with the group.

---

# 19. SNMPv3 User

Next, you create an SNMPv3 **user**.

The user has credentials and security parameters associated with it.

Conceptually:

```text
SNMPv3 Group
      |
      +------ User
                |
                +-- Username
                +-- Authentication
                +-- Authentication password
                +-- Privacy
                +-- Privacy password
```

The exact Cisco syntax varies with IOS/platform, and the lesson text does not provide the exact command sequence, so the important thing from this lesson is the **configuration structure**, rather than memorizing a command that isn't shown in the source.

---

# 20. SNMPv2c vs SNMPv3 Configuration

The difference becomes very obvious.

### SNMPv2c

```text
Cisco Device
     |
     +-- Community string
     |
     +-- RO/RW
```

Very simple.

### SNMPv3

```text
Cisco Device
     |
     +-- Group
     |
     +-- User
     |
     +-- Authentication
     |      |
     |      +-- SHA
     |
     +-- Privacy
            |
            +-- AES-128
```

More complicated, but significantly more secure.

---

# 21. Side-by-Side Comparison

| Feature                          | SNMPv2c          | SNMPv3                              |
| -------------------------------- | ---------------- | ----------------------------------- |
| Authentication mechanism         | Community string | User/security model                 |
| Encryption                       | No               | Yes, when privacy is configured     |
| Configuration complexity         | Low              | Higher                              |
| Security                         | Weaker           | Stronger                            |
| Monitoring                       | Yes              | Yes                                 |
| Read-only monitoring             | Yes              | Yes                                 |
| Read-write capability            | Yes              | Yes, with appropriate configuration |
| Common in legacy environments    | Very             | Increasingly preferred              |
| Lesson's security recommendation | Less secure      | Preferred for modern environments   |

The lesson's analogy is useful:

```text
SNMPv2c ≈ Telnet
SNMPv3  ≈ SSH
```

The comparison is primarily about **security**:

```text
Older / simpler
      ↓
SNMPv2c

Modern / stronger security
      ↓
SNMPv3
```

---

# 22. Packet Tracer vs Real Cisco Devices

An important warning from the lesson:

> **Packet Tracer gives you a simplified version of SNMP.**

Real Cisco devices provide considerably more SNMP configuration options.

The lesson specifically mentions options involving:

* Access lists
* Views
* Version controls
* Authentication
* Encryption
* Other SNMP controls

So don't assume:

> "Packet Tracer only has a couple of SNMP commands, therefore that's everything Cisco SNMP can do."

It isn't.

Think:

```text
Packet Tracer
     ↓
Simplified learning implementation

Real Cisco IOS
     ↓
Much broader SNMP feature set
```

This distinction is important when moving from CCNA labs to real networking.

---

# 23. What Is the Cisco Device Actually Doing?

Suppose you've configured SNMPv2c.

The process becomes:

```text
                 PRTG
                  |
                  |
           SNMP GET request
                  |
                  ↓
          Cisco Router
                  |
             SNMP Agent
                  |
             Find OID/value
                  |
                  ↓
          SNMP response
                  |
                  ↓
                 PRTG
```

For example:

```text
PRTG:
"Give me interface traffic."

Cisco:
"Current value = X."

PRTG:
"Store that value."

PRTG:
"Plot it on the graph."
```

Repeat that every monitoring interval and you get a historical traffic graph.

---

# 24. Where the MIB Fits

Remember:

**MIB = Management Information Base**

The MIB provides the definitions/catalog of management information.

The OID identifies a particular management object.

So:

```text
MIB
 |
 +-- Interface traffic object
 |       |
 |       +-- OID
 |
 +-- Uptime object
 |       |
 |       +-- OID
 |
 +-- Interface status
         |
         +-- OID
```

The NMS uses its knowledge of these management objects to determine what it can monitor.

This is why a platform such as PRTG can automatically discover interfaces on supported devices.

---

# 25. Connecting Cisco to PRTG

The lesson demonstrates the practical workflow of adding a Cisco router to PRTG.

Conceptually:

### Step 1 — Configure Cisco

```text
Cisco Router
      ↓
Enable SNMP
      ↓
Configure credentials/security
```

### Step 2 — Configure PRTG

```text
PRTG
  ↓
Add device
  ↓
Enter device IP
  ↓
Select SNMP version
  ↓
Enter SNMP credentials
```

### Step 3 — PRTG communicates with Cisco

```text
PRTG
  ↓
SNMP
  ↓
Cisco Router
```

### Step 4 — Add sensors

PRTG can then monitor things such as:

```text
Ping
Uptime
Interface traffic
Interface errors
CPU
Memory
```

depending on device/platform support.

---

# 26. Ping Sensor vs SNMP Sensor

The lesson makes an important distinction.

A **ping sensor** can be useful for determining whether the device is reachable.

But:

> **Ping is not SNMP.**

For example:

```text
Ping
────
PRTG → ICMP → Router
```

while:

```text
SNMP
────
PRTG → SNMP → Router
```

They answer different questions.

### Ping

> "Can I reach this device?"

### SNMP

> "What is happening inside this device?"

You can therefore use both.

```text
Device
 |
 +-- Ping → Availability
 |
 +-- SNMP → Metrics / status / statistics
```

---

# 27. Why Start With Ping?

The lesson recommends starting with a basic ping sensor because you want to know whether the device is reachable.

Imagine:

```text
SNMP sensor → DOWN
```

Before immediately assuming SNMP configuration is broken, you want to know:

```text
Can I even reach the device?
```

So:

```text
Ping
 ↓
Is device reachable?
 ↓
YES
 ↓
Check SNMP
```

This helps separate:

**connectivity problems**

from

**SNMP configuration problems**.

---

# 28. Adding an SNMP Traffic Sensor

Once PRTG can communicate with the device using SNMP, it can discover supported interfaces and associated management information.

For example:

```text
Router
 |
 +-- G0/0
 |
 +-- G0/1
 |
 +-- G0/2
```

The NMS can monitor interface traffic.

Conceptually:

```text
G0/0
 |
 +-- Traffic In
 +-- Traffic Out
 +-- Traffic Total
 +-- Counters
```

This becomes:

```text
SNMP
 ↓
Interface OIDs
 ↓
PRTG
 ↓
Traffic graphs
```

---

# 29. Auto Discovery

The lesson also introduces **auto discovery**.

Auto discovery can automatically find many monitorable objects/sensors.

That's convenient.

But there's a downside.

Suppose a device has:

```text
50 interfaces
+
100 other monitorable objects
```

Automatic discovery could produce a huge number of sensors.

You might end up monitoring things you don't actually care about.

---

# 30. Why Too Many Sensors Can Be Bad

More monitoring isn't automatically better.

Imagine:

```text
Network
   ↓
1000 sensors
   ↓
Huge amount of data
   ↓
Alerts everywhere
   ↓
Noise
   ↓
Important alert gets buried
```

This is called **alert/data noise** conceptually.

The lesson recommends being intentional.

Start with the high-value information.

---

# 31. Recommended Starting Sensor Set

The lesson specifically recommends starting with:

### 1. Ping

```text
Is the device reachable?
```

### 2. Uptime

```text
How long has it been running?
```

### 3. Interface traffic

```text
How much traffic is moving?
```

### 4. Interface errors

```text
Are interfaces experiencing errors?
```

### 5. CPU / Memory

If supported:

```text
Is the device itself under resource pressure?
```

So:

```text
             Core Sensors
                  |
      +-----------+-----------+
      |           |           |
     Ping       Uptime      Traffic
                              |
                       +------+------+
                       |             |
                    Errors       CPU/Memory
```

This gives you a useful initial view without unnecessarily monitoring everything.

---

# 32. Why Interface Errors Matter

Traffic utilization tells you **how much traffic** is passing through an interface.

Errors can tell you something different:

> **Whether the interface is experiencing transmission/reception problems.**

Imagine:

```text
Traffic = Normal
Errors  = Increasing
```

The interface isn't necessarily overloaded, but something could still be wrong.

Therefore, monitoring both:

```text
Traffic
+
Errors
```

provides a better picture of interface health.

---

# 33. Why CPU and Memory Matter

A network device can have healthy interfaces but still experience resource problems.

For example:

```text
Interfaces → UP
Traffic    → Normal
CPU        → 95%
```

This is useful evidence.

Similarly:

```text
CPU       → Normal
Memory    → 95%
```

could indicate resource pressure.

The lesson therefore recommends monitoring CPU and memory **if the platform supports those metrics**.

---

# 34. Proactive vs Reactive Networking

This is the deeper purpose behind the entire lesson.

### Reactive model

```text
Something breaks
       ↓
Users complain
       ↓
Engineer investigates
       ↓
Engineer finds problem
       ↓
Fix
```

### Proactive model

```text
Monitoring
     ↓
Trend detected
     ↓
Warning
     ↓
Engineer investigates
     ↓
Problem prevented or reduced
```

SNMP enables the second approach.

---

# 35. NetworkChuck Coffee Example

Imagine the coffee shop has an access point.

Over several days:

```text
Monday     → 50% utilization
Tuesday    → 55%
Wednesday  → 62%
Thursday   → 75%
Friday     → 90%
```

Nothing has necessarily failed yet.

But monitoring tells you:

> **The AP is approaching a potential capacity problem.**

You can investigate before customers start complaining.

That's what the lesson means by becoming **proactive**.

---

# 36. Castle Rysen Application

This lesson fits directly into the Castle Rysen network project.

The RFP requires monitoring of essential elements of each coffeehouse network so that trends can be tracked and local/Internet connectivity maintained. 

It also explicitly requires:

> **SNMP and syslog** for network monitoring and management. 

Therefore, a production-oriented Castle Rysen design could conceptually look like:

```text
                     NMS
                      |
             +--------+--------+
             |        |        |
           SNMP     SNMP     SNMP
             |        |        |
           Router   Switch     AP
             |        |        |
          Metrics  Metrics   Metrics
```

The NMS can then provide centralized visibility across:

* Central Office
* Fallout Shelters
* District Shops

This is especially important because the RFP expects the network to scale across multiple shelters and shops. 

---

# 37. Security Perspective

SNMP itself is a management protocol, so you should treat SNMP access as **management-plane traffic**.

Don't think:

> "It's only monitoring, so security doesn't matter."

Monitoring credentials can provide valuable access to information about your infrastructure.

For SNMPv2c:

```text
Community string
      ↓
Shared secret
      ↓
Weak protection
      ↓
Potential exposure
```

For SNMPv3:

```text
Username
   +
Authentication
   +
Encryption
   ↓
Stronger management security
```

This is why SNMPv3 is preferable where supported and practical.

---

# 38. The Most Important Configuration Difference

If you are asked in an interview:

### "How do you configure SNMPv2c?"

Answer conceptually:

```text
Configure an SNMP community string
and specify the access level, typically
read-only for monitoring.
```

### "How do you configure SNMPv3?"

Answer:

```text
Create an SNMPv3 group with the required
security level, create a user associated
with that group, and configure authentication
and privacy/encryption.
```

That's the conceptual difference the lesson wants you to understand.

---

# 39. Complete SNMPv2c Flow

```text
                  PRTG
                   |
             Community String
                   |
                   ↓
             SNMPv2c
                   |
                   ↓
             Cisco Router
                   |
              SNMP Agent
                   |
                   ↓
             OID / MIB data
                   |
                   ↓
                  PRTG
                   |
          +--------+--------+
          |        |        |
        Graphs   Trends   Alerts
```

---

# 40. Complete SNMPv3 Flow

```text
                  PRTG
                   |
            SNMPv3 User
                   |
          Authentication
             +     |
          Privacy/Encryption
                   |
                   ↓
             Cisco Router
                   |
              SNMP Agent
                   |
                   ↓
             OID / MIB data
                   |
                   ↓
                  PRTG
```

---

# 41. What You Should Memorize

## SNMPv2c

```text
SNMPv2c
   ↓
Community string
   ↓
RO / RW
   ↓
Simple
   ↓
No modern encryption
   ↓
Weaker security
```

## SNMPv3

```text
SNMPv3
   ↓
Group
   ↓
User
   ↓
Authentication
   ↓
Privacy
   ↓
Encryption
   ↓
Stronger security
```

---

# 42. Important Terms

| Term                 | Meaning                                                                      |
| -------------------- | ---------------------------------------------------------------------------- |
| **NMS**              | Network Management System                                                    |
| **SNMP Agent**       | Component on the managed device that handles SNMP management communication   |
| **Community String** | Shared credential used by SNMPv1/v2c                                         |
| **OID**              | Object Identifier; identifies a management object/data point                 |
| **MIB**              | Management Information Base; collection/definition of management information |
| **RO**               | Read-only                                                                    |
| **RW**               | Read-write                                                                   |
| **SNMPv2c**          | SNMP version using community strings                                         |
| **SNMPv3**           | SNMP version with stronger security capabilities                             |
| **Authentication**   | Verifies identity/credentials                                                |
| **Privacy**          | SNMP term for protecting data through encryption                             |
| **SHA**              | Authentication algorithm mentioned in the lesson                             |
| **AES-128**          | Encryption algorithm mentioned for SNMPv3 privacy                            |
| **Polling**          | NMS repeatedly requesting information                                        |
| **Trap**             | Device-generated SNMP notification to the NMS                                |
| **Sensor**           | Monitoring item in an NMS such as PRTG                                       |

---

# 43. Final Mental Model

The whole lesson can be reduced to this:

```text
                    NETWORK DEVICE
                          |
                    SNMP Agent
                          |
             +------------+------------+
             |                         |
          SNMPv2c                   SNMPv3
             |                         |
      Community String          Group + User
             |                         |
        RO / RW                  Authentication
                                       +
                                    Privacy
                                       |
                                  AES-128 etc.
             |                         |
             +------------+------------+
                          |
                         NMS
                    (e.g. PRTG)
                          |
              +-----------+-----------+
              |           |           |
            OIDs        Graphs      Alerts
              |           |           |
            Data       History     Events
```

### The progression you should understand

**SNMP**
→ protocol for network monitoring/management

**OID**
→ individual piece of management information

**MIB**
→ organized catalog/definition of management information

**NMS**
→ system that collects and presents the information

**SNMPv2c**
→ community-string based, simple, weaker security

**SNMPv3**
→ user/group-based security with authentication and privacy

**PRTG**
→ NMS that uses SNMP to turn those device values into sensors, graphs, trends, and alerts

And the operational goal is:

> **Don't merely configure a network so that it works. Configure it so that you can see when it stops working—and ideally see the warning signs before it stops working.**
