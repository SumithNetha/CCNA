![alt text](image.png)

# Skill 23 — Lesson 01: What Is SNMP?

## 1. What is SNMP?

**SNMP = Simple Network Management Protocol**

At its simplest, SNMP is a protocol that allows a management system to **ask network devices for information and receive values in return**.

Think of it as:

> **SNMP = "Give me the value of this thing."**

For example, a monitoring system could ask a switch:

```text
"What is your CPU utilization?"
"What is the traffic on interface GigabitEthernet0/1?"
"How long have you been running?"
```

The device responds with the requested information.

So the basic model is:

```text
NMS
 |
 | SNMP request
 ↓
Router / Switch
 |
 | SNMP response
 ↓
NMS
```

The lesson's central idea is:

> **SNMP turns mystery into visibility.**

Instead of relying on user complaints or manually checking every device, you can collect measurable information from the network.

---

# 2. Why SNMP Is Important

Suppose you're managing a coffee shop network.

Users report:

> "The Wi-Fi is slow."

That's not enough information to identify the problem.

There could be many possibilities:

```text
                    Wi-Fi Slow
                        |
       +----------------+----------------+
       |                |                |
      AP              Switch           WAN
    overloaded       overloaded       saturated
       |                |                |
       +----------------+----------------+
                        |
                    Other causes
                 DHCP / DNS / etc.
```

SNMP allows you to collect actual information.

For example:

```text
AP CPU              → 92%
Switch interface    → 95% utilization
WAN utilization     → 87%
Device uptime       → 3 days
```

Now troubleshooting becomes much more precise.

Instead of:

> "Something is wrong."

You have:

> "This interface has been heavily utilized during the period users reported the problem."

---

# 3. SNMP Is Behind Many Monitoring Systems

The lesson points out that monitoring platforms such as:

* SolarWinds
* PRTG
* Nagios

can provide graphs, dashboards, alerts, and reports.

But these tools aren't necessarily doing something magical.

A major part of the process is:

```text
Network Devices
      |
      | SNMP
      ↓
Monitoring System
      |
      +---- Graphs
      +---- Trends
      +---- Alerts
      +---- Reports
      +---- Dashboards
```

The monitoring platform repeatedly collects information from network devices and turns that information into something humans can interpret.

---

# 4. OID — Object Identifier

One of the most important terms in this lesson is:

**OID = Object Identifier**

An OID identifies a specific piece of information that can be queried.

The easiest way to remember it:

> **OID = one specific data point**

Examples from the lesson:

* CPU usage
* Interface bandwidth
* Device uptime
* Interface status
* Temperature

can each correspond to measurable information represented through OIDs.

Think of an OID as a **specific sensor/data item**.

```text
Device
 |
 +-- OID → CPU usage
 |
 +-- OID → Memory
 |
 +-- OID → Interface traffic
 |
 +-- OID → Uptime
 |
 +-- OID → Temperature
```

So when an NMS asks for a particular OID, it is effectively asking:

> **"Give me the current value associated with this particular data point."**

---

# 5. MIB — Management Information Base

Now we have another acronym:

**MIB = Management Information Base**

If:

> **OID = one data point**

then:

> **MIB = the catalog/library of available data points**

The lesson describes the MIB like a catalog or book containing information about the OIDs supported by a device.

Think about it like this:

```text
MIB
 |
 +-- OID → CPU
 |
 +-- OID → Memory
 |
 +-- OID → Interface 1
 |
 +-- OID → Interface 2
 |
 +-- OID → Uptime
 |
 +-- OID → Temperature
```

### Easy analogy

Imagine a restaurant menu.

```text
Menu = MIB

Coffee = OID
Burger = OID
Pizza  = OID
Cake   = OID
```

The **MIB** tells you what items/data points are available.

The **OID** identifies the specific item you want.

---

# 6. OID vs MIB — Very Important

| Term    | Meaning                     | Easy way to remember           |
| ------- | --------------------------- | ------------------------------ |
| **OID** | Object Identifier           | One specific data point        |
| **MIB** | Management Information Base | Catalog/library of data points |

### Remember:

```text
MIB
 ↓
contains/describes
 ↓
OIDs
 ↓
identify
 ↓
specific information
```

