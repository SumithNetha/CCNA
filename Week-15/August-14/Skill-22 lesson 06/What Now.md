# Week 15 — August 14

# Skill 22 Lesson 06 — What Now?

This lesson is the **wrap-up of the entire IP Services section**. The main purpose isn't to introduce another command. It is to change how you think about deploying network devices.

The central idea is:

> **NTP, DHCP, DNS, and SSH should become part of your normal device-deployment baseline rather than technologies you remember only when a problem occurs.**

The lesson explicitly calls these the **"core four"**:

1. **NTP**
2. **DHCP**
3. **DNS**
4. **SSH**



---

# 1. The real lesson: stop treating services as separate topics

Earlier, you learned these technologies individually:

```text
NTP
 ↓
Time synchronization

DHCP
 ↓
Automatic IP configuration

DNS
 ↓
Name resolution

SSH
 ↓
Secure remote management
```

The lesson wants you to stop thinking:

> "Today I learned NTP."

> "Yesterday I learned DHCP."

Instead, think:

> **"Whenever I deploy a network device or network, what core services does it need to operate properly?"**

That's the shift.

A Cisco switch or router isn't just:

```text
Hostname
IP address
Password
```

A production device is part of a larger operational system.

For example:

```text
                  Network
                     |
       +-------------+-------------+
       |             |             |
      NTP           DNS           SSH
       |             |             |
   Correct time   Name lookup   Management
                     |
                    DHCP
                     |
              IP configuration
```

These services make the network **manageable, predictable, and supportable**.

---

# 2. The "core four"

The lesson gives you four services that should immediately come to mind whenever you're deploying equipment.

| Service  | What it does                           | Why you care                                |
| -------- | -------------------------------------- | ------------------------------------------- |
| **NTP**  | Synchronizes time                      | Accurate timestamps and troubleshooting     |
| **DHCP** | Automatically assigns IP configuration | Reduces manual configuration                |
| **DNS**  | Maps names to IP addresses             | Makes services and devices easier to access |
| **SSH**  | Provides secure remote management      | Allows secure administration                |



But there is an important qualification:

### Not every device necessarily provides DHCP.

The lesson says to include DHCP **where the design requires the device to serve or depend on automatic addressing**.

So don't blindly configure every router and switch as a DHCP server.

Instead:

```text
NTP → baseline service
DNS → baseline service
SSH → baseline management service
DHCP → deploy when the network design requires it
```

---

# 3. NTP — Make time normal

## What NTP does

**NTP = Network Time Protocol**

Its job is to keep network devices synchronized with a reliable time source.

For example:

```text
NTP Server
    |
    | time synchronization
    |
    +-------- Router
    |
    +-------- Switch
    |
    +-------- Firewall
```

Without synchronization, devices could have different times:

```text
Router:   10:15
Switch:   10:21
Firewall: 09:58
Server:   10:17
```

That creates operational problems.

---

# 4. Why time is important for troubleshooting

Imagine that a network failure occurs.

Your logs show:

```text
Router:
10:15:01 interface down

Switch:
10:07:43 link failure

Firewall:
09:58:12 connection denied
```

You now have difficulty establishing the correct sequence of events.

With synchronized clocks:

```text
09:58:12  Firewall denies connection
10:07:43  Switch detects link failure
10:15:01  Router interface goes down
```

You can build an accurate timeline.

This becomes particularly important when you later learn **Syslog, SNMP, security monitoring, and troubleshooting**.

---

# 5. DHCP — Don't manually configure everything

**DHCP = Dynamic Host Configuration Protocol**

DHCP provides network configuration automatically.

Instead of manually configuring every client:

```text
PC1 → IP manually
PC2 → IP manually
PC3 → IP manually
PC4 → IP manually
...
```

you can use:

```text
DHCP Server
     |
     +---- PC1
     +---- PC2
     +---- PC3
     +---- PC4
```

The DHCP service can provide things such as:

```text
IP address
Subnet mask
Default gateway
DNS server
```

---

# 6. Why DHCP belongs in your deployment thinking

Imagine a new Castle Rysen coffee shop with 15 employees and many customer devices.

If you manually configure every endpoint:

```text
Device 1 → configure
Device 2 → configure
Device 3 → configure
...
Device 100 → configure
```

that's inefficient and introduces configuration errors.

Instead:

```text
Client
   |
   | DHCP request
   ↓
DHCP service
   |
   ↓
IP configuration
```

