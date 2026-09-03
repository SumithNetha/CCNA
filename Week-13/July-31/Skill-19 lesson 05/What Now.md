# Week 13 — July 31

## Skill 19 Lesson 05 — What Now?

This lesson is essentially the **transition point between security concepts and security implementation**. In your study plan, it is the final lesson of Skill 19, followed by the introduction to Skill 20. 

---

## 1. The Main Purpose of This Lesson

The lesson's central message is:

> **Don't implement security controls just because you know the commands. Understand the threat first, then choose the control that addresses it.**

The previous Skill 19 lessons built the security foundation:

```text
Vulnerabilities
      ↓
Exploits
      ↓
Threats
      ↓
Types of attacks
      ↓
Identity & password attacks
      ↓
Cisco AAA
      ↓
Security implementation
```

So this lesson closes the **conceptual security section** and prepares you for hands-on defensive configuration.

---

# 2. Security Isn't Just Memorizing Commands

A common beginner approach is:

```text
Learn command
     ↓
Configure feature
     ↓
Done
```

The lesson argues for a different approach:

```text
Understand threat
       ↓
Identify vulnerability
       ↓
Determine what needs protection
       ↓
Choose appropriate security control
       ↓
Configure it
       ↓
Verify effectiveness
```

### Why?

Because a security feature is only useful when it addresses an actual risk.

For example, configuring a security feature without understanding the threat can result in:

* Protecting the wrong thing
* Overengineering
* Unnecessary complexity
* Failing to address the real attack surface

---

# 3. Think Like a Defender

The lesson describes network security as understanding the **battlefield**.

You should be asking:

```text
What are we protecting?
        ↓
Who/what are we protecting it from?
        ↓
Where are the weak points?
        ↓
How could an attacker move through the environment?
        ↓
What control reduces that risk?
```

This is the difference between simply knowing security technologies and actually **designing security**.

---

# 4. NetworkChuck Coffee Example

Consider a coffee shop network containing:

```text
              Coffee Shop
                   │
       ┌───────────┼────────────┐
       │           │            │
      POS       Guest Wi-Fi   Internal
    Systems                   Systems
                               │
                    ┌──────────┼──────────┐
                    │          │          │
                 Cameras    Servers    Admin PCs
```

You shouldn't simply configure every security feature available.

Instead, identify the assets and threats.

### Example questions

**What are we protecting?**

* POS systems
* Internal systems
* Servers
* Cameras
* Administrative devices

**What shouldn't have access to those systems?**

* Guest devices
* Compromised endpoints
* Unauthorized users

Then design controls accordingly.

---

# 5. Security as a System

One of the most important ideas in this lesson is that security technologies shouldn't be viewed as isolated features.

For example:

### VLAN

Not simply:

> "A VLAN is a logical Layer 2 network."

Instead:

> "I can use segmentation to separate different trust zones."

### ACL

Not simply:

> "An ACL permits or denies traffic."

Instead:

> "I can use an ACL to enforce which traffic is allowed between security zones."

### Authentication

Not simply:

> "It checks a password."

Instead:

> "It allows the organization to control and verify who receives access."

So:

```text
VLAN + ACL + Authentication + Switch Security
                    ↓
             Defensive Strategy
```

Each technology contributes to the larger security architecture.

---

# 6. Security Architecture vs Individual Features

Think of it this way:

```text
                 SECURITY
                    │
       ┌────────────┼────────────┐
       │            │            │
   Identity      Network       Endpoint
    Security     Security       Security
       │            │
      AAA       VLAN / ACL
       │            │
      SSH       L2 Security
       │            │
       └────────────┼────────────┘
                    ↓
              Risk Reduction
```

The objective isn't:

> "Use as many security features as possible."

The objective is:

> **Use appropriate controls to reduce the relevant risks.**

---

# 7. Security Implementation Starts Now

The lesson marks a transition:

### Before

```text
"What could happen?"
"What attacks exist?"
"Why are they dangerous?"
```

### Now

```text
"How do we prevent it?"
"How do we configure the protection?"
"How do we verify it?"
```

This is the shift from **security theory → security implementation**.

---

# 8. The Real-World Security Question

The lesson gives an important approach:

> **"What are we trying to protect, and from whom?"**

This should become one of your core security-design questions.

Don't begin with:

```text
"What security features does my Cisco switch have?"
```

Begin with:

```text
What is valuable?
      ↓
Who needs access?
      ↓
Who shouldn't have access?
      ↓
What threats exist?
      ↓
What controls should I implement?
```

---

# 9. Preventing Lateral Movement