So don't confuse:

**OID ≠ MIB**

An OID identifies an individual object/data point.

A MIB provides the organized definitions/catalog for management information.

---

# 7. Vendor MIBs

Different devices can expose different information.

For example, a switch manufacturer may provide a MIB describing the management objects supported by that device.

This allows monitoring software to understand what information can be obtained from the device.

The lesson gives examples such as:

```text
Switch
Router
Server
IoT Device
```

Each may have relevant MIB information describing the objects it supports.

---

# 8. NMS — Network Management System

Now we get to the component that actually **does the asking**.

**NMS = Network Management System**

The NMS is the monitoring/management platform that communicates with network devices.

Think:

```text
              NMS
               |
        "Give me this OID"
               |
              SNMP
               |
               ↓
         Network Device
               |
        "Here is the value"
               |
              SNMP
               |
               ↓
              NMS
```

The NMS can repeatedly query devices and collect their information.

---

# 9. SNMP Polling

The NMS doesn't normally ask just once.

It can **poll** devices at regular intervals.

For example:

```text
09:00 → Ask switch for interface traffic
09:01 → Ask switch for interface traffic
09:02 → Ask switch for interface traffic
09:03 → Ask switch for interface traffic
09:04 → Ask switch for interface traffic
```

The lesson gives examples of intervals such as:

* 30 seconds
* 60 seconds

The exact interval depends on the monitoring system/configuration.

---

# 10. Why Repeated Polling Is Powerful

Suppose the NMS collects interface utilization every minute:

```text
Time       Utilization
----------------------
07:00          25%
07:15          31%
07:30          38%
07:45          45%
08:00          62%
08:15          78%
08:30          91%
```

A single measurement tells you the **current state**.

Repeated measurements give you a **history**.

That history can produce:

* Graphs
* Trends
* Baselines
* Alerts
* Performance analysis

This is where SNMP becomes much more useful than simply asking a device one question.

---

# 11. Baselines

Repeated monitoring allows you to establish what is **normal**.

For example, suppose a coffee shop's WAN normally operates around:

```text
30–50% utilization
```

But suddenly:

```text
95%
```

That is significant because you know what normal behavior looks like.

You can compare:

```text
Normal behavior
       ↓
Historical data
       ↓
Current behavior
       ↓
Difference detected
       ↓
Investigate
```

The lesson specifically emphasizes using historical data to identify recurring patterns.

For example:

> Network performance becomes poor every morning around 7:15 a.m.

SNMP history can help reveal that pattern.

---

# 12. SNMP Operations

SNMP supports several types of operations.

The lesson introduces three particularly important ones:

```text
GET
SET
TRAP
```

---

## 12.1 GET

**GET = request information from a device**

This is the common read-oriented operation.

Example:

```text
NMS
 |
 | GET: "Give me interface traffic"
 ↓
Switch
 |
 | Response: value
 ↓
NMS
```

The key idea:

> **GET = read information**

This is why SNMP is useful for monitoring.

---

# 13. SET

**SET = change a value on the device**

This is fundamentally different from GET.

### GET

```text
NMS → "Tell me the value."
```

### SET

```text
NMS → "Change the value."
```

Therefore, SET introduces considerably more risk.

For example:

```text
GET
 ↓
Read-only monitoring
 ↓
Lower risk
```

versus:

```text
SET
 ↓
Modify device state
 ↓
Higher trust / higher risk
```

The lesson recommends being cautious with read-write SNMP access and generally favoring **read-only monitoring** unless there is a specific reason to allow changes.

---

# 14. TRAP

A **TRAP** is different from polling.

With normal polling:

```text
NMS → Device
     "What's happening?"
```

With a trap:

```text
Device → NMS
         "Something happened!"
```

So a trap is essentially an event notification sent from the device to the monitoring system.

For example:

```text
Switch
   |
   | TRAP
   ↓
NMS

"Important event occurred."
```

This is useful because the NMS doesn't necessarily have to wait until its next polling cycle to learn about certain events.

---

# 15. GET vs SET vs TRAP

