# Skill 26 — Lesson 05: What Now?

## Ansible — Turning Knowledge into Practical Skill

> **Core idea:** Don't stop at understanding Ansible conceptually. Build a small lab, install Ansible, connect it to something, automate a real task, break it, fix it, and repeat.

This lesson is the **practical conclusion of Skill 26**. The previous lesson introduced Ansible, Puppet, Chef, YAML, JSON, and XML. This lesson focuses on what you should actually do next: **get hands-on experience with Ansible**.

---

# 1. The Biggest Mistake After Learning Ansible

It is easy to watch an Ansible lesson and think:

> "Yeah, I understand Ansible."

But understanding the concept isn't the same as being able to use the tool.

There is a major difference between:

```text
"I know what Ansible is."
```

and:

```text
"I installed Ansible,
created an inventory,
built a playbook,
connected to a device,
ran automation,
troubleshot an error,
and verified the result."
```

The second demonstrates **practical skill**.

### The lesson's philosophy

```text
Learn
  ↓
Build
  ↓
Break
  ↓
Troubleshoot
  ↓
Fix
  ↓
Repeat
  ↓
Skill
```

That's the transition from **knowledge → competence**.

---

# 2. You Don't Need an Enterprise Lab

One of the biggest points of the lesson is that you don't need:

* A datacenter
* Hundreds of switches
* Multiple routers
* Expensive Cisco equipment
* A rack full of hardware

You can start with:

* A Raspberry Pi
* A Linux server
* A virtual machine
* One router
* One switch

The important thing is that the environment is **real enough to interact with**.

---

# 3. Your Ansible Control Node

The lesson introduces the concept of an **Ansible box**.

This is the machine where Ansible is installed and from which automation tasks are launched.

A useful mental model is:

```text
              Ansible Control Node
                    Linux VM
                       |
                       |
                  Automation
                       |
          +------------+------------+
          |                         |
       Network                  Network
       Device 1                 Device 2
```

The control node is where you:

* Install Ansible
* Create inventories
* Write playbooks
* Define variables
* Run automation
* Review results
* Troubleshoot failures

---

# 4. Start Small

The recommended approach is:

> **Build something small.**

Don't begin by trying to automate an entire enterprise network.

Start with one simple objective.

For example:

```text
Linux VM
   ↓
Ansible
   ↓
One network device
   ↓
Run one task
```

Once that works:

```text
One device
   ↓
Two devices
   ↓
Five devices
   ↓
Multiple sites
```

This follows the same scalability principle used in real infrastructure.

---

# 5. A Simple Learning Progression

A good progression based on this lesson is:

### Stage 1 — Install

Get Ansible running on a Linux system.

```text
Linux VM
   ↓
Install Ansible
```

### Stage 2 — Connect

Get Ansible communicating with a target.

```text
Ansible
   ↓
Connection
   ↓
Device
```

### Stage 3 — Execute

Run a basic task.

```text
Ansible
   ↓
Task
   ↓
Device
   ↓
Result
```

### Stage 4 — Automate

Put multiple tasks into a playbook.

```text
Playbook
   ↓
Task 1
Task 2
Task 3
Task 4
```

### Stage 5 — Scale

Use the same automation against multiple devices.

```text
              Playbook
                  |
       +----------+----------+
       |          |          |
      R1         R2         R3
       |          |          |
    Same       Same       Same
   process    process    process
```

---

# 6. What Is a Playbook?

A **playbook** is Ansible's set of automation instructions.

Think of it as a recipe.

For example:

```text
PLAYBOOK
   |
   +-- Connect to switches
   |
   +-- Configure VLAN
   |
   +-- Configure SSH
   |
   +-- Configure management settings
   |
   +-- Verify configuration
```

Instead of manually remembering every command, the procedure is represented in a reusable automation file.

---

# 7. Manual Configuration vs Ansible

## Manual approach

Imagine Castle Rysen has 20 coffee shops.

Each shop has a switch.

You need to configure the same SSH settings everywhere.

You could do:

```text
SSH → SW1 → configure
SSH → SW2 → configure
SSH → SW3 → configure
SSH → SW4 → configure
...
SSH → SW20 → configure
```

The engineer has to repeat the process 20 times.

---

## Automated approach

Instead:

```text
                  Ansible
                     |
                 Playbook
                     |
       +-------------+-------------+
       |             |             |
      SW1           SW2           SW3
       |             |             |
       +-------------+-------------+
                     |
             Standard configuration
```

The automation describes the desired procedure and applies it across the infrastructure.

---

# 8. The Importance of Repeatability

This is one of the most important concepts in the entire automation section.

Suppose you need this configuration:

```text
SSH enabled
NTP configured
Management VLAN configured
Standard hostname
Standard security settings
```

You don't want:

```text
Device 1 → configuration A
Device 2 → configuration B
Device 3 → configuration A + random change
Device 4 → configuration C
```

You want:

```text
                 Standard
                Definition
                     |
                     ↓
                 Automation
                     |
        +------------+------------+
        |            |            |
       R1           R2           R3
        |            |            |
        ↓            ↓            ↓
     Standard     Standard     Standard
      result       result       result
```

This is **repeatability**.

---

# 9. Home Labs Have Real Value

The lesson makes an important career point:

A home lab isn't "fake experience."

A properly documented home lab can demonstrate:

* Initiative
* Curiosity
* Troubleshooting ability
* Willingness to learn
* Practical experimentation
* Understanding of automation
* Ability to work independently

There is a major difference between putting:

> "Studied Ansible"

on a resume and being able to explain:

> "I built an Ansible lab, created an inventory and playbook, connected to network devices, automated configuration, and documented the troubleshooting process."

The second statement gives you something concrete to discuss.

---

# 10. Document Your Lab

The lesson strongly recommends documenting what you do.

Keep:

* Playbooks
* Inventory files
* Configuration files
* Screenshots
* Network diagrams
* Commands
* Errors
* Troubleshooting steps
* Final results

For example:

```text
ansible-lab/
│
├── inventory/
│   └── hosts
│
├── playbooks/
│   ├── vlan.yml
│   ├── ssh.yml
│   └── backup.yml
│
├── variables/
│
├── documentation/
│   └── lab-notes.md
│
└── screenshots/
```

This turns random experimentation into a **portfolio project**.

---

# 11. Break Things on Purpose

One of the best learning techniques mentioned in the lesson is:

> **Build it, break it, fix it.**

For example:

```text
Working configuration
        ↓
Make a controlled change
        ↓
Something breaks
        ↓
Investigate
        ↓
Identify cause
        ↓
Fix
        ↓
Verify
```

This develops troubleshooting ability.

And troubleshooting is particularly important in networking because real networks don't fail in neat textbook ways.

---

# 12. NetworkChuck Coffee — Applying the Concept

The lesson returns to the Castle/NetworkChuck Coffee scenario.

Imagine a district shop has:

```text
                Router
                  |
                Switch
                  |
       +----------+----------+
       |          |          |
    Admin       Guest      Video
     VLAN        VLAN       VLAN
```

You might eventually want Ansible to automate things such as:

* Standard switch configuration
* VLAN configuration
* SSH configuration
* Configuration backups
* Standard management settings
* Access policies
* Repeated updates

Instead of manually logging into each device:

```text
Engineer → Device 1
Engineer → Device 2
Engineer → Device 3
Engineer → Device 4
...
```

you move toward:

```text
                Ansible
                   |
                Playbook
                   |
        +----------+----------+
        |          |          |
       Shop 1     Shop 2     Shop 3
        |          |          |
       SW         SW         SW
```

---

# 13. Scaling the Network

The real value of Ansible becomes increasingly obvious as Castle Rysen expands.

### One coffee shop

Manual configuration may be manageable.

```text
1 router
1 switch
1 WAP
```

### Three shops

Still manageable.

```text
3 routers
3 switches
3 WAPs
```

### 50 shops

Manual configuration starts becoming painful.

```text
50 routers
50 switches
50 WAPs
```

### Hundreds of devices

Now consistency becomes a major operational concern.

```text
             Automation
                  |
       +----------+----------+
       |          |          |
     Site 1     Site 2     Site 3
       |          |          |
      ...        ...        ...
```

This is where automation changes from:

> "Cool technology"

to:

> **Operational necessity.**

---

# 14. Automation as a Toolbox Skill

The lesson doesn't want you to treat Ansible as something you simply memorize for an exam.

Instead:

```text
Problem
   ↓
Ask: Can automation solve this?
   ↓
Build automation
   ↓
Test
   ↓
Deploy
```

Ansible becomes another tool in your network-engineering toolbox.

You don't necessarily use it for every task.

You use it when automation provides value.

---

# 15. What the CCNA Actually Expects

For this particular CCNA section, you don't need to become an advanced Ansible engineer.

The lesson's focus is primarily:

### Know what Ansible is

> Automation/configuration-management tool.

### Know why it matters to networking

> It is **agentless** and can automate network devices.

### Know what a playbook is

> A set of instructions/tasks Ansible executes.

### Know why hands-on experience matters

> Understanding the concept is different from actually using the tool.

### Know the related formats

```text
Ansible → YAML
REST APIs → JSON
XML → tag-based structured data
```

---

# 16. The Complete Skill 26 Picture

