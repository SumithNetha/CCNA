# Week 13 — July 31

# Skill 19 — Understanding the Place of Cisco AAA

> **Core idea:** AAA is not simply a password mechanism. It is an **access-control framework** that answers three questions: **Who are you? What are you allowed to do? What did you do?**

---

## 1. What is Cisco AAA?

**AAA = Authentication, Authorization, Accounting**

Cisco AAA provides a framework for controlling access to network infrastructure and tracking activity.

| AAA component      | Question                    | Purpose                |
| ------------------ | --------------------------- | ---------------------- |
| **Authentication** | Who are you?                | Verifies identity      |
| **Authorization**  | What are you allowed to do? | Determines permissions |
| **Accounting**     | What did you do?            | Records activity       |

### Easy memory trick

```text
Authentication → WHO?
Authorization  → WHAT CAN I DO?
Accounting     → WHAT DID I DO?
```

AAA is therefore about **identity, access control, and accountability**.

---

# 2. Why AAA Exists

In a small network, you could create local accounts on every device:

```text
Router 1 → local users
Router 2 → local users
Switch 1 → local users
Switch 2 → local users
Firewall → local users
WAP      → local users
```

This might work for a few devices.

But imagine an enterprise with dozens or hundreds of network devices.

If an administrator:

* changes their password
* leaves the company
* changes roles
* needs access revoked

you would potentially have to modify accounts on **many individual devices**.

That does not scale well.

---

# 3. Centralized Authentication

AAA allows network devices to use a **centralized identity/access system**.

Instead of each device independently maintaining every user's account:

```text
                    AAA / Identity System
                           │
              ┌────────────┼────────────┐
              │            │            │
              ▼            ▼            ▼
            Router       Switch         WAP
```

When someone attempts to log in, the network device can ask the centralized AAA system to make the access decision.

### Basic flow

```text
Administrator
      │
      │ Credentials
      ▼
Network Device
      │
      │ Authentication request
      ▼
AAA Server
      │
      │ Accept / Reject
      ▼
Network Device
      │
      ▼
Access granted/denied
```

---

# 4. AAA Is an Architecture, Not Necessarily a Single Cisco Box

An important point from the lesson:

**AAA is an idea/framework and architecture, not necessarily one specific Cisco appliance.**

The centralized identity system could connect to:

* **Microsoft Active Directory**
* Linux-based identity systems
* Cloud identity platforms
* Other centralized identity sources

So think of AAA as the **access-control architecture**, rather than automatically thinking:

> "AAA = a particular Cisco server."

---

# 5. Authentication

### Definition

**Authentication verifies a user's identity.**

It answers:

> **"Who are you?"**

For example:

```text
Username: admin
Password: ********
```

The system checks whether those credentials correspond to a legitimate identity.

### Authentication failure

```text
User
 ↓
Credentials
 ↓
AAA
 ↓
Invalid
 ↓
ACCESS DENIED
```

### Authentication success

```text
User
 ↓
Credentials
 ↓
AAA
 ↓
Valid
 ↓
Continue to authorization
```

---

# 6. Authorization

Authentication does **not** automatically mean the user should have unlimited access.

Authorization determines what an authenticated user is allowed to do.

Example:

```text
Administrator A
→ Full network administration

Administrator B
→ Limited configuration access

Administrator C
→ Read-only access
```

Therefore:

```text
Authentication
      ↓
"Yes, this is Sumith."
      ↓
Authorization
      ↓
"What is Sumith allowed to access?"
```

This distinction is extremely important.

---

# 7. Accounting

**Accounting records activity after access has been granted.**

It can provide:

* Logs
* Monitoring information
* Audit trails
* Evidence of activity

The basic question is:

> **"What did this user do?"**

For example:

```text
User: NetworkAdmin
Login: 10:15 AM
Device: Router-01
Activity: Configuration changes
Logout: 10:47 AM
```

Accounting provides **accountability**.

---

# 8. Why Accounting Matters

Suppose a network suddenly stops working.

Someone changed the configuration.

Without proper records:

```text
Who changed it?
     ↓
Unknown
```

With accounting:

```text
Configuration change
        ↓
Audit record
        ↓
Administrator identified
        ↓
Investigate change
```

This is useful for:

* Troubleshooting
* Auditing
* Security investigations
* Accountability

---

# 9. AAA Solves the Scale Problem

Consider NetworkChuck Coffee growing from one shop into a large organization.

### Without centralized AAA

```text
              Local Accounts
              /     |      \
             /      |       \
         Router   Switch    WAP
```

Every device manages users independently.

### With centralized AAA

```text
                 Central Identity
                       │
       ┌───────────────┼───────────────┐
       │               │               │
     Router          Switch            WAP
```

Now access can be managed centrally.

---

# 10. Employee Departure Example

This is one of the most important real-world examples from the lesson.

### Local accounts

An administrator leaves:

```text
Employee leaves
      ↓
Find every network device
      ↓
Remove/disable account
      ↓
Repeat across devices
```

This is slow and error-prone.

### Centralized AAA

