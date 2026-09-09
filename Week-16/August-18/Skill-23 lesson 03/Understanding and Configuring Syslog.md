# Week 16 — August 18

# Skill 23 Lesson 03 — Understanding and Configuring Syslog

## 1. What Is Syslog?

**Syslog is a logging mechanism used by network devices to report events and messages about what is happening on the device.**

Think of a Cisco device as constantly generating a stream of events:

* Interface went down.
* Interface came back up.
* Device rebooted.
* Authentication attempt occurred.
* Configuration changed.
* An error occurred.
* A warning was generated.
* Debugging information was produced.

Syslog provides a standardized way to **record, display, and forward these messages**.

### Simple definition

> **Syslog = Network devices reporting events and operational messages.**

This is different from SNMP.

| SNMP                            | Syslog                     |
| ------------------------------- | -------------------------- |
| Primarily monitoring/management | Primarily event logging    |
| Gives metrics/status            | Gives event messages       |
| CPU utilization                 | Interface went down        |
| Memory utilization              | Authentication event       |
| Interface statistics            | Configuration/system event |
| "How is the device doing?"      | "What happened?"           |

A useful mental model is:

**SNMP → Pulse of the network**

**Syslog → Story of the network**

---

# 2. Why Do We Need Syslog?

Imagine Castle Rysen has:

* 1 central office
* 2 fallout shelters
* 50 district shops
* Multiple routers
* Multiple switches
* Wireless infrastructure
* Firewalls
* Servers

If something goes wrong at 7:03 AM, you don't want to SSH into every device individually and inspect its logs.

You want the devices to automatically report their events to a **central logging system**.

For example:

```text
Router-01 ─────┐
Switch-01 ─────┤
Switch-02 ─────┤
Router-02 ─────┼──→ Syslog Server
Switch-03 ─────┤
WAP-01 ────────┘
```

Now the administrator has one centralized location for investigating events.

---

# 3. Syslog Messages

A Syslog message generally contains several important pieces of information.

The lesson identifies three major components:

### 1. Facility / Category

Identifies the general source or category responsible for generating the message.

It helps determine **what part of the system generated the message**.

### 2. Severity

Indicates how serious the event is.

Syslog has **8 severity levels**, numbered:

```text
0 → Emergency
1 → Alert
2 → Critical
3 → Error
4 → Warning
5 → Notice
6 → Informational
7 → Debugging
```

### 3. Message Text

This is the human-readable description of what actually happened.

For example, a message might tell you that an interface changed state.

---

# 4. Syslog Severity Levels

You should memorize the levels because they are important both for CCNA and practical network administration.

| Level | Name          | Meaning                                |
| ----: | ------------- | -------------------------------------- |
| **0** | Emergency     | System is unusable / extremely serious |
| **1** | Alert         | Immediate action is required           |
| **2** | Critical      | Critical condition                     |
| **3** | Error         | Error condition                        |
| **4** | Warning       | Warning condition                      |
| **5** | Notice        | Significant but normal condition       |
| **6** | Informational | General information                    |
| **7** | Debugging     | Detailed troubleshooting information   |

### Severity order

The lower the number, the **more severe** the message.

```text
0  Emergency      ← Most severe
1  Alert
2  Critical
3  Error
4  Warning
5  Notice
6  Informational
7  Debugging      ← Least severe / most detailed
```

### Important

Do **not** interpret `7` as "more serious" because it is numerically larger.

It is the opposite.

```text
0 = highest severity
7 = lowest severity
```

But Debugging can generate **a lot of information**, so it can be very noisy.

---

# 5. A Simple Example

Suppose a router's interface goes down.

The router generates a Syslog event describing the interface state change.

Conceptually:

```text
Timestamp
    ↓
Hostname
    ↓
Severity
    ↓
Message
    ↓
Interface changed state
```

An administrator can then determine:

* **When** did it happen?
* **Which device** reported it?
* **How serious** was it?
* **What happened?**

