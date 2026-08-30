# Week 13 — July 27

## Skill 18 Lesson 02 — Standard ACL Configuration

This lesson is about **building, modifying, applying, and safely managing Standard ACLs**. The central idea is that Standard ACLs are simple because they make decisions primarily from the **source IP address**, but that simplicity makes placement and the **implicit deny** especially important.

---

## 1. What is a Standard ACL?

A **Standard Access Control List (ACL)** is a set of rules used by a router to decide whether traffic should be **permitted or denied**.

The key limitation:

> **Standard ACLs filter traffic based only on the source IP address.**

They cannot distinguish traffic based on:

* Destination IP
* TCP/UDP port
* Protocol

So a Standard ACL is relatively **blunt** compared with an Extended ACL.

---

## 2. The Most Important Rule: Implicit Deny

Every ACL effectively ends with:

```text
deny any
```

even if you never type it.

For example:

```cisco
ip access-list standard PC1-FILTER
 deny host 10.0.0.10
```

This does **not** mean:

> "Deny PC1 and allow everyone else."

It effectively means:

```text
deny host 10.0.0.10
deny everything else
```

Therefore, if your intention is:

> Block PC1 but allow everyone else

you need:

```cisco
ip access-list standard PC1-FILTER
 deny host 10.0.0.10
 permit any
```

### Think of it like this

```text
Traffic
   |
   v
Is source 10.0.0.10?
   |
   +---- YES ---> DENY
   |
   +---- NO ----> PERMIT
```

The `permit any` is what prevents the implicit deny from blocking everyone else.

---

## 3. Why ACL Cleanup Matters

One of the biggest lessons here isn't even an ACL command.

It is **change management**.

Before deleting or rebuilding an ACL, first determine:

> **Where is this ACL currently applied?**

An ACL can be applied to things such as:

* Interfaces
* VTY lines
* Other features that reference the ACL

The safe approach is:

```text
1. Find where the ACL is applied
        ↓
2. Remove the ACL from that location
        ↓
3. Delete or modify the ACL
```

### Why?

Suppose an ACL is still attached to an interface.

You delete its entries and later create a new rule using the same ACL number/name.

That ACL can immediately begin filtering traffic because **the ACL is still attached to the interface**.

This is why the lesson describes ACLs as policies "with teeth."

---

## 4. Named Standard ACLs

The lesson uses a **named ACL** rather than the older numbered approach.

Example:

```cisco
ip access-list standard PC1-FILTER
```

Then enter the ACL configuration mode:

```cisco
(config-std-nacl)#
```

You can then create rules:

```cisco
deny host 10.0.0.10
permit any
```

### Why use names?

Compare:

```text
access-list 50
```

with:

```text
PC1-FILTER
```

The second immediately communicates the purpose.

Good ACL names make troubleshooting and maintenance easier.

---

## 5. Matching a Single Host

The lesson shows two ways to identify one specific host.

### Method 1 — `host`

```cisco
deny host 10.0.0.10
```

This explicitly means:

> Match exactly this host.

### Method 2 — wildcard mask

```cisco
deny 10.0.0.10 0.0.0.0
```

A wildcard mask of:

```text
0.0.0.0
```

means every bit must match.

So both represent the same single-host match.

Cisco may display the second form as:

```text
host 10.0.0.10
```

which is easier to read.

---

## 6. Building the PC1 Filter

The practical example is:

> **Block PC1 while allowing everyone else.**

Suppose PC1 has:

```text
10.0.0.10
```

Create the named Standard ACL:

```cisco
ip access-list standard PC1-FILTER
```

Deny PC1:

```cisco
deny host 10.0.0.10
```

Then allow everyone else:

```cisco
permit any
```

The resulting logic is:

```text
Source IP = PC1
      ↓
    DENY

Any other source
      ↓
   PERMIT
```

---

## 7. Sequence Numbers

Named ACLs can contain **sequence numbers**.

Conceptually:

```text
10 deny host 10.0.0.10
20 permit any
```

The numbers represent the position of each ACL entry.