```text
Employee leaves
      ↓
Disable central account
      ↓
Access removed from systems
      ↓
Done
```

This gives security teams much greater control over **employee changes, contractor turnover, and emergency lockouts**.

---

# 11. Security Is Also About Trusted Users

A major point in the lesson:

Security isn't only about stopping unknown attackers.

You also have to properly manage **people who are supposed to have access**.

A legitimate account can still become a security problem if:

* It has excessive privileges.
* It isn't disabled after the employee leaves.
* Its credentials are compromised.
* Access isn't properly controlled.

Therefore:

```text
Security
├── Stop unauthorized access
└── Properly control authorized access
```

AAA primarily helps with the second part while also contributing to the first.

---

# 12. RADIUS

One of the major protocols associated with AAA is:

**RADIUS = Remote Authentication Dial-In User Service**

The lesson describes RADIUS as the **broad, common industry-standard protocol** used to communicate with AAA systems.

Conceptually:

```text
Network Device
      │
      │ RADIUS
      ▼
AAA System
```

The protocol carries information involved in authentication, authorization, and accounting.

---

# 13. TACACS+

The second major protocol is:

**TACACS+ = Terminal Access Controller Access-Control System Plus**

The lesson strongly associates TACACS+ with **Cisco administration and device-management use cases**.

Conceptually:

```text
Cisco Router/Switch
        │
        │ TACACS+
        ▼
   AAA System
```

---

# 14. RADIUS vs TACACS+

For this lesson, remember the practical distinction:

|                              | RADIUS         | TACACS+                  |
| ---------------------------- | -------------- | ------------------------ |
| AAA-related protocol         | Yes            | Yes                      |
| Centralizes access decisions | Yes            | Yes                      |
| Common/broad standard        | **Yes**        | Less broad               |
| Strong association           | Network access | **Cisco administration** |
| Main lesson takeaway         | Broad/common   | Device administration    |

### Memory shortcut

```text
RADIUS   → Broad network access
TACACS+  → Cisco/device administration
```

Don't over-focus on protocol internals yet. The lesson explicitly emphasizes understanding **their role** before going deeper into configuration.

---

# 15. AAA Is Not Only for Network Administrators

AAA concepts also apply to **end-user network access**.

This becomes particularly important with wireless networks.

The lesson introduces:

* **802.1X**
* **EAP**
* **WPA2-Enterprise**

---

# 16. Traditional Pre-Shared Key

With a normal shared Wi-Fi password:

```text
              Wi-Fi
                │
       ┌────────┼────────┐
       │        │        │
     User A   User B   User C
       │        │        │
       └── Same password ──┘
```

Everyone uses the same credential.

### Problem

If an employee leaves:

```text
Employee leaves
      ↓
Still knows Wi-Fi password
      ↓
Potential continued access
```

You may need to change the shared password and update all legitimate devices.

---

# 17. 802.1X

**802.1X** provides a framework for controlling network access based on authentication.

Instead of everyone sharing one Wi-Fi password:

```text
User A → individual identity
User B → individual identity
User C → individual identity
```

A device can authenticate before receiving normal network access.

---

# 18. EAP

**EAP = Extensible Authentication Protocol**

EAP is part of the authentication framework used with technologies such as 802.1X.

Simplified:

```text
Client
  │
  │ Authentication
  ▼
802.1X / EAP
  │
  ▼
AAA infrastructure
```

The lesson's key point is not to memorize every EAP variant yet, but to understand that **EAP supports flexible authentication methods within this architecture**.

---

# 19. WPA2-Enterprise

The lesson uses **WPA2-Enterprise** as an example of user-based wireless authentication.

Instead of:

```text
Everyone
   ↓
ONE shared Wi-Fi password
```

you can have:

```text
User A ──┐
User B ──┼──► Individual authentication
User C ──┘
```

This allows an organization to apply different policies.

For example:

```text
Employee
   ↓
Authenticated
   ↓
Internal network access

Guest
   ↓
Authenticated/restricted
   ↓
Internet-only access
```

---

# 20. AAA as a Policy Engine

This is an important conceptual evolution.

AAA isn't simply:

> "Check whether the password is correct."

It can become part of a broader **policy-based access architecture**.

For example:

```text
                Identity
                   │
                   ▼
              Authentication
                   │
                   ▼
              Authorization
                   │
        ┌──────────┼──────────┐
        ▼          ▼          ▼
     Employee     Guest     Restricted
        │          │          │
        ▼          ▼          ▼
    Internal     Internet   Limited
     Access       Only      Access
```

So identity can influence what type of access a user receives.

---

# 21. Cisco AAA Big Picture

Put everything together:

```text
                         AAA
                          │
          ┌───────────────┼───────────────┐
          │               │               │
          ▼               ▼               ▼
   Authentication   Authorization    Accounting
       WHO?             WHAT?          WHAT DID?
          │               │               │
          └───────────────┼───────────────┘
                          │
                          ▼
                Centralized Access
                          │
              ┌───────────┴───────────┐
              │                       │
           RADIUS                  TACACS+
              │                       │
              └───────────┬───────────┘
                          │
                    Network Devices
```