This is why timestamps and device identity become extremely important when troubleshooting.

---

# 6. Local Logging

Cisco devices can maintain logs locally.

Commands such as:

```text
show log
```

or

```text
show logging
```

allow you to inspect logging information.

The device can store messages in a **logging buffer in memory**.

Conceptually:

```text
Cisco Router
     │
     ├── Event
     ├── Event
     ├── Event
     └── Event
          ↓
    Memory Buffer
```

This is useful for troubleshooting.

For example:

```text
Something broke
      ↓
Log into router
      ↓
show logging
      ↓
Look at previous events
      ↓
Determine what happened
```

---

# 7. The Problem With Local Logs

Local logging has a major limitation:

## The logs may disappear when the device reboots.

Because the logs are commonly stored in memory, a reload can cause those locally stored messages to disappear.

Consider:

```text
Router
  │
  └── Logs stored in RAM
          │
          ↓
       Router reloads
          │
          ↓
       Logs lost
```

That becomes a serious problem during troubleshooting.

Imagine:

```text
07:00 → Network operating normally
07:03 → Interface fails
07:04 → Router crashes
07:05 → Router reboots
07:06 → Administrator investigates
```

If the useful evidence was only stored locally in memory, some of the information from 07:03 may no longer be available.

---

# 8. Another Problem: Too Many Devices

Local logging also creates a scalability problem.

Suppose you have:

```text
1 router
```

Checking:

```text
show logging
```

is easy.

But imagine:

```text
50 routers
100 switches
50 WAPs
20 firewalls
```

You don't want to manually connect to every device.

You need **centralized logging**.

---

# 9. Centralized Syslog

This is where a **Syslog server** comes into the picture.

Instead of keeping all logs only on individual devices, network devices can send their messages to a central server.

```text
             ┌── Router
             │
             ├── Switch
             │
             ├── Firewall
             │
             ├── WAP
             │
             └── Server
                   │
                   ↓
             Syslog Server
                   │
                   ↓
        Centralized log storage
```

This is called:

## Off-box logging

**Off-box logging** means that logs leave the device that generated them and are stored somewhere else.

---

# 10. Why Off-Box Logging Is Better

Centralized logging provides several advantages.

### 1. Centralized visibility

Instead of checking every device individually:

```text
Router → logs
Switch → logs
Firewall → logs
```

you have:

```text
Router ──┐
Switch ──┤
Firewall ┼──→ Central Syslog Server
WAP ─────┤
Server ──┘
```

---

### 2. Historical information

A central logging system can retain logs for later investigation.

This is extremely useful when the problem doesn't happen continuously.

For example:

```text
Monday
10:00 → Interface normal
14:32 → Interface down
14:33 → Interface up

Tuesday
09:15 → Interface down
09:16 → Interface up

Wednesday
17:45 → Interface down
17:46 → Interface up
```

You can recognize that the interface is **flapping**.

---

### 3. Correlation

This is one of the most important benefits.

Suppose several devices report events around the same time:

```text
07:03:12  Switch-A → Interface down
07:03:13  Router-A → Route changed
07:03:15  Firewall-A → Connection failures
07:03:18  Switch-B → Interface down
```

When those events are centralized, an administrator can correlate them.

Instead of seeing isolated events, you can construct a timeline.

---

# 11. Syslog Server

A **Syslog server** is a system that receives Syslog messages from network devices and stores them.

The lesson uses a Synology system with a logging package as an example.

Other enterprise logging platforms can also perform this role.

Conceptually:

```text
Cisco Device
     │
     │ Syslog message
     ↓
Network
     │
     ↓
Syslog Server
     │
     ├── Store
     ├── Search
     ├── Filter
     └── Correlate
```

The important concept isn't the particular server product.

It is:

> **Network devices send logs to a centralized system where the logs can be retained and analyzed.**

---

# 12. Syslog Transport

The lesson identifies the default Syslog transport as:

## UDP port 514

Remember:

