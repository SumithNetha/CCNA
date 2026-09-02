# Skill 18 — Lesson 04

# Deploying Access Control at Castle Rysen

> **Core idea:** ACLs become useful when they are translated from business/security requirements into precise traffic policies. The important skill is not memorizing ACL syntax; it is determining **who can communicate with whom, using which protocols/ports, and where the policy should be enforced**.

---

## 1. Why ACL Deployment Matters

Knowing how to configure a Standard or Extended ACL is different from deploying one on a real network.

A poorly planned ACL can immediately cause an outage because of:

* Incorrect ACL order
* Incorrect source/destination
* Incorrect wildcard mask
* Incorrect interface
* Incorrect direction
* Forgetting the implicit `deny`
* Blocking traffic that the business still requires

### Basic ACL mindset

Before configuring:

1. Identify the **source**
2. Identify the **destination**
3. Identify the **protocol**
4. Identify the **port**
5. Decide what must be **permitted**
6. Decide what must be **denied**
7. Determine where the ACL should be applied
8. Test both permitted and denied traffic

> **Draw first, configure second.**

---

# 2. Castle Rysen Access-Control Requirements

The Castle Rysen RFP requires the Cafe to separate:

* **Patron devices**
* **Administrative devices**

Administrative devices include:

* Network equipment
* Cameras
* Plex server
* Other infrastructure

Patrons still need access to the Plex server, but only through the required Plex/application ports.

The RFP specifies Plex access as:

| Protocol | Destination Ports |
| -------- | ----------------- |
| TCP      | `443`             |
| TCP      | `32400`           |
| TCP      | `32469`           |
| UDP      | `1900`            |
| UDP      | `5353`            |

The RFP also requires remote device management to be restricted to authorized management networks. 

---

# 3. ACL Traffic-Flow Design

The Cafe patron network needs three logical rules.

### Rule 1 — Permit Plex

Allow:

```text
Patron VLAN → Admin VLAN → Plex server
```

but **only** for the required Plex ports.

### Rule 2 — Deny other Patron → Admin traffic

After the Plex permits:

```text
Patron VLAN → Admin VLAN → DENY
```

This prevents patrons from reaching:

* Cameras
* Network equipment
* Other administrative devices
* Other services
* Management interfaces

### Rule 3 — Permit everything else

Finally:

```text
permit ip any any
```

This allows patrons to continue reaching destinations outside the protected Admin VLAN, such as the Internet.

### Result

Conceptually:

```text
                 ┌── TCP 443 ────────┐
                 ├── TCP 32400 ──────┤
Patron VLAN ─────┼── TCP 32469 ──────┼──> Plex Server
                 ├── UDP 1900 ───────┤
                 └── UDP 5353 ───────┘
                         │
                         │
                 Other Admin Traffic
                         │
                         X
                       DENY

Patron VLAN ────────────────> Internet
                              ALLOW
```

---

# 4. ACL Processing Order Is Critical

ACL entries are processed **top-to-bottom**.

The first matching ACE determines what happens to the packet.

Therefore:

```text
PERMIT specific Plex traffic
DENY other Patron → Admin traffic
PERMIT everything else
```

is fundamentally different from:

```text
DENY Patron → Admin
PERMIT Plex
```

The second version never reaches the Plex permit because the broader deny matches first.

### Remember

> **Specific permit → broader deny → general permit**

This is one of the most important practical ACL concepts.

---

# 5. Extended ACL — `cafe_filter`

The lesson creates an Extended ACL named:

```text
cafe_filter
```

Its logic is:

```text
Source = Patron subnet
Destination = Admin subnet
Protocol = TCP/UDP
Ports = Required Plex ports
```

Then:

```text
deny ip <patron-subnet> <admin-subnet>
```

followed by:

```text
permit ip any any
```

The exact addresses depend on the Castle Rysen topology/configuration being used.

---

# 6. Why Extended ACL?

An Extended ACL can inspect considerably more information than a Standard ACL.

It can make decisions based on:

* Source IP
* Destination IP
* Protocol
* Source port
* Destination port

That makes it appropriate for the Plex requirement.

For example, the policy isn't simply:

> "Allow this subnet."

It is:

> "Allow this subnet to reach this other subnet **only for these specific services**."

That requires the additional matching capability of an Extended ACL.

---

# 7. Source Port vs Destination Port

This is an important point from the lesson.

When allowing access to an application/service, the application normally listens on a **destination port**.

For example:

```text
Patron → Plex
             ↓
       destination port
          TCP 32400
```

The client normally uses a dynamically selected source port.

Therefore, when creating an ACL to permit access to a server application, think:

```text
Source = client subnet
Destination = server/subnet
Destination port = application port
```

