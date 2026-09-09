![alt text](<PRTG tool.png>)
## Understanding the PRTG Screenshot

This screenshot shows exactly what the lesson is describing: **SNMP being used by a Network Management System (NMS) to monitor a real network device.**

The software shown is **PRTG Network Monitor**. The screenshot gives you a practical picture of how SNMP becomes graphs, statistics, status indicators, and alerts.

### 1. PRTG is the NMS

On the left, you can see a hierarchy of monitored infrastructure:

```text
KITS
└── Root
    ├── Hosted Probe
    │   └── Probe Device
    ├── VMware8 Probe
    │   ├── Wireless Access Points
    │   ├── Switches
    │   └── Router / Firewall
    └── ...
```

This is the **NMS side** of SNMP.

PRTG is monitoring many devices and their individual sensors.

---

## 2. The Switch Being Monitored

The selected device is:

**UniFi Switch - 24 250W-POE (MDF)**

Under that switch, PRTG has multiple sensors:

```text
UniFi Switch
│
├── Ping
├── Uptime
├── Port 1 - Dream Machine (Port 1)
├── Port 1 - Detailed Packet Statistics
├── Port 2 - USW-48-POE+ TV
├── Port 2 - Detailed Packet Statistics
├── Port 3 - Proxmox Allen PC
├── Port 4 - Synology DS1618 1GbE
├── Port 5 - US-XG 10Gbps Switch
└── ...
```

This is a very important visualization of the **"OID = individual data point"** concept from your lesson.

PRTG isn't just monitoring "the switch."

It's monitoring **many different aspects of the switch**.

---

# 3. The Selected Sensor

The selected sensor is:

**Sensor Port 1 - Dream Machine (Port 1)**

You can see:

```text
Status:       Up
Last Value:   32,519 kbit/s
Interval:     60 s
Sensor Type:  SNMP Traffic 64bit
```

This is a direct practical example of SNMP monitoring.

### What does `SNMP Traffic 64bit` mean?

PRTG is obtaining traffic information from the device using **SNMP**.

The sensor is measuring traffic associated with that interface.

So conceptually:

```text
PRTG/NMS
    |
    | SNMP
    |
    ↓
UniFi Switch
    |
    └── Port 1
          |
          └── Traffic counters
```

PRTG periodically retrieves those values and turns them into the information displayed on screen.

---

# 4. The 60-Second Interval

Look at:

**Interval: 60 s**

That means PRTG is checking/updating this sensor at a configured interval of approximately **60 seconds**.

This connects directly to the lesson's explanation of **polling**.

```text
12:00:00 → poll
12:01:00 → poll
12:02:00 → poll
12:03:00 → poll
       ...
```

Each measurement contributes to the historical dataset.

Eventually, you can produce:

```text
Current value
      +
Historical values
      ↓
Graphs
      ↓
Trends
      ↓
Alerts
      ↓
Troubleshooting
```

---

# 5. What Does `32,519 kbit/s` Mean?

The selected sensor currently shows:

**32,519 kbit/s**

That's approximately:

**32.5 Mbit/s**

of total traffic represented by the sensor at that moment.

Notice that PRTG also separates traffic into:

* **Traffic In**
* **Traffic Out**
* **Traffic Total**

The screenshot shows approximately:

```text
Traffic In     → 1,395 kbit/s
Traffic Out    → 31,124 kbit/s
Traffic Total  → 32,519 kbit/s
```

So most of the traffic at that instant is **outbound**.

This is exactly the kind of measurable information that SNMP can provide.

---

# 6. The Graphs Are Historical Data

On the right side, you can see graphs covering different time periods.

For example:

* Recent traffic
* Longer historical traffic
* Peaks and changes over time

This demonstrates an important point from the lesson:

> SNMP isn't useful only for knowing what's happening **right now**.

Repeated polling allows the NMS to build a history.

For example:

```text
SNMP polling
     ↓
60 sec
     ↓
60 sec
     ↓
60 sec
     ↓
60 sec
     ↓
Historical database
     ↓
Graph
```

Now you can identify patterns.

---

# 7. Minimum, Maximum, and Current Values

The screenshot also displays statistics such as:

```text
Last Value
Minimum
Maximum
```

For example, the traffic table shows values for:

```text
Traffic In
Traffic Out
Traffic Total
```

with minimum and maximum values.

This is useful because a current value alone might not tell the whole story.

Suppose:

```text
Current traffic = 5 Mbps
Maximum traffic = 900 Mbps
```

The current state looks harmless.

But the historical maximum tells you that the interface has experienced much heavier utilization.

That's why **historical monitoring matters**.

---

# 8. Uptime Sensor

Notice another sensor:

**Uptime**

It currently shows:

```text
Up
55 d
```

This is another example of the kind of information that can be monitored.

Remember from your lesson:

> **Uptime is a measurable data point.**