```text
Syslog
   ↓
UDP
   ↓
Port 514
```

This is an important CCNA fact.

### Why UDP?

The lesson focuses on the fact that Syslog commonly uses UDP 514 for sending log messages.

The administrator therefore needs to make sure the network allows the Syslog traffic between the device and the logging server.

---

# 13. Basic Cisco Syslog Configuration

The fundamental Cisco configuration is straightforward.

From global configuration mode:

```text
logging <SYSLOG-SERVER-IP>
```

For example:

```text
Router(config)# logging 192.168.10.100
```

This tells the router:

> Send logging messages to the Syslog server at `192.168.10.100`.

The overall process is:

```text
Router
  ↓
logging 192.168.10.100
  ↓
Router knows Syslog server
  ↓
Events generated
  ↓
Messages forwarded
  ↓
Syslog Server
```

---

# 14. Verifying Syslog

Use:

```text
show logging
```

This allows you to inspect the device's logging information and verify the logging configuration/activity.

Conceptually:

```text
Configuration
      ↓
show logging
      ↓
Check logging status
      ↓
Check configured server
      ↓
Check generated messages
```

---

# 15. How Do You Test Syslog?

A good test is to deliberately create a harmless event.

For example, take an unused interface and enable it:

```text
interface GigabitEthernet0/1
 no shutdown
```

If the interface changes state, the device should generate a logging message.

The event can then be observed on the centralized Syslog server.

The important troubleshooting principle is:

> **Don't just configure monitoring—generate a known event and verify that the monitoring system actually sees it.**

---

# 16. What Information Does the Syslog Server Give You?

Centralized logging allows you to build a timeline.

For example:

```text
Time        Device       Severity       Event
-------------------------------------------------------
07:03:01    SW-01        Notice         Interface event
07:03:02    R1           Error           Routing event
07:03:04    FW-01        Warning        Security event
07:03:06    SW-02        Informational  Interface event
```

Now you can ask:

* What happened first?
* Which device detected it?
* Was it an error or warning?
* Did multiple devices experience problems?
* Did the events happen simultaneously?

That is the real value of centralized logging.

---

# 17. Syslog and Troubleshooting

Syslog is extremely useful for troubleshooting because it provides **historical evidence**.

Suppose users report:

> "The network keeps disconnecting."

You don't necessarily know when it happens.

Syslog can reveal repeated events:

```text
08:15 → Interface down
08:16 → Interface up

09:42 → Interface down
09:43 → Interface up

11:07 → Interface down
11:08 → Interface up
```

This points you toward an unstable link or another recurring condition.

Without logging, you might only see the interface working normally when you investigate it.

---

# 18. Syslog and Security

Syslog is also useful for security monitoring.

For example, suppose multiple devices report authentication-related events:

```text
Router-01 → failed login
Switch-01 → failed login
Switch-02 → failed login
Firewall → suspicious activity
```

Centralized logging allows an administrator to investigate these events together.

This is particularly useful for identifying:

* Repeated login failures
* Unauthorized activity
* Configuration changes
* Device reboots
* Unexpected events

The key idea is:

**Security events become much more useful when you have historical records and timestamps.**

---

# 19. Syslog and Interface Flapping

A common network troubleshooting scenario is **interface flapping**.

Interface flapping means an interface repeatedly changes between up and down.

For example:

```text
UP
 ↓
DOWN
 ↓
UP
 ↓
DOWN
 ↓
UP
```

Syslog can expose this pattern.

You might discover:

```text
10:00:12 → Interface down
10:00:14 → Interface up
10:00:25 → Interface down
10:00:27 → Interface up
10:00:41 → Interface down
10:00:43 → Interface up
```

This is far more informative than simply checking the interface once.

---

# 20. Syslog and Device Reboots

Suppose a router unexpectedly reloads.

Centralized logs can help establish a timeline around the event.

You might find:

```text
Event A
   ↓
Event B
   ↓
Unexpected device event
   ↓
Device reboot
   ↓
Device comes back online
```