| Operation | Direction/idea                | Purpose                      |
| --------- | ----------------------------- | ---------------------------- |
| **GET**   | NMS asks device               | Retrieve information         |
| **SET**   | NMS changes device            | Modify a value/configuration |
| **TRAP**  | Device sends NMS notification | Notify about an event        |

### Easy memory trick

```text
GET   → Give me information
SET   → Change something
TRAP  → Something happened!
```

---

# 16. SNMP Versions

The lesson focuses on three versions you may encounter:

* **SNMPv1**
* **SNMPv2c**
* **SNMPv3**

The two major versions you need to pay particular attention to are:

**SNMPv2c and SNMPv3**

---

# 17. SNMPv1

SNMPv1 is the original version.

You may still encounter it on:

* Older devices
* Legacy equipment
* Some low-powered/older systems

The important practical lesson is:

> **Old technology can still exist in real networks.**

So when troubleshooting, don't automatically assume every device supports modern features.

---

# 18. SNMPv2c

SNMPv2c is widely encountered because it is relatively simple to deploy.

It uses a:

**Community string**

The community string functions as a shared credential/secret used for SNMP access.

Conceptually:

```text
NMS
 |
 | SNMPv2c
 | Community String
 ↓
Device
```

However, there is a major security weakness.

### Clear-text communication

The lesson states that the community string is sent in **clear text**.

Therefore, if someone can capture the SNMP traffic, the community string could potentially be exposed.

That makes SNMPv2c much weaker from a security perspective than SNMPv3.

---

# 19. SNMPv3

**SNMPv3** was designed to provide stronger security.

The lesson highlights:

* **Authentication**
* **Encryption**

This makes SNMPv3 much more appropriate for modern secure environments.

Conceptually:

```text
NMS
 |
 | SNMPv3
 | Authentication
 | Encryption
 ↓
Device
```

Instead of simply relying on a clear-text community string, SNMPv3 provides security mechanisms designed to protect management communication.

---

# 20. SNMPv2c vs SNMPv3

This is a key comparison.

| Feature                    | SNMPv2c                     | SNMPv3                           |
| -------------------------- | --------------------------- | -------------------------------- |
| Community string           | Yes                         | No as the primary security model |
| Authentication             | Weak/shared community model | Yes                              |
| Encryption                 | No                          | Yes                              |
| Security                   | Lower                       | Higher                           |
| Deployment                 | Simple                      | More involved                    |
| Modern secure environments | Less desirable              | Preferred when supported         |

The lesson's analogy is:

```text
SNMPv2c ≈ Telnet
SNMPv3  ≈ SSH
```

The point of the analogy is **security**, not that the protocols perform identical functions.

---

# 21. Why SNMPv3 Is Better for Production

Consider a management network.

You don't want monitoring credentials exposed to someone who can capture traffic.

With SNMPv2c:

```text
NMS
 |
 | Community string
 |------------------------>
 |
Device
```

The lesson describes this communication as clear text.

With SNMPv3:

```text
NMS
 |
 | Authentication
 | + Encryption
 |------------------------>
 |
Device
```

This provides significantly stronger protection for SNMP management traffic.

---

# 22. Read-Only vs Read-Write SNMP

Another important security concept is **access level**.

### Read-only

The monitoring system can ask:

```text
"What is your CPU?"
"What is your uptime?"
"What is interface utilization?"
```

But it cannot use SNMP to change those values.

### Read-write

The management system can potentially:

```text
Read information
+
Change supported values
```

The lesson strongly emphasizes the difference in trust.

```text
Read-only
   ↓
"Tell me what's happening."

Read-write
   ↓
"Tell me what's happening
 AND let me change things."
```

For normal monitoring, **read-only is generally the safer choice**.

---

# 23. SNMP Doesn't Equal the Dashboard

This is an important conceptual point.

When you look at a monitoring dashboard, you might see:

```text
CPU       ███████░░░ 70%
Memory    █████░░░░░ 50%
Traffic   ████████░░ 80%
Uptime    25 days
```

The dashboard isn't the fundamental technology.

Underneath, the monitoring system is collecting information from network devices.

Conceptually:

```text
Network Device
      ↓
     SNMP
      ↓
     OIDs
      ↓
     NMS
      ↓
Historical data
      ↓
Graphs / Alerts / Dashboards
```