### Common mistake

Don't accidentally put:

```text
32400
```

on the source-port side when what you actually want is:

```text
destination port 32400
```

---

# 8. Where Should the Extended ACL Be Applied?

The lesson follows the principle:

> **Extended ACLs should generally be placed as close to the source as possible.**

The reason is efficiency and security.

If unwanted traffic originates from the Patron VLAN, there is little value in allowing it to travel through the network before finally dropping it.

Instead:

```text
Patron
   ↓
Patron VLAN
   ↓
ACL ← STOP unwanted traffic here
   ↓
Routing
   ↓
Destination
```

The ACL is applied **inbound** on the Cafe router's Patron VLAN subinterface.

### How to think about direction

Don't memorize:

> "Use inbound."

Instead ask:

> **Where is the packet entering the router?**

If the packet enters through the Patron VLAN subinterface:

```text
Patron → Router
         ↑
       inbound
```

Therefore the ACL should inspect it inbound on that interface.

---

# 9. Standard ACL vs Extended ACL Placement

| ACL Type     | Typical Placement    | Reason                                                       |
| ------------ | -------------------- | ------------------------------------------------------------ |
| Standard ACL | Close to destination | Matches primarily source address                             |
| Extended ACL | Close to source      | Can precisely filter source, destination, protocol and ports |

### Easy memory rule

```text
Standard  → Near Destination
Extended  → Near Source
```

This is a **general design guideline**, not an absolute rule.

---

# 10. Implicit Deny

Every ACL has an implicit deny at the end.

Conceptually:

```text
permit ...
permit ...
deny ...
```

Even if you don't type the final deny, it exists.

This is why blindly creating an ACL can break connectivity.

For the Castle Rysen Patron ACL, the explicit:

```text
permit ip any any
```

is important because the design wants to restrict Patron → Admin traffic while still allowing other traffic, such as Internet access.

### Without it

You could unintentionally create:

```text
Plex → ALLOW
Admin → DENY
Everything else → implicit DENY
```

That would also kill Internet access.

---

# 11. Ping Can Fail Even When the Application Works

An important testing lesson:

> **Server reachability and application reachability are not the same thing.**

Suppose the ACL permits only:

```text
TCP 443
TCP 32400
TCP 32469
UDP 1900
UDP 5353
```

but doesn't permit ICMP.

Then:

```text
ping Plex-server
```

can fail.

That does **not necessarily mean Plex is broken**.

The ACL may simply be doing exactly what it was designed to do.

### Example

```text
ICMP
Patron ───────────────X──> Plex

TCP 32400
Patron ──────────────────> Plex
                              ✓
```

Therefore, application-specific testing is important.

---

# 12. Testing ACLs

After applying an ACL, test from both sides of the policy.

### Test allowed traffic

Verify that Patron devices can reach the Plex service using the permitted ports.

### Test denied traffic

Verify that Patron devices cannot reach:

* Administrative devices
* Cameras
* Network management interfaces
* Other services on the Admin VLAN

### Test Internet access

Make sure:

```text
Patron → Internet
```

still works.

### Test management access

Verify that unauthorized networks cannot establish:

```text
SSH
Telnet
```

sessions to network devices.

---

# 13. Telnet as a Connectivity Test

The lesson mentions using Telnet as a quick port-connectivity test.

Conceptually:

```text
telnet <IP> <port>
```

If the connection succeeds, you know that something is reachable at that IP/port and that the traffic isn't being blocked along that path.

It is an old-school troubleshooting technique, but it can still be useful for testing TCP connectivity.

---

# 14. ACL Logging

ACL entries can also use the:

```text
log
```

keyword.

This provides visibility into matching traffic.

For example, an ACL can help answer:

> "Which hosts are actually matching this policy?"

This makes ACLs useful not only for **traffic filtering**, but also for **troubleshooting and investigation**.

### Important limitation

ACL logging is not a replacement for packet capture.

It provides useful information about matching ACL traffic, but it isn't equivalent to analyzing packets with a full packet analyzer.

---

# 15. Protecting SSH and Telnet Access

The second major requirement is protecting the network devices themselves.

Cisco devices provide **VTY lines** for remote terminal access.

These lines handle:

* SSH
* Telnet

Instead of filtering management access somewhere else in the network, the lesson applies an ACL directly to the VTY lines.

---

# 16. `access-class`

The mechanism used is:

```text
access-class
```

under the VTY configuration.

Conceptually:

```text
Remote Host
     │
     │ SSH/Telnet
     ↓
  VTY Lines
     │
     ↓
Standard ACL
     │
 ┌───┴────┐
 │        │
Allowed  Denied
```

The ACL checks the **source IP address** of the device attempting remote management.