If the logs were stored only locally and the device rebooted, some evidence might be lost.

That is why **centralized logging should be configured before an incident occurs**, not after.

---

# 21. Syslog vs. SNMP

This distinction is extremely important.

### SNMP

SNMP is useful for answering:

> **"How is my device doing?"**

Examples:

```text
CPU = 85%
Memory = 72%
Interface traffic = high
Interface errors = increasing
```

### Syslog

Syslog is useful for answering:

> **"What happened?"**

Examples:

```text
Interface went down
User authentication failed
Device rebooted
Configuration event occurred
Warning generated
```

### Together

```text
              Network Monitoring
                     │
          ┌──────────┴──────────┐
          ↓                     ↓
        SNMP                  Syslog
          │                     │
     Device health           Events
     Metrics                 Messages
     Statistics              History
          │                     │
          └──────────┬──────────┘
                     ↓
              Better visibility
```

This is much more powerful than relying on either one alone.

---

# 22. Example: SNMP + Syslog Together

Imagine a switch's CPU suddenly becomes very high.

### SNMP tells you:

```text
CPU utilization = 95%
```

You know **something is wrong**.

But you still need to know what caused it.

Syslog may show an event around the same timestamp.

```text
SNMP:
10:05 → CPU = 95%

Syslog:
10:05 → Device generated a significant event
```

Now you have two different types of evidence:

**SNMP → metric**

**Syslog → event**

Combining both provides much better operational visibility.

---

# 23. Castle Rysen Example

The Castle Rysen RFP specifically requires **network monitoring** and calls for the implementation of **SNMP and syslog** as part of its network services and monitoring requirements. 

Imagine a district shop with:

```text
                District Shop
                     │
       ┌─────────────┼─────────────┐
       ↓             ↓             ↓
     Router        Switch         WAP
       │             │             │
       └─────────────┼─────────────┘
                     │
                     ↓
              Fallout Shelter
                     │
                     ↓
               Syslog Server
```

Instead of investigating each device individually, the operations team can collect events centrally.

For a business with many shops, this becomes essential.

---

# 24. Why Centralized Logging Is Important in a Large Network

Consider 100 devices.

Without centralized logging:

```text
Device 1 → SSH → show logging
Device 2 → SSH → show logging
Device 3 → SSH → show logging
...
Device 100 → SSH → show logging
```

This is inefficient.

With centralized logging:

```text
Device 1 ──┐
Device 2 ──┤
Device 3 ──┤
Device 4 ──┤
    ...     ├──→ Syslog Server
Device 100 ─┘
```

One location contains the events.

This makes:

* troubleshooting easier
* historical analysis possible
* event correlation easier
* security investigations easier
* large-scale operations more manageable

---

# 25. Syslog Configuration Concept

At the simplest level:

### Step 1 — Have a Syslog server

```text
Syslog Server
IP: 192.168.10.100
```

### Step 2 — Tell the Cisco device where it is

```text
Router(config)# logging 192.168.10.100
```

### Step 3 — Generate an event

For example:

```text
interface GigabitEthernet0/1
 no shutdown
```

### Step 4 — Check the Cisco device

```text
show logging
```

### Step 5 — Check the Syslog server

Confirm that the message arrived.

The complete flow:

```text
Cisco Device
     │
     │ Event occurs
     ↓
Syslog message generated
     │
     │ UDP 514
     ↓
Syslog Server
     │
     ↓
Message stored
     │
     ↓
Administrator investigates
```

---

# 26. Syslog in Network Operations

A mature network should not depend entirely on an administrator noticing problems manually.

Instead:

```text
Device
  ↓
Detect event
  ↓
Generate Syslog message
  ↓
Central Syslog server
  ↓
Store event
  ↓
Search / correlate / investigate
```

This changes network administration from:

> "Something broke. Let me figure out what happened."

to:

> "I have a historical record of what happened and when."

That distinction is extremely important in enterprise networking.

---

