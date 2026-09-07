# Skill 22 — Lesson 02: Setting the Clock with NTP

## 1. Why NTP?

Manually setting the clock works for a single device, but it becomes a maintenance problem as the network grows.

### Problems with manual clocks

* Every device must be configured individually.
* Device clocks can **drift apart over time**.
* Different timestamps make troubleshooting difficult.
* Logs from different devices may appear to show events happening at different times.
* Authentication, certificates, monitoring, and event correlation can depend on accurate time.

### NTP solves this

**NTP — Network Time Protocol** provides a consistent time source to network devices.

```text
                  NTP Server
                      │
             Time synchronization
                      │
          ┌───────────┼───────────┐
          ▼           ▼           ▼
       Router       Switch        AP
          │           │           │
       Correct      Correct      Correct
          time        time        time
```

The important concept is:

> **The goal isn't simply to set the correct time once; the goal is continuous synchronization.**

---

# 2. NTP Server vs NTP Client

NTP operates using a client/server relationship.

### NTP Server

The device that provides time.

It could be:

* Linux server
* Windows server
* Dedicated NTP server
* Public Internet NTP server
* Cisco router configured as an NTP master

### NTP Client

The device that obtains time from an NTP server.

Examples:

* Router
* Switch
* Access point
* Other network devices

```text
NTP Server
    │
    │ provides time
    ▼
NTP Client
    │
    │ synchronizes
    ▼
Local system clock
```

---

# 3. Stratum

**Stratum** indicates how far an NTP device is from the original authoritative time source.

### Lower = better

A lower stratum means the device is closer to the original source of time.

Example:

```text
Atomic / highly accurate reference
              │
              ▼
        Stratum 1
              │
              ▼
        Stratum 2
              │
              ▼
        Stratum 3
              │
              ▼
        Stratum 4
```

If a device obtains time from a **Stratum 1** source, it becomes **Stratum 2**.

If another device obtains time from that Stratum 2 device, it becomes **Stratum 3**.

### Maximum stratum

NTP uses **Stratum 16** to represent an unusable/untrusted synchronization state.

So remember:

| Stratum | Meaning                               |
| ------- | ------------------------------------- |
| 1       | Very close to authoritative reference |
| 2       | One synchronization level away        |
| 3       | Two levels away                       |
| ...     | Increasing distance                   |
| 16      | Not considered synchronized/trusted   |

### CCNA memory trick

**Lower stratum → closer to source → more authoritative.**

---

# 4. Configuring a Cisco Router as an NTP Master

In the lab, the router's clock is first manually configured.

Then:

```cisco
ntp master
```

This makes the Cisco router act as an **NTP time source** for other devices.

### Important

The lesson demonstrates that a Cisco router configured with:

```cisco
ntp master
```

reports itself with a default **stratum of 8**.

That does **not** mean the router is actually connected to an atomic clock.

It is simply acting as the authoritative time source for this local NTP environment.

---

# 5. Verifying NTP

One important command is:

```cisco
show ntp status
```

This allows you to determine whether the device is actually synchronized.

For an NTP master, seeing that the router is synchronized to itself makes sense because it is the authority in that particular NTP environment.

### Important distinction

Don't confuse:

```text
Configured as NTP server
```

with:

```text
Successfully synchronized to an NTP source
```

**Configuration ≠ synchronization.**

Always verify.

---

# 6. Enterprise NTP Design

A small lab might look like:

```text
Router
(NTP Master)
    │
    ├──────── Switch 1
    ├──────── Switch 2
    └──────── Router
```

But in a larger production network, you generally don't want every device directly relying on an external Internet NTP server.

A better architecture is:

```text
             Public / Authoritative
                  NTP Sources
                       │
              ┌────────┴────────┐
              ▼                 ▼
          Internal NTP      Internal NTP
            Server 1           Server 2
              │                 │
              └────────┬────────┘
                       │
              Internal Network
                       │
        ┌──────────────┼──────────────┐
        ▼              ▼              ▼
      Router         Switch           AP
```

### Benefits

