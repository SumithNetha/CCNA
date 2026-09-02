# What Now? — ACLs

This lesson is the **conceptual conclusion of the ACL section**. The main idea is that you should stop thinking of an ACL as merely a security mechanism and start thinking of it as a **traffic-classification tool**.

## 1. The Core Idea

An ACL is fundamentally a list of:

```text
permit
deny
```

statements used to determine whether traffic, devices, or connections **match a particular condition**.

The important part is what happens **after the match**.

> **ACL = classify/match traffic → another feature acts on that traffic**

So:

```text
Identify what you care about
        ↓
Create ACL to match it
        ↓
Apply ACL to a feature
        ↓
Feature changes behavior
```

This is the major mindset shift from the lesson.

---

# 2. ACL ≠ Just Blocking Traffic

A common beginner interpretation is:

```text
ACL
 ↓
Security
 ↓
Block unwanted traffic
```

That's only one application.

The lesson wants you to think:

```text
ACL
 ↓
Classification
 ↓
"Which traffic/devices are we talking about?"
 ↓
Another feature takes action
```

The ACL itself is essentially providing the **matching logic**.

---

# 3. Where ACLs Are Used

The lesson gives several examples.

### Security / Traffic Filtering

You can use an ACL to determine:

```text
Who can communicate?
What traffic is allowed?
What traffic is denied?
```

This is the traditional ACL use case you practiced at Castle Rysen.

---

### QoS

ACLs can identify traffic that should receive different treatment.

For example:

```text
Voice traffic
     ↓
ACL identifies it
     ↓
QoS gives it priority
```

The ACL isn't necessarily saying:

> "Block everything except voice."

Instead, it's saying:

> "This is the traffic I'm interested in."

QoS then determines what happens to that traffic.

---

### VPN

ACLs can identify traffic that should be encrypted and sent through a VPN tunnel.

Conceptually:

```text
Interesting traffic
       ↓
      ACL
       ↓
VPN identifies it
       ↓
Encrypt / tunnel traffic
```

Again, the ACL is functioning as a **classifier**.

---

### Remote Administration

This is what you implemented at Castle Rysen.

You created:

```cisco
ip access-list standard ADMIN_LIMIT
 permit 10.0.18.0 0.0.0.31
 permit 10.0.16.0 0.0.0.127
```

Then applied it to the VTY lines:

```cisco
line vty 0 4
 access-class ADMIN_LIMIT in
```

Conceptually:

```text
Management source
       ↓
 ADMIN_LIMIT
       ↓
Does source match?
       ↓
   Yes → VTY access
   No  → denied
```

So the ACL is identifying **which source devices are allowed to administer the device**.

---

# 4. NetworkChuck Coffee Example

Imagine a network containing:

* Point-of-sale systems
* Guest Wi-Fi
* Security cameras
* Management laptops

You don't necessarily want all of them treated identically.

For example:

```text
POS systems
   ↓
Priority treatment

Management subnet
   ↓
Allowed to SSH into routers

Branch-to-branch traffic
   ↓
Encrypted through VPN

Guest traffic
   ↓
Restricted access
```

Different requirements can use the same fundamental mechanism:

```text
ACL → identify traffic
```

The feature using the ACL determines what happens next.

---

# 5. The Four-Step Pattern

This is probably the **most important thing to remember from this lesson**.

### Step 1 — Identify

Determine what traffic or devices you care about.

Example:

```text
10.0.18.0/27
```

---

### Step 2 — Match

Create an ACL that identifies that traffic.

```cisco
permit 10.0.18.0 0.0.0.31
```

---

### Step 3 — Apply

Attach the ACL to the appropriate feature.

For example:

```cisco
line vty 0 4
 access-class ADMIN_LIMIT in
```

---

### Step 4 — Change behavior

The feature takes action based on the ACL match.

For VTY:

```text
Allowed source → remote administration
Denied source  → remote administration blocked
```

The reusable pattern is:

```text
IDENTIFY
   ↓
MATCH
   ↓
APPLY
   ↓
CHANGE BEHAVIOR
```

---

# 6. Why This Matters for CCNA

The lesson's bigger point is that ACLs shouldn't remain an isolated CCNA topic.

As you encounter more Cisco features, you should start asking:

> **"What traffic am I matching?"**

For example:

```text
ACL + filtering
ACL + QoS
ACL + VPN
ACL + VTY
```

The underlying thought process is similar:

```text
What am I interested in?
        ↓
How do I identify it?
        ↓
What feature uses that identification?
        ↓
What behavior results?
```

This makes future configurations easier to understand because you're recognizing a **reusable design pattern**, rather than memorizing unrelated commands.

---

# 7. Syntax vs Concept

The lesson makes an important distinction:

### Syntax can be difficult

ACL syntax is very particular.

A small mistake in:

* wildcard mask
* source/destination
* protocol
* port
* rule order

can produce unexpected behavior.

For example, during your Castle Rysen lab, you initially entered:

```cisco
permit tcp 10.0.18.32 0.0.0.31 10.0.18.0 0.0.0.31 443
```

and IOS required the port operator:

```cisco
eq 443
```

So the correct syntax became:

```cisco
permit tcp 10.0.18.32 0.0.0.31 10.0.18.0 0.0.0.31 eq 443
```

But the syntax difficulty doesn't mean the underlying concept is difficult.

The conceptual operation is simply:

```text
Match this traffic.
```

---

# 8. The Real-World Mindset

The lesson's strongest practical takeaway is:

> Don't start by thinking, **"I need an ACL."**

Instead think:

> **"I need to match this traffic so I can do something with it."**

For example:

```text
"I need to prioritize voice."
        ↓
What traffic is voice?
        ↓
Match it
        ↓
QoS acts on it
```

Or:

```text
"I need only administrators to SSH."
        ↓
What addresses are administrators?
        ↓
Match them
        ↓
VTY access control acts on them
```

Or:

```text
"I need this traffic encrypted."
        ↓
What traffic should enter the VPN?
        ↓
Match it
        ↓
VPN acts on it
```

---

# 9. The Mental Model to Keep

You can reduce the entire lesson to this:

```text
              ACL
               │
        ┌──────┴──────┐
        │   MATCH     │
        │ traffic /   │
        │   devices   │
        └──────┬──────┘
               │
               ▼
       Another feature
               │
       ┌───────┼────────┐
       ▼       ▼        ▼
      QoS     VPN      VTY
       │       │        │
    Priority Encrypt  Manage
```

So the key word is:

# **CLASSIFY**

Not merely:

# **BLOCK**

---

## CCNA Takeaways

| Concept                 | Remember                                        |
| ----------------------- | ----------------------------------------------- |
| ACL                     | List of permit/deny statements                  |
| Primary conceptual role | **Classify/match traffic or devices**           |
| ACL itself              | Doesn't necessarily perform the final action    |
| Filtering               | One application                                 |
| QoS                     | ACL can identify traffic for priority treatment |
| VPN                     | ACL can identify traffic to encrypt/tunnel      |
| VTY                     | ACL can restrict remote administration          |
| Core workflow           | **Identify → Match → Apply → Change behavior**  |
| Real-world question     | **"What traffic am I matching?"**               |

### One-sentence summary

**An ACL is a reusable classification mechanism: identify the traffic or devices you care about, match them with an ACL, apply that ACL to a feature, and let that feature determine what happens next.**
