# Skill 26 — Lesson 00: Why Is This Important?

## 1. Core Idea

This lesson introduces **Network Automation and Programmability**.

At this stage, the goal is **awareness and understanding**, not becoming an automation engineer.

The central idea is:

> **Network automation is about achieving consistency at scale.**

When a network has only a few devices, manually configuring them may be manageable. But as the network grows to dozens, hundreds, or thousands of devices, manual configuration becomes:

* Slow
* Repetitive
* Error-prone
* Difficult to maintain consistently

---

# 2. Why Network Automation Matters

The fundamental problem is **human inconsistency**.

Network engineers can:

* Make typing mistakes
* Forget a configuration
* Miss an update
* Configure one device differently
* Get interrupted while performing repetitive work

For example, imagine manually configuring 500 switches:

```text
Switch 1  → Manual configuration
Switch 2  → Manual configuration
Switch 3  → Manual configuration
...
Switch 500 → Manual configuration
```

Even if you intend to apply exactly the same configuration, eventually you can get:

```text
499 switches → Correct
1 switch     → Slightly different
                         ↓
                   Unexpected issue
```

The problem becomes difficult to troubleshoot because the network is supposed to be standardized.

---

# 3. NetworkChuck Coffee Example

Imagine NetworkChuck Coffee expands from one store to many locations.

Every location might need:

* Routers
* Switches
* Wireless equipment
* VLANs
* Security settings
* NTP configuration
* Access-control configuration
* Standard device configurations

At a small scale:

```text
3 devices
   ↓
Manual configuration
   ↓
Reasonable
```

At a larger scale:

```text
500 devices
   ↓
Manual configuration
   ↓
Slow + repetitive + risky
```

Automation provides a better approach:

```text
                    Automation
                        │
             ┌──────────┼──────────┐
             ↓          ↓          ↓
          Router     Switch       WAP
             │          │          │
             └──── Standardized ───┘
                     configuration
```

The objective is **repeatability**.

---

# 4. Automation Does NOT Replace Networking

A very important distinction:

> **Automation isn't about replacing networking. It's about replacing repetitive manual work.**

You still need to understand:

* Networking
* Device configuration
* Security
* Routing
* Switching
* Troubleshooting

Automation simply gives you a better mechanism for applying and managing those things across many devices.

Think of it as:

```text
Network knowledge
       +
Automation
       ↓
Scalable network operations
```

If you don't understand what the network should look like, automating the wrong configuration just lets you make mistakes faster.

---

# 5. Automation at Scale

The value of automation increases as the network grows.

### Small environment

```text
5 devices
↓
Manual configuration
↓
Possible
```

### Medium environment

```text
50 devices
↓
Manual configuration
↓
Becomes painful
```

### Large environment

```text
500–5,000 devices
↓
Manual configuration
↓
Extremely difficult to maintain
```

At large scale, automation becomes much more than a convenience.

It becomes a practical method of maintaining:

* **Consistency**
* **Repeatability**
* **Efficiency**

---

# 6. The "Snowflake" Network Problem

A particularly important concept is avoiding **snowflake networks**.

A snowflake is an environment where each device or location becomes slightly different from the others.

For example:

```text
Store A
 ├── VLAN 10
 ├── NTP configured
 └── Standard security

Store B
 ├── VLAN 10
 ├── NTP missing
 └── Standard security

Store C
 ├── VLAN 20
 ├── NTP configured
 └── Different security
```

Now every location behaves differently.

That's exactly what automation can help prevent.

Instead, the goal is:

```text
                    Standard Template
                           │
            ┌──────────────┼──────────────┐
            ↓              ↓              ↓
         Store A         Store B        Store C
            │              │              │
            └──────── Same baseline ──────┘
```

### Desired outcome

**Repeatable deployments**

**Consistent configurations**

**Faster rollouts**

**Fewer mistakes**

---

# 7. Ansible

The lesson specifically introduces **Ansible**.

### What is Ansible?

**Ansible is an open-source automation tool** that can push configurations to network devices in a repeatable way.

The important CCNA-level understanding is:

```text
Ansible
   ↓
Automation tool
   ↓
Communicates with devices
   ↓
Applies repeatable configurations
```

Ansible is **not a Cisco-created tool**.

It is an open-source tool used broadly in IT and network environments.

---

# 8. Exam Version vs. Real-World Version

This distinction is important.

### CCNA exam perspective

You primarily need to understand:

* What network automation is
* Why automation is useful
* Why automation becomes important at scale
* That automation improves consistency
* That tools such as **Ansible** exist

You aren't expected from this lesson to become an automation engineer.

### Real-world perspective

Network automation can involve much more:

* Linux servers
* APIs
* Scripting
* Automation tools
* Inventory files
* Credentials
* Templates
* Configuration management
* Orchestration

The lesson describes the CCNA material as essentially the **tip of the iceberg**.

```text
             ┌─────────────────┐
             │   CCNA LEVEL    │
             │   Awareness     │
             └────────┬────────┘
                      │
                 TIP OF ICEBERG
                      │
──────────────────────┼──────────────────────
                      │
             ┌────────▼────────┐
             │     REAL-WORLD  │
             │    AUTOMATION   │
             ├─────────────────┤
             │ Linux           │
             │ APIs            │
             │ Scripting       │
             │ Ansible         │
             │ Inventory       │
             │ Credentials     │
             │ Templates       │
             │ Orchestration   │
             └─────────────────┘
```