* Centralized time management
* Better control
* Reduced dependency on external sources
* Easier troubleshooting
* Internal devices can synchronize consistently
* Redundancy can be implemented with multiple time sources

For a real enterprise network, **redundant internal time sources** are preferable to creating a single point of failure.

---

# 7. Why Loopback Interfaces Matter for NTP

A router may have many interfaces:

```text
Gi0/0 → LAN
Gi0/1 → WAN
Gi0/2 → another network
VLAN interfaces → various networks
```

Which IP should other devices use as the NTP server?

Using a physical interface can create a problem if that interface changes or goes down.

### Loopback

A **loopback interface** is a logical/virtual interface that provides a stable IP address.

Example:

```cisco
interface Loopback0
 ip address 10.1.0.1 255.255.255.255
```

This creates:

```text
10.1.0.1/32
```

You can use that stable address as the router's NTP identity.

```text
             Router
        ┌────────────────┐
        │ Loopback0      │
        │ 10.1.0.1/32    │
        └────────────────┘
              ▲
              │
        NTP clients
```

The loopback can then be advertised through the routing system so other devices can reach it.

### Why this is useful

Physical interface:

```text
Gi0/0 = 192.168.10.1
```

could change or become unavailable.

Loopback:

```text
Lo0 = 10.1.0.1/32
```

provides a stable logical identity.

**Key idea:**

> Use a stable loopback address when you need a consistent identity for network services.

The lesson specifically notes that Packet Tracer did not support the full `ntp source` behavior desired in this scenario, but the **design concept remains important**.

---

# 8. Configuring an NTP Client

Once the client has IP connectivity to the NTP server, configure the client with:

```cisco
ntp server <NTP-server-IP>
```

Example:

```cisco
Switch(config)# ntp server 10.1.0.1
```

This tells the switch:

> "Use 10.1.0.1 as my NTP server."

The basic process is therefore:

```text
1. NTP server has correct time
           ↓
2. NTP server configured
           ↓
3. Client has IP connectivity
           ↓
4. Client configured with ntp server
           ↓
5. Wait for synchronization
           ↓
6. Verify NTP status
```

---

# 9. NTP Requires Network Connectivity

This is one of the most important practical lessons.

NTP is an **IP-based service**.

If the client cannot reach the NTP server, NTP cannot work.

Before troubleshooting NTP, verify basic connectivity.

For example:

```cisco
Switch# ping 10.1.0.1
```

If the ping fails:

```text
NTP problem?
      │
      ▼
Check basic connectivity first
      │
      ├── VLAN
      ├── IP address
      ├── Default gateway
      ├── Routing
      └── ACL/security
```

### Troubleshooting principle

> **Infrastructure services are not magic.**

If the underlying network path is broken, the service won't work.

---

# 10. NTP Synchronization Takes Time

After configuring:

```cisco
ntp server <IP>
```

don't immediately assume something is broken if:

```cisco
show ntp status
```

reports that the device is not synchronized.

NTP may take some time to establish synchronization.

The lesson specifically emphasizes that you may need to **wait a minute or two** before synchronization occurs.

### Practical workflow

```text
Configure NTP
     ↓
Check connectivity
     ↓
Wait
     ↓
show ntp status
     ↓
Synchronized?
   ↙       ↘
 YES        NO
  │          │
Done     Troubleshoot
```

---

# 11. Time Zones vs NTP

This is a common source of confusion.

NTP synchronization and local time-zone display are related but **not the same thing**.

Think of it as:

```text
NTP
 │
 └── Provides synchronized reference time
                │
                ▼
        Device time settings
                │
                ▼
          Local time zone
                │
                ▼
        Displayed local time
```

Therefore, a device can be **successfully synchronized** but still display a time that looks incorrect because its timezone is configured incorrectly.

### Troubleshooting sequence

If the displayed time looks wrong:

**Don't immediately conclude that NTP failed.**

First check:

```cisco
show ntp status
```

Then check the device's timezone configuration.

### Key distinction

```text
NTP synchronization
        ≠
Timezone configuration
```

---