# 27. Real-World Example

Suppose customers at a coffee shop report:

> "The Wi-Fi stopped working around 7:03 AM."

The administrator checks centralized Syslog.

They discover:

```text
07:02:58  Switch → interface event
07:03:01  WAP → connectivity event
07:03:04  Router → routing event
07:03:07  Firewall → connection event
```

Now the administrator has a **timeline**.

Instead of randomly troubleshooting:

```text
Wi-Fi?
Router?
Switch?
Firewall?
Internet?
```

they can investigate the events in chronological order.

That is the operational power of centralized logging.

---

# 28. Important Terms to Know

| Term                    | Meaning                                                                 |
| ----------------------- | ----------------------------------------------------------------------- |
| **Syslog**              | Standard mechanism for reporting/logging device events                  |
| **Syslog message**      | Individual event/log generated by a device                              |
| **Facility**            | Category/source associated with the message                             |
| **Severity**            | Indicates seriousness of the message                                    |
| **Syslog server**       | Central system receiving and storing logs                               |
| **Local logging**       | Logs retained on the device                                             |
| **Off-box logging**     | Logs sent to another system                                             |
| **Logging buffer**      | Memory area used to hold local log messages                             |
| **UDP 514**             | Default Syslog transport/port referenced in the lesson                  |
| **Centralized logging** | Collecting logs from many devices in one location                       |
| **Event correlation**   | Connecting events from multiple devices/times to understand an incident |

---

# 29. CCNA Memorization Section

### Syslog severity levels

Memorize this sequence:

```text
0 Emergency
1 Alert
2 Critical
3 Error
4 Warning
5 Notice
6 Informational
7 Debugging
```

### Severity rule

```text
Lower number = higher severity
Higher number = lower severity
```

### Syslog default port

```text
UDP 514
```

### Basic Cisco command

```text
logging <Syslog-server-IP>
```

Example:

```text
logging 192.168.10.100
```

### Verification

```text
show logging
```

### Core concept

```text
SNMP → Metrics / health
Syslog → Events / messages
```

---

# 30. The Most Important Concept

Don't think of Syslog merely as:

> "A place where logs are stored."

Think of it as an **event visibility system**.

A network device continuously experiences events:

```text
                    DEVICE
                      │
       ┌──────────────┼──────────────┐
       ↓              ↓              ↓
   Interface       Login          System
     events        events         events
       │              │              │
       └──────────────┼──────────────┘
                      ↓
                  SYSLOG
                      ↓
              Centralized Server
                      ↓
        ┌─────────────┼─────────────┐
        ↓             ↓             ↓
     Search        Correlate      History
        │             │             │
        └─────────────┼─────────────┘
                      ↓
                Troubleshooting
                 & Security
```

The major reason to use centralized Syslog is **context**.

One device's message may only tell you that something happened. Messages from many devices, combined with timestamps, can tell you **what happened across the network and in what sequence**.

---

# 31. Final Mental Model

Remember this entire lesson with one picture:

```text
                 NETWORK DEVICES
        ┌──────────┬──────────┬──────────┐
        │          │          │          │
      Router     Switch       WAP      Firewall
        │          │          │          │
        └──────────┴──────────┴──────────┘
                         │
                    Syslog messages
                         │
                      UDP 514
                         │
                         ↓
                  ┌───────────────┐
                  │ Syslog Server │
                  └───────────────┘
                         │
             ┌───────────┼───────────┐
             ↓           ↓           ↓
          History     Search      Correlation
             │           │           │
             └───────────┼───────────┘
                         ↓
                  Troubleshooting
                    & Security
```

### If you remember only 5 things:

1. **Syslog records device events and messages.**
2. **There are 8 severity levels: 0–7.**
3. **0 is the most severe; 7 is Debugging.**
4. **Syslog commonly uses UDP 514.**
5. **Centralized/off-box logging preserves history and allows correlation across many devices.**

**SNMP tells you the network's condition; Syslog tells you the network's events.**