The client can automatically receive the appropriate network configuration.

---

# 7. DNS — Stop thinking only in IP addresses

**DNS = Domain Name System**

DNS converts names into IP addresses.

Instead of remembering:

```text
10.0.18.10
```

you can work with something like:

```text
plex-server
```

or:

```text
server.example.com
```

Conceptually:

```text
Client
  |
  | "What is the IP of plex-server?"
  ↓
DNS
  |
  | "10.0.18.10"
  ↓
Client
```

---

# 8. Why DNS is part of the baseline

Without DNS, administrators and applications increasingly have to deal directly with IP addresses.

With DNS:

```text
Name
 ↓
DNS
 ↓
IP
 ↓
Communication
```

That makes the environment easier to operate.

The lesson's point isn't simply:

> "Know what DNS is."

It is:

> **When deploying a device, make sure name resolution is considered from the beginning.**

---

# 9. SSH — The service that was "snuck in"

The lesson makes a specific point about SSH.

Initially, SSH might seem like something that belongs later in the security section.

But the lesson argues that **SSH should be treated as a core service** because secure remote administration is fundamental to operating network devices.



---

# 10. What SSH gives you

**SSH = Secure Shell**

It allows you to remotely manage network devices.

Instead of:

```text
Administrator
     |
     | console cable
     ↓
   Router
```

you can have:

```text
Administrator
     |
     | SSH
     ↓
   Router
```

The major point is that SSH provides a secure remote-management mechanism.

---

# 11. Why SSH should be configured early

Imagine you've installed a new switch in a remote location.

If you only configure basic connectivity and then leave:

```text
Switch
 ├── hostname
 ├── IP address
 └── VLANs
```

you may later realize:

> "How am I going to manage this thing remotely?"

You then have to physically access it or perform additional configuration under pressure.

Instead, the deployment standard should include secure remote management from the beginning:

```text
New Cisco Device
       |
       +── Basic configuration
       |
       +── NTP
       |
       +── DNS
       |
       +── SSH
       |
       +── Other required services
```

---

# 12. Why SSH is not just a "security feature"

This is one of the most important conceptual points.

Security and operations overlap.

SSH is certainly a security technology because it protects remote management.

But operationally, it is also essential because administrators need a reliable way to manage devices.

So:

```text
SSH
├── Security
└── Operations
```

That is why the lesson puts SSH alongside NTP, DHCP, and DNS.

---

# 13. The baseline configuration mindset

The lesson repeatedly emphasizes the concept of a **template**.

Instead of configuring every new device from memory:

```text
New Router #1 → "What do I need again?"
New Router #2 → "Did I configure NTP?"
New Switch #1 → "Did I configure SSH?"
New Switch #2 → "Did I configure DNS?"
```

create a standard baseline.

Conceptually:

```text
                    BASELINE
                       |
        +--------------+--------------+
        |              |              |
       NTP            DNS            SSH
        |
     Time sync

                    DHCP
              where required
```

Then every device deployment starts from that baseline.

---

# 14. Standardization

This is actually a major **enterprise networking principle**.

Imagine an organization has:

```text
100 switches
30 routers
20 firewalls
50 access points
```

If every engineer configures devices differently, the environment becomes difficult to maintain.

For example:

```text
Switch A → NTP configured
Switch B → no NTP
Switch C → different NTP server
Switch D → SSH
Switch E → Telnet
Switch F → different DNS
```

That's inconsistent.

Instead:

```text
                    STANDARD
                       |
        +--------------+--------------+
        |              |              |
      Switch         Router         Firewall
        |              |              |
       NTP            NTP            NTP
       DNS            DNS            DNS
       SSH            SSH            SSH
```

Now engineers know what to expect.

---

# 15. Standardization does NOT mean identical configuration

This distinction is important.

A **standard** doesn't mean every device gets exactly the same commands.

It means every device follows the same **deployment philosophy**.

For example:

### Router

May need:

```text
NTP
DNS
SSH
DHCP
NAT
Routing
```

### Layer 2 switch

May need:

```text
NTP
DNS
SSH
VLANs
STP
Layer 2 security
```

### DHCP server

May need:

```text
DNS
NTP
DHCP scopes
```

So the standard is:

> **Every device gets the appropriate baseline services for its role.**

---

# 16. The Castle Rysen example

The lesson uses **NetworkChuck Coffee** as the practical scenario.