---

# 17. Standard ACL for Management Access

The management policy only permits approved management networks.

### Cafe

Allow:

```text
Admin VLAN
```

### Fallout Shelter

Allow:

```text
Management VLAN
```

Everyone else:

```text
DENY
```

This matches the Castle Rysen requirement that administrative access to network devices be limited to the appropriate management segments. 

---

# 18. Why a Standard ACL for VTY Access?

For VTY access, the question is primarily:

> **Who is allowed to manage this device?**

The decision is based on the **source IP/subnet**.

You don't necessarily need to build a complex traffic policy involving:

```text
source
destination
protocol
destination port
```

The VTY access control itself is the target.

Therefore, a Standard ACL is a clean solution.

### Key distinction

**Extended ACL:**

```text
Who → Where → What protocol → Which port?
```

**VTY Standard ACL:**

```text
Who is allowed to remotely manage this device?
```

---

# 19. Two Different ACL Jobs

Castle Rysen demonstrates two distinct uses of ACLs.

| Requirement                       | ACL          | Applied Where         | Purpose                      |
| --------------------------------- | ------------ | --------------------- | ---------------------------- |
| Patron → Admin/Plex filtering     | Extended ACL | Patron VLAN interface | Control routed traffic       |
| SSH/Telnet management restriction | Standard ACL | VTY lines             | Control remote device access |

This distinction is extremely important.

---

# 20. Real-World Deployment Workflow

A good ACL deployment process is:

```text
1. Gather requirements
        ↓
2. Identify source
        ↓
3. Identify destination
        ↓
4. Identify required services/ports
        ↓
5. Define permitted traffic
        ↓
6. Define denied traffic
        ↓
7. Consider implicit deny
        ↓
8. Determine ACL type
        ↓
9. Choose placement
        ↓
10. Choose direction
        ↓
11. Configure
        ↓
12. Verify
        ↓
13. Test allowed traffic
        ↓
14. Test denied traffic
        ↓
15. Monitor/log
```

---

# 21. Maintenance Window

ACL changes should ideally be performed during an appropriate maintenance window.

Why?

Because a single mistake can immediately cause:

```text
Users disconnected
        ↓
Services unavailable
        ↓
Business outage
```

Potential mistakes include:

* Wrong subnet
* Wrong wildcard mask
* Wrong port
* Wrong interface
* Wrong ACL direction
* Wrong ACL sequence/order
* Missing permit statement

### Recommended approach

**Plan → Configure → Verify → Test → Monitor**

---

# 22. Most Important Concepts for CCNA

### ACL logic

```text
Top → Bottom
First match wins
Implicit deny at the end
```

### Extended ACL

Can match:

```text
Source
Destination
Protocol
Ports
```

### Standard ACL

Primarily controls based on:

```text
Source IP
```

### Placement

```text
Standard → generally near destination
Extended → generally near source
```

### Interface direction

Ask:

> **Where does the packet enter the router?**

### Application filtering

Application ports are normally matched as **destination ports** when allowing access to a server service.

### VTY protection

Use:

```text
access-class
```

to restrict who can remotely access device VTY lines.

---

# 23. Castle Rysen Policy — One-Page Summary

```text
                    CASTLE RYSEN CAFE
                           │
                    ┌──────┴──────┐
                    │             │
              Patron VLAN      Admin VLAN
                    │             │
                    │       ┌─────┼─────────┐
                    │       │     │         │
                    │     Plex  Cameras   Network
                    │     Server           Devices
                    │
                    │
             Extended ACL
                    │
        ┌───────────┴────────────┐
        │                        │
  Plex required ports       Other Admin traffic
        │                        │
      ALLOW                    DENY
        │
        └──────────────┐
                       │
                    Internet
                      ALLOW
```

And for device management:

```text
Cafe Admin VLAN ──────────┐
                          │
                          ▼
                    VTY access-class
                          │
                          ▼
                    SSH / Telnet
                          │
                    Network Device
                          ▲
                          │
Fallout Management VLAN ──┘
```

---

## 🔑 Final Takeaway

The real lesson is **policy thinking**.

Don't start with:

> "What ACL command do I type?"

Start with:

> **Who is communicating?**
> **Where are they going?**
> **What service are they using?**
> **Should that traffic be allowed?**
> **Where can I enforce the policy most effectively?**

For Castle Rysen:

**Patrons → Plex:** allowed only on required ports
**Patrons → other Admin resources:** denied
**Patrons → Internet:** allowed
**Unauthorized networks → device SSH/Telnet:** denied
**Approved management networks → device SSH/Telnet:** allowed

That is the difference between **knowing ACL syntax** and **designing network access control**. 