So the dashboard is the **presentation layer**.

SNMP provides a mechanism for obtaining management information.

---

# 24. SNMP and Syslog Are Different

From the previous lesson, remember that SNMP and syslog complement one another.

```text
SNMP
 ↓
Metrics / status / monitoring

Syslog
 ↓
Events / messages / context
```

For example:

```text
SNMP:
Interface utilization = 95%

        +

Syslog:
Interface state changed at 09:03

        ↓

Better troubleshooting context
```

SNMP tells you about measurable state.

Syslog provides event information that can help explain what happened.

---

# 25. Complete SNMP Architecture

Put everything together:

```text
                         NMS
                          |
                    SNMP Requests
                          |
                          ↓
                +------------------+
                | Network Device   |
                |                  |
                |  MIB             |
                |   |              |
                |   +-- OID        |
                |   +-- OID        |
                |   +-- OID        |
                |   +-- OID        |
                +------------------+
                          |
                    SNMP Response
                          |
                          ↓
                         NMS
                          |
              +-----------+-----------+
              |           |           |
            Graphs      Trends      Alerts
```

And when an important event occurs:

```text
Network Device
      |
    TRAP
      ↓
     NMS
```

---

# 26. Complete Mental Model

If you remember only one diagram from this lesson, remember this:

```text
                         SNMP
                          |
            +-------------+-------------+
            |                           |
       NMS asks                  Device can notify
            |                           |
           GET                         TRAP
            |                           |
            ↓                           ↓
       Device data                   NMS
            |
            ↓
          OID
            |
            ↓
     Specific data point
            |
            ↓
          MIB
    Catalog of information
            |
            ↓
           NMS
            |
     +------+------+------+
     |      |      |      |
   Graphs Trends Baseline Alerts
```

---

# 27. SNMP in Castle Rysen

The Castle Rysen RFP specifically requires network monitoring and calls for **SNMP and syslog** to monitor and manage the network. 

Imagine the environment:

```text
                    NMS
                     |
          +----------+----------+
          |          |          |
       Central    Fallout    District
        Office    Shelter      Shop
          |          |          |
       SNMP       SNMP       SNMP
```

The NMS can collect information from network devices throughout the infrastructure.

That supports the RFP's requirement to monitor essential network elements and track trends to maintain connectivity. 

---

# 28. Exam/Interview-Level Distinctions

Make sure you can immediately answer these:

### What does SNMP stand for?

**Simple Network Management Protocol**

### What is an OID?

**An Object Identifier representing a specific management data point/object.**

### What is a MIB?

**Management Information Base — a catalog/definition of management information and the objects available to query.**

### What is an NMS?

**Network Management System — the system that monitors/manages network devices and performs SNMP queries.**

### What is GET?

**Retrieves information.**

### What is SET?

**Changes a supported value.**

### What is a TRAP?

**An unsolicited notification sent by a device to the NMS about an event.**

### What does SNMPv2c use?

**Community strings.**

### What is the major security problem with SNMPv2c?

**The community string is transmitted in clear text.**

### What does SNMPv3 add?

**Authentication and encryption/security mechanisms.**

### Which is more secure?

**SNMPv3.**

---

# 29. The Most Important Things to Memorize

```text
SNMP
= Simple Network Management Protocol

NMS
= Network Management System
= asks/collects information

OID
= Object Identifier
= one specific data point

MIB
= Management Information Base
= catalog of management information/OIDs

GET
= read

SET
= change

TRAP
= device → NMS event notification

SNMPv2c
= community string
= clear text
= weaker security

SNMPv3
= authentication
= encryption
= stronger security
```

## The big picture

**SNMP is fundamentally a mechanism for obtaining management information from network devices.**

The workflow is:

```text
NMS
 ↓
SNMP request
 ↓
OID
 ↓
Device
 ↓
Value returned
 ↓
NMS
 ↓
Historical data
 ↓
Graphs / trends / alerts
```

And that's the foundation for the next lesson in your Week 16 plan: **Configuring SNMPv2c and SNMPv3**. Your study plan places that lesson immediately after this one on August 17, with the hands-on configuration lab and quiz on August 18. 