Suppose a new coffee shop needs a Cisco switch.

You rack it and configure only enough to pass traffic.

Technically:

```text
PC → Switch → Router → Internet
```

might work.

But operationally, you could still have problems.

For example:

```text
No NTP
    ↓
Incorrect timestamps

No DNS
    ↓
Poor name resolution

No DHCP where required
    ↓
Manual client configuration

No SSH
    ↓
Poor remote management
```

So:

> **"It passes traffic" does not automatically mean "it's production ready."**

That's a very important enterprise-networking mindset.

---

# 17. Production network vs working network

Consider these two networks.

### Network A

```text
Ping works.
Internet works.
```

Technically functional.

### Network B

```text
Ping works
+
NTP configured
+
DNS configured
+
DHCP correctly deployed
+
SSH enabled
+
Standard configuration
+
Documented
+
Secure management
```

Network B is much more operationally mature.

The lesson is pushing you toward Network B.

---

# 18. The baseline checklist

When a new Cisco device arrives, your thought process should become:

### 1. Identity

```text
Hostname
```

### 2. Management connectivity

```text
Management IP
```

### 3. Time

```text
NTP
```

### 4. Name resolution

```text
DNS
```

### 5. Remote management

```text
SSH
```

### 6. Automatic addressing

```text
DHCP
```

**if required by the network design.**

Then continue with the device-specific configuration.

---

# 19. Core services aren't necessarily "extras"

This is the biggest message of the lesson.

A common beginner mindset is:

```text
Routing = important
Switching = important

NTP = small thing
DNS = small thing
DHCP = small thing
SSH = security thing
```

The lesson wants you to reject that thinking.

Instead:

```text
              Production Network
                     |
        +------------+------------+
        |            |            |
       NTP          DNS          SSH
        |            |            |
     Time         Naming       Management
                     |
                   DHCP
                     |
                Addressing
```

These are **foundational operational services**.

---

# 20. Why "common" doesn't mean "unimportant"

The lesson makes an important observation:

> **Common usually means foundational.**

The technologies you use constantly are often the ones that are easiest to overlook.

For example:

* DHCP works so automatically that people forget how important it is.
* DNS works in the background and is often ignored until it fails.
* NTP seems simple until timestamps become inconsistent.
* SSH seems routine until you need to manage a remote device.

These are precisely the kinds of things that can generate major operational problems when missing.

---

# 21. Build the habit, not just the lab

This is probably the most important practical instruction.

You don't want:

```text
Learn NTP
 ↓
Pass quiz
 ↓
Forget NTP
```

You want:

```text
Learn NTP
 ↓
Configure NTP
 ↓
Use NTP repeatedly
 ↓
Recognize when it's missing
 ↓
Make it part of deployment
```

The same applies to:

```text
DHCP
DNS
SSH
```

That's how knowledge becomes **muscle memory**.

---

# 22. Why experienced network engineers seem fast

The lesson explains that experienced engineers don't necessarily memorize every feature.

Instead, they know the things that **consistently matter**.

When they enter a new environment, they have a mental checklist.

For example:

```text
New device
    ↓
What's the role?
    ↓
What's the management IP?
    ↓
What's the NTP source?
    ↓
What's the DNS configuration?
    ↓
How will I securely manage it?
    ↓
Does it need DHCP?
    ↓
What device-specific configuration is required?
```

That lets them get productive quickly without reinventing their process every time.

---

# 23. The recommended real-world template

The lesson's explicit recommendation is to create a **base configuration template for each Cisco device type** and include:

```text
NTP
DNS
SSH
```

by default.

Then add:

```text
DHCP
```

where the design requires the device to provide addresses. 

This is much better than relying on memory.

---

# 24. Why templates reduce errors

Suppose you manually configure 50 devices.

There's a probability you'll forget something.

For example:

```text
Device 1 → NTP ✓
Device 2 → NTP ✓
Device 3 → NTP ✗
Device 4 → NTP ✓
Device 5 → NTP ✗
```

Now you have an inconsistent environment.

With a standard template:

```text
Template
   |
   +── NTP
   +── DNS
   +── SSH
   +── baseline security
   +── standard management
```

you reduce the chance of forgetting the baseline.

This concept becomes especially important later when you study **automation and programmability**.

---

# 25. Connection to automation

Notice where this lesson is taking you.

Today:

```text
Human
 ↓
Template
 ↓
Cisco device
```