# 12. `show clock` vs `show clock detail`

You have already used:

```cisco
show clock
```

This displays the device's current clock.

For more information:

```cisco
show clock detail
```

The detailed output can help identify where the device's time is coming from.

Once NTP synchronization is working, the source should reflect **NTP rather than user configuration**.

---

# 13. Important Commands

| Command             | Purpose                                      |
| ------------------- | -------------------------------------------- |
| `clock set ...`     | Manually configure the device clock          |
| `show clock`        | Display current time                         |
| `show clock detail` | Display detailed clock information/source    |
| `ntp master`        | Configure a Cisco device as an NTP master    |
| `ntp server <IP>`   | Configure an NTP client to use an NTP server |
| `show ntp status`   | Verify NTP synchronization/status            |
| `ping <IP>`         | Verify IP connectivity to the NTP server     |

---

# 14. Troubleshooting NTP

If NTP isn't synchronizing, don't randomly change NTP commands.

Work through the layers.

### Step 1 — Check the clock

```cisco
show clock
```

### Step 2 — Check IP connectivity

```cisco
ping <NTP-server-IP>
```

### Step 3 — Verify NTP configuration

```cisco
show running-config
```

Look for the NTP configuration.

### Step 4 — Check NTP status

```cisco
show ntp status
```

### Step 5 — Check timezone

If NTP is synchronized but the displayed time is wrong, inspect the timezone configuration.

### Step 6 — Give NTP time

Don't forget that synchronization may not be immediate.

---

# 15. Real-World Enterprise Example

Imagine Castle Rysen has:

```text
                  Internet
                     │
              External NTP source
                     │
              ┌──────┴──────┐
              │             │
          Internal NTP   Internal NTP
             Server 1       Server 2
              │             │
              └──────┬──────┘
                     │
             Castle Rysen Core
                     │
        ┌────────────┼────────────┐
        ▼            ▼            ▼
     Router        Switch       Router
        │            │            │
      NTP          NTP          NTP
```

Now when an event occurs:

```text
09:15:02 Router
09:15:02 Switch
09:15:03 Firewall
09:15:02 Server
```

you can correlate events much more reliably.

Without synchronized time:

```text
Router:   09:15
Switch:   09:08
Firewall: 09:21
Server:   09:13
```

Troubleshooting becomes significantly harder.

This connects directly to Castle Rysen's RFP requirement for **NTP as an IP service**. 

---

# 16. The Big Picture

The lesson is really teaching **three levels of thinking**:

### Level 1 — Manual configuration

```cisco
clock set ...
```

Useful for initially setting a device's time.

### Level 2 — NTP synchronization

```cisco
ntp master
ntp server <IP>
```

Allows devices to continuously synchronize.

### Level 3 — Enterprise time architecture

```text
Authoritative time
       ↓
Internal NTP sources
       ↓
Network infrastructure
       ↓
Servers / applications / endpoints
```

This gives you **consistent, maintainable, and reliable time across the environment**.

---

# 🧠 CCNA Takeaways

1. **NTP = Network Time Protocol.**
2. NTP keeps network devices synchronized over time.
3. An **NTP server** provides time; an **NTP client** receives it.
4. **Lower stratum = closer to the authoritative time source.**
5. **Stratum 16** represents an unusable/untrusted synchronization state.
6. `ntp master` makes a Cisco device act as an NTP time source.
7. `ntp server <IP>` tells a device which NTP server to use.
8. NTP requires **working IP connectivity**.
9. NTP synchronization may take some time.
10. A **loopback interface** can provide a stable address for network services.
11. **NTP synchronization and timezone configuration are different things.**
12. `show ntp status` is essential for verifying synchronization.
13. `show clock detail` can help identify the source of the device's time.
14. In enterprise networks, centralized/internal NTP sources are preferable to having every device independently depend on an external source.
15. Accurate time is essential for **log correlation, troubleshooting, authentication, certificates, monitoring, and security investigations**.

### ⭐ One sentence to remember

> **NTP doesn't just set the clock—it keeps the network's clocks synchronized to a common time source.**