This is useful because you can modify individual rules instead of rebuilding the entire ACL.

For example, if you want to remove sequence 10:

```cisco
no 10
```

You can then add or modify specific entries without destroying the rest of the ACL.

---

## 8. ACL Remarks

ACLs can also contain **remarks**.

A remark explains why a rule exists.

For example:

```cisco
remark Block compromised PC1
deny host 10.0.0.10
```

The important idea isn't the exact wording.

It is:

> **Document the reason behind the security rule.**

Six months later, someone looking at:

```text
deny host 10.0.0.10
```

might have no idea why that host was blocked.

A useful remark provides context.

---

## 9. Creating an ACL ≠ Applying an ACL

This is one of the most important concepts.

Creating:

```cisco
ip access-list standard PC1-FILTER
```

does **not** automatically filter traffic.

The ACL must be **applied** somewhere.

For an interface, the command is:

```cisco
ip access-group PC1-FILTER in
```

or:

```cisco
ip access-group PC1-FILTER out
```

Until the ACL is applied, it is simply a configured policy.

---

## 10. Inbound vs Outbound

This is where you should stop thinking like the PC and start thinking like the **router**.

Ask:

> **From the router's perspective, is the packet entering this interface or leaving this interface?**

### Inbound

```text
Packet
  ↓
[ Interface ]
  ↓
 ROUTER
```

The packet is entering the router through that interface.

Use:

```cisco
ip access-group ACL-NAME in
```

### Outbound

```text
ROUTER
  ↓
[ Interface ]
  ↓
 Packet
```

The packet is leaving the router through that interface.

Use:

```cisco
ip access-group ACL-NAME out
```

### The rule

Don't ask:

> "Is the PC sending?"

Ask:

> **"Is the packet entering or leaving the router interface?"**

That distinction prevents a lot of ACL mistakes.

---

## 11. The Practical PC1 Example

In the NetworkChuck Coffee lab, the ACL was applied to the **subinterface connected to PC1's subnet**.

Traffic from PC1 was entering the router through that interface.

Therefore:

```cisco
ip access-group PC1-FILTER in
```

The result:

```text
PC1
 |
 | traffic
 v
Router interface
 |
 | ACL checks source
 v
DENY
```

The ping failed immediately.

When the ACL was removed from the interface, connectivity returned.

That is a good demonstration of the fact that:

> **ACLs take effect immediately once applied.**

---

## 12. Standard ACL Placement

This is probably the most important design principle from the lesson.

Because Standard ACLs only understand the **source address**, they should generally be placed:

> **As close to the destination as possible.**

### Why?

Imagine:

```text
PC1
 |
Router
 |
 +---- Server A
 |
 +---- Server B
 |
 +---- Server C
```

Suppose you want to prevent PC1 from reaching **Server C**.

A Standard ACL only knows:

```text
Source = PC1
```

It doesn't know:

```text
Destination = Server C
```

If you place the ACL too close to PC1:

```text
PC1
 |
[ACL]
 |
Router
 +---- Server A
 +---- Server B
 +---- Server C
```

you could prevent PC1 from reaching **all destinations**.

Instead, place the Standard ACL closer to the destination you're protecting.

```text
PC1
 |
Router
 |
 +---- Server A
 |
 +---- Server B
 |
 +---- [ACL] ---> Server C
```

This minimizes unnecessary blocking.

---

## 13. Standard ACL vs Extended ACL

| Feature             | Standard ACL     | Extended ACL |
| ------------------- | ---------------- | ------------ |
| Source IP           | ✅                | ✅            |
| Destination IP      | ❌                | ✅            |
| Protocol            | ❌                | ✅            |
| TCP/UDP ports       | ❌                | ✅            |
| Filtering precision | Lower            | Higher       |
| Typical placement   | Near destination | Near source  |

The key distinction:

**Standard ACL**

```text
WHO?
```

**Extended ACL**

```text
WHO?
   +
WHERE?
   +
WHAT PROTOCOL?
   +
WHICH PORT?
```

---