Later:

```text
Automation
 ↓
Template
 ↓
Many Cisco devices
```

For example:

```text
                   Standard Configuration
                           |
              +------------+------------+
              |            |            |
           Router 1      Router 2     Router 3
              |            |            |
             NTP          NTP          NTP
             DNS          DNS          DNS
             SSH          SSH          SSH
```

This is one reason standardization matters before automation.

**You cannot effectively automate a deployment process that you haven't standardized.**

---

# 26. Connection to troubleshooting

Your baseline also makes troubleshooting easier.

Suppose you're troubleshooting a device and discover:

```text
NTP → configured
DNS → configured
SSH → configured
DHCP → correctly configured
```

You immediately know the standard baseline exists.

If one device is missing NTP:

```text
Expected: NTP
Actual:   Missing
```

you have a concrete configuration deviation to investigate.

This is much better than asking:

> "What did the previous engineer configure on this thing?"

---

# 27. Connection to security

SSH also establishes an important bridge between **IP services** and **network security**.

A production device needs:

```text
Connectivity
     +
Management
     +
Security
```

SSH contributes to secure management.

Later, this combines with:

```text
ACLs
AAA
Port Security
DHCP Snooping
DAI
VPN
```

So the lessons aren't isolated chapters.

They're building toward a complete enterprise network.

---

# 28. The four services as a mental checklist

When you see a new network device, immediately think:

```text
                NEW DEVICE
                     |
          +----------+----------+
          |          |          |
         NTP        DNS        SSH
          |          |          |
        Time       Names     Management
                     |
                    DHCP
                     |
                Addressing
```

Then ask:

### NTP

**Does this device have a reliable time source?**

### DNS

**Can it resolve names appropriately?**

### SSH

**Can I securely manage it remotely?**

### DHCP

**Does this design require the device to provide or obtain addressing dynamically?**

That is the mental model the lesson wants you to develop.

---

# 29. What "production ready" should mean to you

After this lesson, don't define a configured Cisco device as:

> "I gave it an IP and it responds to ping."

Instead:

> **"It has been configured according to the organization's baseline and is ready to be operated, monitored, managed, and troubleshot."**

That is a much more professional definition.

---

# 30. Complete Skill 22 mental model

You can now combine the entire August 10–14 sequence:

```text
                   IP SERVICES
                        |
       +----------------+----------------+
       |                |                |
      NTP              DNS              DHCP
       |                |                |
  Time sync        Name resolution   IP configuration
       |                |                |
       +----------------+----------------+
                        |
                     NETWORK
                        |
                       SSH
                        |
               Secure management
```

And in the actual network:

```text
                         INTERNET
                            |
                           NAT
                            |
                         ROUTER
                       /        \
                      /          \
                ADMIN VLAN     GUEST VLAN
                    |              |
                 SSH/Admin      DHCP Clients
                    |              |
              Network devices      DNS
                                   |
                                Internet
                                   |
                                  Plex
```

This ties together much of what you've learned throughout the course.

---

# 31. What you should remember for CCNA

### NTP

**Purpose:** Time synchronization.

Think:

> **"Are all my devices using the correct time?"**

---

### DHCP

**Purpose:** Automatic IP configuration.

Think:

> **"How are my clients getting their addressing?"**

---

### DNS

**Purpose:** Name resolution.

Think:

> **"How does a device turn a name into an IP address?"**

---

### SSH

**Purpose:** Secure remote management.

Think:

> **"How will I securely administer this device?"**

---

# 32. The ultimate takeaway

The lesson isn't asking you to memorize:

```text
NTP
DHCP
DNS
SSH
```

as four exam acronyms.

It's asking you to develop a **deployment standard**.

Your mental process should become:

```text
New device
    ↓
Baseline configuration
    ↓
NTP
DNS
SSH
    ↓
DHCP if required
    ↓
Device-specific configuration
    ↓
Security
    ↓
Documentation
    ↓
Production
```

The goal is that eventually **you feel something is missing when a newly deployed device doesn't have these fundamentals**.

That's the real meaning of:

> **"Build the habit, not just the lab."**

And this is the final concept of **Skill 22 — IP Services** before you move into **Skill 23: Network Monitoring**, beginning with SNMP on August 17. Your study plan confirms that August 14 completes Skill 22 with this **"What Now?"** lesson and the **Securing Castle Rysen Services** lab. 