And for user network access:

```text
             AAA Concepts
                  │
             802.1X / EAP
                  │
                  ▼
             Enterprise Wi-Fi
```

---

# 22. AAA vs Local Authentication

| Feature            | Local Login                        | Centralized AAA                 |
| ------------------ | ---------------------------------- | ------------------------------- |
| User database      | Individual device                  | Central identity system         |
| Scalability        | Poor                               | Good                            |
| Central management | No                                 | Yes                             |
| Employee removal   | Manual per device                  | Centralized                     |
| Auditing           | Limited depending on configuration | Stronger centralized capability |
| Large enterprise   | Difficult                          | Much more suitable              |
| Small network      | Often sufficient                   | May be unnecessary              |

---

# 23. AAA vs ACL

Don't confuse these.

### AAA

Deals primarily with:

```text
WHO
WHAT THEY CAN DO
WHAT THEY DID
```

### ACL

Deals with:

```text
WHICH TRAFFIC IS ALLOWED/DENIED
```

They can work together.

Example:

```text
Administrator
      │
      ▼
ACL
"Are you allowed to reach the device?"
      │
      ▼
SSH
"Can you establish a secure management session?"
      │
      ▼
AAA
"Who are you and what are you allowed to do?"
      │
      ▼
Cisco Device
```

This distinction becomes especially useful in the Castle Rysen project.

---

# 24. Castle Rysen Connection

The RFP requires administrative access to network devices to be restricted to specific management segments. At the café, administration is associated with the **Admin VLAN**, while the Fallout Shelter uses a **Management VLAN**. 

This can be combined with AAA:

```text
                    Network Device
                         │
             ┌───────────┴───────────┐
             │                       │
       Access restriction          AAA
            (ACL)                    │
             │            ┌──────────┼──────────┐
             │            │          │          │
             │       Authentication Authorization Accounting
             │
             ▼
       Admin/Management VLAN
```

So:

**ACL → controls where management traffic can originate.**

**SSH → protects the management session.**

**AAA → controls administrator identity, privileges, and accountability.**

---

# 25. Real-World Enterprise Flow

A realistic enterprise management scenario could look like:

```text
Administrator
     │
     ▼
Management Network
     │
     ▼
SSH
     │
     ▼
Cisco Switch
     │
     ▼
AAA
     │
     ├── Authentication
     │      ↓
     │   Verify identity
     │
     ├── Authorization
     │      ↓
     │   Determine privileges
     │
     └── Accounting
            ↓
        Record activity
```

This is much more robust than:

```text
Administrator
     ↓
Shared password
     ↓
Device
```

---

# 26. Important Terminology

| Term                           | Meaning                                                                 |
| ------------------------------ | ----------------------------------------------------------------------- |
| **AAA**                        | Authentication, Authorization, Accounting                               |
| **Authentication**             | Verifying identity                                                      |
| **Authorization**              | Determining permitted actions/access                                    |
| **Accounting**                 | Recording user activity                                                 |
| **RADIUS**                     | Common AAA protocol                                                     |
| **TACACS+**                    | AAA protocol strongly associated with Cisco device administration       |
| **802.1X**                     | Network access-control framework                                        |
| **EAP**                        | Extensible Authentication Protocol                                      |
| **WPA2-Enterprise**            | Enterprise wireless security using individual/user-based authentication |
| **Centralized authentication** | Authentication handled through a central identity/access system         |
| **Local authentication**       | Credentials stored and checked directly on the network device           |

---

# 27. What You Should Remember for CCNA

### The absolute essentials

```text
AAA
│
├── Authentication → WHO?
├── Authorization  → WHAT?
└── Accounting     → WHAT DID THEY DO?
```

### Protocols

```text
RADIUS
→ broad/common AAA protocol

TACACS+
→ strongly associated with Cisco device administration
```

### Scaling

```text
Local accounts
→ okay for small environments
→ difficult at scale

Centralized AAA
→ centralized identity/access
→ easier user lifecycle management
→ better control and accountability
```

### Wireless

```text
802.1X
   +
EAP
   +
AAA infrastructure
   ↓
User-based network authentication
```

---

# 28. The One Mental Model to Keep

If you remember only one diagram from this lesson, remember this:

```text
                  AAA
                   │
        ┌──────────┼──────────┐
        ▼          ▼          ▼
      WHO?       WHAT?      DID WHAT?
        │          │          │
 Authentication Authorization Accounting
        │          │          │
        └──────────┼──────────┘
                   ▼
          Centralized Control
                   │
          ┌────────┴────────┐
          ▼                 ▼
       RADIUS            TACACS+
          │                 │
          └────────┬────────┘
                   ▼
             Network Access
             /           \
       Administrators    Users
          │                │
        SSH            802.1X/EAP
```

## July 31 takeaway

**Cisco AAA is the framework that moves network security from "every device has a password" to centralized identity and access control.**

The progression is:

**Authentication → identify the user → Authorization → determine privileges → Accounting → record activity.**

And the broader lesson is that AAA can protect both **network-device administration** and **user network access**, which is why it becomes important as a network grows.
