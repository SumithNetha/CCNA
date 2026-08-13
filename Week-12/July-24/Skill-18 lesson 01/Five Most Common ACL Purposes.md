# Skill 18 — Access Control Lists (ACLs)

## Lesson 01: Five Most Common ACL Purposes

This lesson builds on the previous idea: **an ACL is a tool, not the action itself**. The meaning of `permit` and `deny` depends on the network feature that uses the ACL.

---

## 1. What an ACL Contains

At its simplest, an ACL is a sequence of:

```text
permit
deny
permit
deny
...
```

The ACL defines **matching rules**. Another feature uses those rules to decide what to do.

### Standard ACL

A **standard ACL** matches only the:

> **Source IP address**

So it answers:

```text
"Where did this traffic come from?"
```

It is relatively coarse-grained.

### Extended ACL

An **extended ACL** provides much more precise matching.

It can match:

* Source IP
* Destination IP
* Protocol
* Port number

For example, conceptually:

```text
Source → Destination → Protocol → Port
```

This allows you to distinguish between different types of traffic rather than simply identifying the source.

**Key idea:** Extended ACLs are generally more useful for detailed traffic filtering because they provide greater precision.

---

# 2. ACL Processing Rules

This is one of the most important parts of the lesson.

ACLs are processed:

1. **Top-down**
2. **One statement at a time**
3. **First match wins**

The router does **not** search the entire ACL and choose the "best" matching statement.

Example:

```text
10.10.10.50 → permit
10.10.10.0/24 → deny
```

Traffic from `10.10.10.50` matches the first statement:

```text
10.10.10.50 → PERMIT
```

The router stops processing the ACL.

It never reaches the second statement.

### Why order matters

If you reverse them:

```text
10.10.10.0/24 → deny
10.10.10.50 → permit
```

The specific host is already included in the `/24` network.

Therefore:

```text
10.10.10.50
      ↓
matches 10.10.10.0/24
      ↓
DENY
      ↓
STOP
```

The later permit is never reached.

### Remember:

> **First match wins.**

---

# 3. Implicit Deny

Every ACL has an invisible **implicit deny** at the bottom.

Conceptually:

```text
ACL
│
├── Statement 1
├── Statement 2
├── Statement 3
├── ...
└── implicit deny
```

If traffic doesn't match a permit statement, it is denied by default.

You don't manually type this rule.

Therefore:

> **If nothing matches, the traffic loses by default.**

This is one of the most important troubleshooting concepts in ACLs.

### Troubleshooting order

When ACL traffic unexpectedly fails, check:

1. **ACL statement order**
2. **Implicit deny**

---

# 4. Five Common ACL Purposes

The lesson identifies five major purposes:

| # | Purpose                | What the ACL helps identify/control       |
| - | ---------------------- | ----------------------------------------- |
| 1 | **Traffic Filtering**  | Which traffic is allowed or dropped       |
| 2 | **NAT**                | Which traffic is eligible for translation |
| 3 | **Route Filtering**    | Which routing information is shared       |
| 4 | **QoS**                | Which traffic receives priority/treatment |
| 5 | **Security Functions** | Who can access network services/devices   |

The important part is that **the same ACL concept has different meanings depending on where it is applied.**

---

# 5. Traffic Filtering

This is the use most people associate with ACLs.

An ACL can be applied to a router interface:

```text
Inbound
```

or:

```text
Outbound
```

The ACL determines whether matching traffic is permitted or dropped.

Conceptually:

```text
Traffic
   ↓
Interface
   ↓
ACL
   ↓
Permit → Continue
Deny   → Drop
```

This is where ACLs can feel similar to a firewall.

### Why Extended ACLs are useful here

Traffic filtering often requires detailed control:

```text
Source
   +
Destination
   +
Protocol
   +
Port
```

That's why extended ACLs are particularly useful for this purpose.

---

# 6. NAT

This is an important distinction.

When an ACL is used with **NAT**, `permit` and `deny` do **not** mean:

```text
permit = allowed onto the network
deny   = blocked from the network
```

Instead, they determine:

```text
permit = eligible for NAT translation
deny   = not eligible for NAT translation
```

For example:

```text
Internal Networks
       │
       ▼
      ACL
       │
   ┌───┴────┐
   │        │
Permit     Deny
   │        │
   ▼        ▼
Translate  No Translation
```

This is a crucial distinction.

### Example

Suppose the shop has:

```text
Admin VLAN
Guest VLAN
Camera VLAN
```

An ACL used for NAT might identify only the internal networks that should be translated to the public Internet.

If a device doesn't match the NAT ACL, that does **not necessarily mean the device is being security-blocked**.