Therefore, conceptually:

```text
Uptime
   ↓
OID
   ↓
SNMP
   ↓
NMS
   ↓
PRTG
```

---

# 9. Status Indicators

On the left, PRTG uses different status indicators.

You can see sensors marked as:

* **Up**
* **Warning**
* **Unusual**
* Paused/other states

This is where raw SNMP data becomes something useful for an administrator.

Instead of manually examining every metric:

```text
141 devices
hundreds of sensors
thousands of values
```

the monitoring system can summarize the situation:

```text
✓ Up
⚠ Warning
! Unusual
```

That is one of the major benefits of an NMS.

---

# 10. This Is the Lesson's "Visibility"

Compare the two situations.

### Without monitoring

```text
User:
"The network is slow."

Engineer:
"Which device?"

User:
"I don't know."

Engineer:
"Which interface?"

User:
"I don't know."

Engineer:
"Okay..."
```

### With PRTG/SNMP

```text
Alert / complaint
      ↓
Open PRTG
      ↓
Check device
      ↓
Check interface
      ↓
Check traffic graph
      ↓
Check historical data
      ↓
Identify abnormal behavior
```

Now you're working with **evidence**.

---

# 11. Where OIDs Fit Into This Screenshot

The screenshot doesn't expose the actual numerical OID strings, but the lesson's concept applies directly.

Think of the sensor:

**Port 1 - Dream Machine (Port 1)**

as requesting specific management information from the switch.

Conceptually:

```text
PRTG
 |
 | "Give me the value for this interface's
 |  traffic-related management object."
 |
 ↓
Switch
 |
 | returns counter/value
 ↓
PRTG
 |
 ↓
32,519 kbit/s
```

The underlying management objects are represented through SNMP OIDs.

You don't normally have to manually memorize those OIDs because PRTG handles the repetitive work.

---

# 12. Why the Screenshot Shows Multiple Sensors

This is another important concept.

A single physical switch can expose **many different management data points**.

Think:

```text
                    SWITCH
                       |
        +--------------+--------------+
        |              |              |
      Uptime         Port 1         Port 2
        |              |              |
       OID            OIDs           OIDs
        |              |              |
        +--------------+--------------+
                       |
                      SNMP
                       |
                      PRTG
```

That's why PRTG can have multiple sensors under the same switch.

Each sensor focuses on a particular aspect of the device.

---

# 13. The Right Side Is Basically Your Operations Dashboard

The selected sensor's overview gives you:

```text
Status
Last Scan
Last Up
Last Down
Uptime
Downtime
Sensor Type
Interval
ID
```

Then you have tabs such as:

* **Overview**
* **Graphs**
* **Historic Data**
* **Log**
* **Settings**
* **Notification Triggers**
* **Comments**
* **History**

This nicely connects the two lessons you've studied:

### SNMP

Provides:

**metrics + status + measurements**

### Syslog/logging

Provides:

**events + messages + context**

And an NMS brings these kinds of information together into an operational interface.

---

# 14. One Important Correction to Keep in Mind

Don't think:

> **"SNMP = PRTG."**

That's incorrect.

Think:

```text
SNMP
   ↓
Protocol used for network management/monitoring

PRTG
   ↓
Network monitoring/NMS software

Switch
   ↓
Device being monitored
```

So:

```text
             PRTG
              |
             NMS
              |
             SNMP
              |
              ↓
       Network Devices
```

PRTG is **using SNMP** to obtain information from supported devices.

---

# 15. How This Connects to Your SNMP Lesson

Your lesson introduced:

| Lesson concept      | Screenshot                                  |
| ------------------- | ------------------------------------------- |
| **SNMP**            | Sensor type shows `SNMP Traffic 64bit`      |
| **NMS**             | PRTG                                        |
| **Device**          | UniFi switch                                |
| **OID**             | Underlying management objects being queried |
| **Polling**         | 60-second interval                          |
| **Metrics**         | Traffic values                              |
| **Availability**    | Up/Down status                              |
| **Historical data** | Graphs                                      |
| **Trends**          | Traffic history                             |
| **Alerts**          | Warning/unusual states                      |

So this screenshot is essentially the theory you've just learned **turned into an actual monitoring system**.

### The complete flow

```text
                 NETWORK DEVICE
                       |
             Management Information
                       |
                      OIDs
                       |
                      SNMP
                       |
                       ↓
                     PRTG
                      NMS
                       |
          +------------+------------+
          |            |            |
       Current      Historical    Alerts
        values        graphs
          |            |            |
          +------------+------------+
                       |
                Network Engineer
                       |
                       ↓
              Evidence-based
               troubleshooting
```

**The key takeaway:** when you look at this screenshot, don't just see a pretty monitoring dashboard. See **an NMS repeatedly querying network-device management objects through SNMP and converting those measurements into operational visibility.**