---

# 9. Orchestration

The lesson also introduces **orchestration**.

Here, orchestration means:

> **Coordinated automation across systems.**

Instead of automating one isolated action, orchestration coordinates multiple automated operations.

Conceptually:

```text
             Orchestration
                   │
       ┌───────────┼───────────┐
       ↓           ↓           ↓
   Configure    Update      Validate
    devices     devices     devices
       │           │           │
       └───────────┼───────────┘
                   ↓
             Desired state
```

You don't need to master orchestration yet. You need to recognize the concept.

---

# 10. When Should You Automate?

The lesson gives a very practical rule:

> **If you hear the same change requested repeatedly, consider whether it is a candidate for automation.**

For example:

```text
"Apply this standard configuration
to every new switch."

          ↓

Potential automation candidate
```

Another example:

```text
"Gather information from every router."

          ↓

Potential automation candidate
```

The recommendation is **not**:

> Automate everything immediately.

Instead:

> **Start with one safe, repetitive task.**

For example:

1. Gather device information.
2. Automate a standard configuration snippet.
3. Test it.
4. Build confidence.
5. Expand automation gradually.

---

# 11. Why Consistency Is So Important

Suppose every new NetworkChuck Coffee location needs the same baseline configuration.

Without automation:

```text
Engineer
   ↓
Configure Store 1
   ↓
Configure Store 2
   ↓
Configure Store 3
   ↓
Configure Store 4
   ↓
...
```

With automation:

```text
Standard configuration
        ↓
    Automation
        ↓
 ┌──────┼──────┐
 ↓      ↓      ↓
S1     S2     S3
 ↓      ↓      ↓
Same baseline configuration
```

This provides a much more repeatable deployment process.

---

# 12. Automation and Network Engineering

The lesson's broader point is that modern network engineers shouldn't only think:

> "How do I configure this device?"

They should also think:

> **"How can I manage this change more efficiently across many devices?"**

This is an important evolution in networking.

### Traditional mindset

```text
Configure device
Configure next device
Configure next device
...
```

### Modern mindset

```text
Define desired configuration
          ↓
Automate deployment
          ↓
Apply consistently
          ↓
Validate
          ↓
Manage at scale
```

---

# 13. NetworkChuck Coffee — The Big Picture

For NetworkChuck Coffee, imagine opening 100 new stores.

Every store needs a baseline:

```text
Router
Switch
WAP
VLANs
Security
NTP
Management
```

Manually configuring each location increases the possibility of differences.

Automation allows the organization to build a **repeatable deployment model**.

That means:

**New store → standard configuration → faster deployment → fewer errors**

This is the real business value.

---

# 14. Key Terms

| Term                   | Meaning                                                                                     |
| ---------------------- | ------------------------------------------------------------------------------------------- |
| **Network Automation** | Using software/tools to perform network management and configuration tasks automatically    |
| **Consistency**        | Keeping configurations and behavior standardized across devices                             |
| **Repeatability**      | Being able to perform the same operation reliably multiple times                            |
| **Ansible**            | Open-source automation tool capable of pushing repeatable configurations to network devices |
| **Orchestration**      | Coordinated automation across multiple systems/components                                   |
| **Scalability**        | Ability to manage increasing numbers of devices without proportional manual effort          |
| **Snowflake Network**  | A network/location that becomes uniquely configured instead of following a standard         |

---

# 15. CCNA Exam Takeaways ⭐

Know these especially well:

### ① Why automation?

**Consistency at scale.**

### ② What problem does automation solve?

Manual configuration is:

* Repetitive
* Slow
* Error-prone
* Difficult to scale

### ③ What is Ansible?

**An open-source automation tool that can push repeatable configurations to network devices.**

### ④ Does automation replace network engineers?

**No.**

It replaces repetitive manual tasks and allows engineers to manage networks more efficiently.

### ⑤ What should you automate first?

A **safe, repetitive task**, rather than trying to automate everything immediately.

### ⑥ What is orchestration?

**Coordinated automation across systems.**

---

# 🧠 One-Minute Revision

```text
                NETWORK AUTOMATION
                        │
                        ↓
               CONSISTENCY AT SCALE
                        │
          ┌─────────────┼─────────────┐
          ↓             ↓             ↓
       Faster        Fewer          Repeatable
      changes       mistakes        deployments
          │             │             │
          └─────────────┼─────────────┘
                        ↓
                   LARGE NETWORKS
                        │
                        ↓
                   Tools like
                     ANSIBLE
```

### The sentence to remember

> **Network automation is about consistency at scale.**

For your CCNA, don't get distracted by trying to learn the entire automation ecosystem from this lesson. Understand **why automation exists, what problem it solves, and recognize Ansible as an important automation tool**. The following Skill 26 lessons will build on this foundation with **SDN, Cisco automation platforms, RESTful APIs, Ansible/Puppet/Chef, and markup languages**. 