## 14. The ACL Processing Mental Model

When traffic reaches an ACL, think:

```text
Packet arrives
     ↓
Read ACL top → bottom
     ↓
First matching rule
     ↓
PERMIT or DENY
     ↓
If nothing matches
     ↓
Implicit DENY
```

### Important:

ACL processing is **top-down**.

Therefore, rule order matters.

For example:

```cisco
deny host 10.0.0.10
permit any
```

works as intended.

But if you put:

```cisco
permit any
deny host 10.0.0.10
```

the second rule is effectively useless because every packet already matches:

```text
permit any
```

This is why ACL design requires careful ordering.

---

## 15. Commands to Remember

### Create named Standard ACL

```cisco
ip access-list standard PC1-FILTER
```

### Deny one host

```cisco
deny host 10.0.0.10
```

### Allow everyone else

```cisco
permit any
```

### Apply inbound

```cisco
ip access-group PC1-FILTER in
```

### Apply outbound

```cisco
ip access-group PC1-FILTER out
```

### Remove an ACL entry by sequence number

```cisco
no <sequence-number>
```

### Remove ACL from interface

```cisco
no ip access-group PC1-FILTER in
```

---

## 16. Verification Mindset

After configuring an ACL, don't assume it works.

Test it.

For the lab:

```text
Before ACL
PC1 ---> destination
       SUCCESS

Apply ACL
PC1 ---> destination
       DENIED

Remove ACL
PC1 ---> destination
       SUCCESS
```

That gives you a clear cause-and-effect relationship.

When troubleshooting ACLs, always ask:

1. **Does the ACL exist?**
2. **What entries are in it?**
3. **Where is it applied?**
4. **Is it inbound or outbound?**
5. **What is the source IP of the traffic?**
6. **Which ACL entry matches first?**
7. **Could the implicit deny be blocking the traffic?**

---

## 17. What You Should Remember for CCNA

If you only remember **8 things** from this lesson:

### 1. Standard ACL = source IP only

```text
SOURCE
  ↓
ACL
```

### 2. ACLs do nothing until applied

```text
Create ACL ≠ Apply ACL
```

### 3. Every ACL has an implicit deny

```text
...
deny any
```

### 4. `permit any` matters

If you deny one host but want everyone else to work:

```cisco
deny host X.X.X.X
permit any
```

### 5. ACLs process top-to-bottom

**First match wins.**

### 6. Think like the router

`in` = packet entering the router interface.

`out` = packet leaving the router interface.

### 7. Standard ACLs are generally placed near the destination

Because they cannot identify the destination.

### 8. Check where an ACL is applied before modifying it

An ACL left attached to an interface can become dangerous when its entries are later changed.

---

## Connection to Your Castle Rysen Project

This lesson is directly building toward the security requirements in your RFP.

Castle Rysen requires separate network segments for administrative and guest traffic, and specifically requires access restrictions between those segments. The RFP also calls for ACL implementation as part of the network security design. 

However, the **Standard ACL** limitation becomes important here: because it can only match the source, it isn't sufficient when you need rules such as:

```text
Guest
  ↓
Plex Server
  ↓
ALLOW TCP 443
ALLOW TCP 32400
ALLOW TCP 32469
ALLOW UDP 1900
ALLOW UDP 5353
```

Those requirements need the additional matching capabilities of **Extended ACLs**, which is exactly why your next lesson is **Extended ACL Configuration**. The RFP explicitly specifies those Plex ports and requires SSH/Telnet access to network devices to be restricted to the appropriate administrative/management VLANs. 

### Your mental progression

```text
STANDARD ACL
     │
     │  "WHO?"
     │
     ▼
Source IP only
     │
     ▼
Good for simple source filtering
     │
     ▼
EXTENDED ACL
     │
     │  "WHO + WHERE + WHAT?"
     │
     ▼
Source + Destination + Protocol + Port
```

**July 27 takeaway:** Standard ACLs are easy to configure, but the real skill is understanding **implicit deny, rule order, application direction, placement, and the limitations of source-only filtering**.
