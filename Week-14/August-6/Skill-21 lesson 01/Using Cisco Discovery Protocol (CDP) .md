# Skill 21 — Lesson 01: Using Cisco Discovery Protocol (CDP)

This lesson takes the idea from Lesson 00 and turns it into a **practical troubleshooting tool**: use CDP to discover the Cisco devices directly connected to the device you're currently on.

---

## 1. What is CDP?

**CDP = Cisco Discovery Protocol**

It is a **Cisco-developed neighbor discovery protocol**.

Cisco devices running CDP periodically send discovery information to their **directly connected neighbors**.

The most important word is:

> **Directly connected**

CDP does **not** give you a complete network map from one command.

Example:

```text
             R1
              |
              |
             SW1
            /   \
           /     \
         SW2     AP1
```

If you're logged into `SW1`, CDP can help you discover:

```text
SW1 → R1
SW1 → SW2
SW1 → AP1
```

But it doesn't magically discover every device several hops away.

Instead:

```text
SW1
 ↓
Discover SW2
 ↓
Log into SW2
 ↓
Discover its neighbors
 ↓
Continue mapping
```

Think of it as **exploring the network one hop at a time**.

---

# 2. The most important CDP command

## `show cdp neighbors`

This gives you a **summary of directly connected CDP neighbors**.

```cisco
show cdp neighbors
```

The information is useful for answering:

> **Who is connected to me, and through which interfaces?**

You can typically determine:

* Neighbor device ID
* Your local interface
* Remote interface
* Device/platform information
* Device capabilities

For example, conceptually:

```text
Device ID     Local Port     Platform       Remote Port
R1            Gi0/1         Router         Gi0/0
SW2           Gi0/2         Switch         Gi0/1
AP1           Gi0/3         AP             Gi0
```

Now you can start drawing your topology:

```text
             R1
              |
            Gi0/1
              |
             SW1
            /   \
       Gi0/2     Gi0/3
          |         |
         SW2        AP1
```

That is the real value of CDP.

---

# 3. `show cdp neighbors detail`

When the summary isn't enough, use:

```cisco
show cdp neighbors detail
```

This provides **more detailed information about each neighbor**.

One particularly useful piece of information is the **IP address of the neighboring device**.

That changes the workflow from:

```text
"I know something is connected."
```

to:

```text
"I know what device it is,
I know its interface,
and I may know its IP address."
```

You can then potentially use that information to access the next device and continue your topology discovery.

### Practical workflow

```text
SW1
 │
 ├── show cdp neighbors detail
 │
 ↓
Discover R1's IP
 │
 ↓
Access R1
 │
 ↓
show cdp neighbors detail
 │
 ↓
Discover R1's neighbors
 │
 ↓
Continue
```

This is extremely useful when documentation is poor.

---

# 4. CDP timers

The lesson gives you two important timers.

### CDP advertisement interval

**60 seconds**

CDP sends updates periodically.

Think:

```text
Every 60 seconds
       ↓
"Hey, I'm still here."
```

### CDP holdtime

**180 seconds**

The neighbor information can remain in the table for up to approximately **180 seconds** after the last received CDP advertisement.

So:

```text
CDP update
    ↓
Neighbor appears
    ↓
Device disappears
    ↓
Entry may remain temporarily
    ↓
180-second holdtime expires
    ↓
Entry removed
```

### Why does this matter?

Suppose you unplug a Cisco device and immediately run:

```cisco
show cdp neighbors
```

You might still see that device.

That doesn't necessarily mean the physical connection is still working.

The CDP entry may simply be **waiting for its holdtime to expire**.

### Remember

```text
60 sec  → Advertisement interval
180 sec → Holdtime
```

---

# 5. CDP can carry useful operational information

CDP isn't limited to:

> "Hi, I'm R1."

It can also provide information that helps devices operate together.

The lesson gives two important examples.

---

## PoE

**PoE = Power over Ethernet**

A switch can provide electrical power to devices such as:

* IP phones
* Wireless access points

CDP can help communicate information about **power requirements**.

Conceptually:

```text
Switch
  │
  │ Ethernet + Power
  │
  ↓
IP Phone
  │
  └── "I need this much power."
```

This helps the switch manage its available power budget rather than unnecessarily reserving maximum power for every device.

---

# 6. Voice VLAN information

CDP can also help an IP phone learn information about the **voice VLAN**.

For example:

```text
Switch
   │
   │ CDP
   ↓
IP Phone
   │
   └── learns voice-related information
```

This allows the phone to be placed into the appropriate logical network segment for voice traffic.

So CDP can contribute to more than simple topology discovery.

---

# 7. CDP is useful — but there is a security concern

This is an important operational point.

CDP can reveal information about network infrastructure.

An attacker could potentially capture CDP traffic and learn information such as:

* Device identity
* Device names
* Platform information
* Software/version information
* Network topology details

That's useful for **reconnaissance**.

So you shouldn't think:

> "CDP is useful, therefore I should enable it everywhere."

Instead:

> **Use CDP where you need it, and consider disabling it where users/endpoints don't need to see it.**

---

# 8. Disabling CDP

### Disable CDP globally

```cisco
no cdp run
```

This disables CDP on the device.

### Disable CDP on a specific interface

```cisco
interface GigabitEthernet0/1
 no cdp enable
```

This gives you much more granular control.

For example:

```text
Network-device links
        ↓
       CDP
     ENABLED

User-facing ports
        ↓
       CDP
     DISABLED
```

The lesson's practical recommendation is essentially:

> Keep CDP where it provides operational value, particularly between network devices, while considering disabling it on user-facing ports.

---

# 9. CDP vs LLDP

You'll learn LLDP next, but establish this distinction now:

|               | CDP                               | LLDP                               |
| ------------- | --------------------------------- | ---------------------------------- |
| Full name     | Cisco Discovery Protocol          | Link Layer Discovery Protocol      |
| Nature        | Cisco's discovery protocol        | Standards-based discovery protocol |
| Environment   | Cisco-focused                     | Multi-vendor                       |
| Main purpose  | Neighbor discovery                | Neighbor discovery                 |
| Key advantage | Very useful in Cisco environments | Interoperability                   |

### Memory trick

```text
CDP
C → Cisco

LLDP
L → Link Layer
D → Discovery
P → Protocol
```

And remember:

> **CDP = Cisco-specific discovery**
>
> **LLDP = standards-based, multi-vendor discovery**

---

# 🔥 The troubleshooting mindset

This is probably the most valuable part of the lesson.

Imagine you inherit this:

```text
                  ???
                   |
                   |
        ??? ---- SW1 ---- ???
                   |
                  ???
```

Don't start randomly configuring things.

Start by asking the network:

```cisco
show cdp neighbors
```

Then:

```cisco
show cdp neighbors detail
```

You gradually turn:

```text
Unknown network
```

into:

```text
             R1
              |
             SW1
            /   \
          SW2   AP1
```

Then move to the next device.

### In other words:

> **CDP doesn't show you the whole network. It shows you the next step.**

That's an excellent way to think about it.

---

# 🧠 CCNA Must-Know

### Commands

```cisco
show cdp neighbors
show cdp neighbors detail
show cdp
no cdp run
no cdp enable
```

### Numbers

```text
CDP update interval = 60 seconds
CDP holdtime        = 180 seconds
```

### Concepts

```text
CDP
 ↓
Cisco neighbor discovery
 ↓
Directly connected devices
 ↓
Device/interface information
 ↓
Topology discovery
 ↓
Troubleshooting
```

### Security

```text
CDP information
      ↓
Can reveal infrastructure details
      ↓
Potential reconnaissance value
      ↓
Disable where unnecessary
```

---

## 🎯 The one thing to take away

When you log into an unfamiliar Cisco switch or router, one of your first questions should be:

> **"What's connected to me?"**

CDP gives you a fast answer.

And once you understand `show cdp neighbors` and `show cdp neighbors detail`, you're no longer just memorizing a protocol—you have a practical tool for **network discovery, topology mapping, and troubleshooting**.