It simply means its traffic isn't eligible for translation.

---

# 7. Route Filtering

ACLs can also be used for **route filtering**.

Here, you're not controlling whether a packet can pass through an interface.

Instead, you're controlling **routing information**.

Imagine NetworkChuck Coffee has multiple locations:

```text
Coffee Shop A
      │
      ├──────── Partner Network
      │
Coffee Shop B
```

You might want to advertise only specific networks.

Conceptually:

```text
Routes
  ↓
ACL
  ↓
Allowed routes → Advertise
Denied routes  → Don't advertise
```

So:

> **Traffic filtering controls traffic. Route filtering controls routing information.**

That's a major distinction.

---

# 8. Quality of Service — QoS

**QoS (Quality of Service)** is another ACL use that is easy to overlook.

The lesson's example is voice traffic.

Voice is sensitive to:

* Delay
* Network conditions

An ACL can help identify the traffic that should receive special treatment.

Conceptually:

```text
Voice Traffic
      ↓
     ACL
      ↓
Identify
      ↓
QoS gives priority
```

The `permit` doesn't necessarily mean:

> "Allow this traffic and block everything else."

Instead, it can mean:

> **"This traffic matches the class that should receive priority."**

So the ACL is being used as a **traffic selector**.

---

# 9. Security Functions

The fifth purpose is broader **security functionality**.

ACLs can help control access to network infrastructure itself.

Examples mentioned in the lesson include:

* SSH access
* Telnet access
* SNMP monitoring
* VPN services

One particularly important application is the **VTY lines**.

VTY lines are the virtual terminal lines used for remote access to a Cisco device.

An ACL can be applied so that only approved networks can remotely manage the device.

Conceptually:

```text
Remote Device
     │
     ▼
   VTY Lines
     │
     ▼
    ACL
   /   \
Allow   Deny
 │       │
 ▼       ▼
Manage  Reject
Device  Access
```

This means ACLs aren't limited to controlling ordinary packet forwarding.

They can also help protect **the management plane of the network device**.

---

# 10. The Same ACL — Different Meaning

This is the central concept of the entire lesson.

Consider:

```text
ACL
```

The ACL itself hasn't fundamentally changed.

But where it is applied changes what `permit` means.

| ACL Application    | `permit` essentially means                 |
| ------------------ | ------------------------------------------ |
| Traffic filtering  | Allow matching traffic                     |
| NAT                | Allow matching traffic to be translated    |
| Route filtering    | Allow matching routing information         |
| QoS                | Identify traffic for the desired treatment |
| Security functions | Allow the specified access                 |

So:

> **The feature using the ACL determines what the ACL's decision means.**

---

# 11. NetworkChuck Coffee Example

Imagine the coffee shop contains:

```text
┌──────────────────────────┐
│     NetworkChuck Coffee  │
├──────────────────────────┤
│ POS Systems              │
│ Office PCs               │
│ Guest Devices            │
│ Monitoring Systems       │
└──────────────────────────┘
```

### Traffic filtering

You could allow POS systems to reach payment services while preventing guest devices from reaching internal resources.

### NAT

You could identify the internal networks that should be translated to the public Internet.

### Route filtering

You could control which networks are advertised between locations or partner networks.

### QoS

You could identify voice traffic so it receives priority.

### Security

You could restrict who can remotely manage network devices.

Same fundamental tool:

```text
ACL
```

Different purpose:

```text
Filtering
NAT
Routing
QoS
Security
```

---

# 12. What You Need to Remember

### ACL fundamentals

```text
ACL = list of matching rules
```

### Standard ACL

```text
Source IP only
```

### Extended ACL

```text
Source IP
Destination IP
Protocol
Port
```

### Processing

```text
Top → Bottom
First match → Stop
```

### Implicit deny

```text
No match → Deny
```

### Five purposes

```text
1. Traffic Filtering
2. NAT
3. Route Filtering
4. QoS
5. Security Functions
```

---

## 🧠 Final Mental Model

```text
                    ACL
                     │
            Matching / Selection
                     │
       ┌─────────────┼─────────────┐
       │             │             │
   Filtering        NAT        Route Filtering
       │             │             │
   Allow/Drop    Translate     Route Sharing
       │
       ├───────────────┐
       │               │
      QoS           Security
       │               │
   Prioritize     Control Access
```

### The sentence to remember

> **An ACL is a list of matching rules, and the network feature that uses it determines what `permit` and `deny` actually mean.**

That concept is more important than memorizing the five categories individually. Once you understand it, **Standard ACLs, Extended ACLs, NAT ACLs, route filtering, QoS classification, and device-access ACLs** become variations of the same fundamental idea.