The lesson gives the example of preventing a compromised device from becoming a **"freeway" into the whole business**.

This is essentially the idea of limiting how far an attacker can move after compromising one system.

For example:

```text
Guest Device
     │
     │ Compromised
     ▼
Guest VLAN
     │
     X
     │
     X──► POS
     X──► Admin PCs
     X──► Cameras
     X──► Network Devices
```

Segmentation and access controls can help prevent a compromise in one area from automatically becoming a compromise of the entire network.

---

# 10. Castle Rysen Connection

This is particularly relevant to the Castle Rysen design.

The RFP requires **distinct network segments** and security boundaries. For example, district shops must separate patron devices from administrative devices, while guest access is restricted to specific resources. 

It also requires:

* ACLs
* Layer 2 security
* Port security
* DHCP Snooping
* DAI
* Secure management access
* Network monitoring



So the Castle Rysen project isn't simply:

```text
Configure VLANs
Configure ACLs
Configure port security
```

It is:

```text
Identify assets
      ↓
Identify threats
      ↓
Create trust boundaries
      ↓
Restrict communication
      ↓
Protect Layer 2
      ↓
Secure administration
      ↓
Monitor the environment
```

---

# 11. Why the Foundation Matters

The lesson strongly emphasizes that **understanding is more valuable than memorized procedures**.

Imagine you memorize:

```text
Command A
Command B
Command C
```

Then the network topology changes.

Your memorized procedure may no longer work.

But if you understand:

```text
Threat
  ↓
Vulnerability
  ↓
Risk
  ↓
Security control
```

you can adapt your solution to the new environment.

---

# 12. Exam Knowledge vs Job Knowledge

A useful distinction from the lesson:

### Exam mindset

```text
"What does this feature do?"
```

### Real-world mindset

```text
"Why did you deploy this feature here?"
"What problem does it solve?"
"What risk does it reduce?"
"What happens if it isn't implemented?"
```

For your CCNA preparation, you need both.

---

# 13. Security Design Methodology

You can turn the lesson into a practical methodology:

### Step 1 — Identify assets

```text
Servers
Users
Applications
Network devices
Cameras
POS systems
Data
```

### Step 2 — Identify threats

```text
Unauthorized users
Compromised devices
Credential attacks
Layer 2 attacks
Malicious traffic
```

### Step 3 — Identify attack surfaces

```text
User ports
Wi-Fi
Management interfaces
Internet edge
Network services
```

### Step 4 — Select controls

```text
Segmentation
ACLs
AAA
SSH
Port Security
DHCP Snooping
DAI
```

### Step 5 — Implement

Configure the actual network devices.

### Step 6 — Verify

Check whether the security control actually works.

---

# 14. The Important Shift

This entire lesson can be summarized as:

```text
          SECURITY THEORY
                │
                ▼
      Understand the Threat
                │
                ▼
       Identify What to Protect
                │
                ▼
       Choose Security Control
                │
                ▼
          CONFIGURE IT
                │
                ▼
            VERIFY IT
                │
                ▼
          REDUCE THE RISK
```

---

# 15. What Comes Next?

Your study plan shows that after July 31:

**Skill 20 — Layer 2 Security**

starts on August 3. It covers:

* **Configuring Switch Port Security**
* **Configuring DHCP Snooping**
* **Configuring DAI**
* Castle Rysen Layer 2 Security Deployment
* Associated labs and quizzes. 

So the transition is:

```text
Skill 19
Security Concepts
       │
       ├── Vulnerabilities
       ├── Threats
       ├── Attacks
       ├── Identity attacks
       └── AAA
              │
              ▼
        "What Now?"
              │
              ▼
Skill 20
Layer 2 Security
       │
       ├── Port Security
       ├── DHCP Snooping
       └── DAI
```

---

# 16. Key Takeaways

### ① Don't configure security blindly

Understand the threat before selecting the control.

### ② Security is a system

VLANs, ACLs, AAA, SSH, Layer 2 security, etc. work together.

### ③ Protect the right assets

Identify what's valuable before deciding what to secure.

### ④ Think about lateral movement

A compromised endpoint shouldn't automatically provide a path to everything else.

### ⑤ Understand instead of memorizing

When topology or requirements change, understanding lets you adapt.

### ⑥ Security implementation is next

The conceptual foundation is now complete; the next skill starts putting defensive controls into actual Cisco configurations.

---

## 🧠 One sentence to remember

> **Good network security isn't "Which security feature should I configure?" — it's "What am I protecting, from whom, how could they attack it, and which control reduces that risk?"**