You've now gone through the entire automation progression:

```text
                NETWORK AUTOMATION
                       |
                       ↓
                Network Automation
                  and SDN concepts
                       |
                       ↓
                 Cisco SDN models
                       |
                       ↓
                    REST APIs
                       |
                       ↓
                     JSON
                       |
                       ↓
           Ansible / Puppet / Chef
                       |
                       ↓
                     YAML
                       |
                       ↓
                 Hands-on Lab
                       |
                       ↓
              Practical Automation
```

This is a very important transition in modern networking.

---

# 17. The Bigger Picture: From CLI to Automation

Earlier in your CCNA journey, the primary interaction model was:

```text
Engineer
   ↓
CLI
   ↓
Cisco IOS
   ↓
Configuration
```

Automation introduces another layer:

```text
Engineer
   ↓
Automation Code
   ↓
Ansible
   ↓
SSH / API
   ↓
Network Device
   ↓
Configuration
```

Instead of the engineer manually performing every operation, the engineer defines **how the operation should be performed** and lets the automation system execute it.

---

# 18. Automation Doesn't Eliminate Networking Knowledge

This is an important practical point.

Automation does **not** replace networking fundamentals.

You still need to understand:

* VLANs
* IP addressing
* Routing
* ACLs
* SSH
* STP
* EtherChannel
* DHCP
* DNS
* NAT
* Wireless
* Security
* Troubleshooting

Otherwise, you could automate the wrong configuration very efficiently.

Think:

```text
Networking knowledge
        +
Automation knowledge
        =
Better network engineer
```

Not:

```text
Automation
    replaces
Networking
```

---

# 19. Your Recommended Learning Path After This Lesson

Based directly on the lesson's recommendation, the practical progression is:

### Step 1

Get a Linux environment.

```text
Linux VM / Raspberry Pi / Linux server
```

### Step 2

Install Ansible.

```text
Linux
  ↓
Ansible
```

### Step 3

Create a small inventory.

```text
Device 1
Device 2
```

### Step 4

Connect to a device.

```text
Ansible
   ↓
SSH / supported network connection
   ↓
Cisco device
```

### Step 5

Create a basic playbook.

```text
Playbook
   ↓
Task
   ↓
Device
```

### Step 6

Automate something useful.

For example:

```text
Backup configuration
```

or:

```text
Configure a VLAN
```

or:

```text
Standardize SSH settings
```

### Step 7

Break something intentionally.

### Step 8

Troubleshoot it.

### Step 9

Document everything.

### Step 10

Expand from one device to multiple devices.

---

# 20. Interview Value

A small Ansible project can give you considerably better interview material than simply saying:

> "I learned Ansible."

You can explain:

**Environment**

```text
Linux VM
```

**Automation**

```text
Ansible
```

**Target**

```text
Cisco network device
```

**Configuration**

```text
VLAN / SSH / backup / management settings
```

**Method**

```text
Inventory + Playbook
```

**Troubleshooting**

```text
Connection failure
↓
Investigated
↓
Fixed
↓
Verified
```

That demonstrates practical understanding.

---

# 🧠 Final Mental Model

Remember the entire lesson using this:

```text
        DON'T JUST LEARN ANSIBLE
                 ↓
             INSTALL IT
                 ↓
             BUILD A LAB
                 ↓
          CONNECT A DEVICE
                 ↓
          WRITE A PLAYBOOK
                 ↓
           RUN AUTOMATION
                 ↓
             BREAK IT
                 ↓
            TROUBLESHOOT
                 ↓
                FIX
                 ↓
             DOCUMENT
                 ↓
              REPEAT
                 ↓
        PRACTICAL EXPERIENCE
                 ↓
          SCALE TO MORE DEVICES
```

## ⭐ Key CCNA Takeaways

| Concept       | Remember                                                       |
| ------------- | -------------------------------------------------------------- |
| Ansible       | Automation/configuration-management tool                       |
| Agentless     | Major reason Ansible is attractive for networking              |
| Playbook      | Ansible automation instructions                                |
| Linux         | Common environment for running automation tools                |
| Lab           | You don't need enterprise hardware                             |
| One device    | Enough to start learning                                       |
| Documentation | Turns lab work into reusable knowledge and portfolio evidence  |
| Scaling       | Automation becomes increasingly valuable as device count grows |
| Main goal     | **Turn knowledge into practical skill**                        |

### One sentence to remember

> **The goal of learning Ansible isn't to know what Ansible is; it's to be able to use it to repeatedly and reliably perform real infrastructure tasks.**

And with this lesson, **Skill 26 is complete**. Your Summer of CCNA planner lists August 28 as the final day, with this lesson followed by its one-question quiz. 
