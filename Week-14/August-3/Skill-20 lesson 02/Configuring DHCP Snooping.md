# Skill 20 — Lesson 02: Configuring DHCP Snooping

## 1. What DHCP Snooping Is

**DHCP Snooping** is a Cisco Layer 2 security feature that establishes a **trust boundary for DHCP traffic**.

Its primary purpose in this lesson is to prevent **rogue DHCP servers** from sending DHCP offers to clients.

The key rule is:

> **Only interfaces explicitly configured as trusted are allowed to receive legitimate DHCP offer traffic.**

All other switch interfaces are treated as **untrusted by default**.

---

# 2. Why DHCP Needs Protection

Normally, a client that needs an IP address broadcasts a DHCP request:

```text
Client
   |
   | DHCP Broadcast
   ↓
Network
   |
   ↓
DHCP Server
   |
   | DHCP Offer
   ↓
Client
```

The problem is that DHCP normally doesn't authenticate the server.

The client essentially accepts a DHCP response from whoever responds.

That creates an opportunity for a **rogue DHCP server**.

---

# 3. Rogue DHCP Server

A rogue DHCP server is an unauthorized device that starts providing DHCP services on the network.

For example:

```text
                 Legitimate DHCP Server
                         |
                         |
                      Switch
                    /       \
                   /         \
             Client A      Rogue Device
                              |
                         Rogue DHCP
```

The rogue device can start answering DHCP requests.

This could happen accidentally:

* Someone connects a home router.
* Someone enables DHCP on an unauthorized device.
* Someone connects incorrectly configured equipment.

Or intentionally:

* An attacker installs a rogue DHCP server.

---

# 4. Why a Rogue DHCP Server Is Dangerous

The attacker doesn't necessarily need to make the network obviously fail.

They can provide **valid-looking IP information**.

For example:

```text
IP Address:      10.0.10.50
Subnet Mask:     255.255.255.0
Default Gateway: 10.0.10.1
DNS Server:      ...
```

Everything might appear normal.

But the attacker could manipulate the **default gateway** so that traffic goes through the attacker's device first.

Conceptually:

```text
Normal:

Client → Legitimate Gateway → Internet


Rogue DHCP:

Client → Attacker → Legitimate Gateway → Internet
             ↑
        Traffic passes here
```

This can create a **Man-in-the-Middle (MITM)** opportunity.

The important lesson is:

> A network can continue working while being compromised.

That makes rogue DHCP particularly dangerous.

---

# 5. What DHCP Snooping Changes

Without DHCP Snooping:

```text
DHCP Offer
    ↓
Any port could potentially deliver it
```

With DHCP Snooping:

```text
                 DHCP Offer
                     ↓
              Is incoming port
                  trusted?
                 /        \
              YES          NO
               ↓            ↓
             Allow         Block
```

This is the central concept of the entire lesson.

---

# 6. Trusted vs Untrusted Ports

This is **the most important concept to understand**.

### Trusted port

A trusted interface is one where legitimate DHCP server/offer traffic is expected.

Usually this is:

* An uplink toward the DHCP server
* An interface toward another switch carrying legitimate DHCP responses

### Untrusted port

An untrusted interface is generally:

* A user-facing access port
* A port where clients connect
* A port where an unauthorized DHCP server could potentially be connected

---

# 7. The Common Mistake

A common incorrect thought is:

> "The client needs DHCP, so I should trust the client's port."

**No.**

Think about the **direction of the DHCP offer**.

The client sends the DHCP request.

The legitimate DHCP server sends the **offer back**.

Therefore, you trust the interface where the **legitimate offer enters the switch**.

### Think like the switch

Ask:

> **"Where should a legitimate DHCP offer come from?"**

Not:

> "Which device needs DHCP?"

That distinction is critical.

---

# 8. Example

Consider:

```text
                   DHCP Server
                       |
                       |
                    G0/1
                     SW1
                    /   \
                 E0/2   E0/3
                  |       |
               Client   Client
```

Suppose:

```text
G0/1 → DHCP server
E0/2 → Client
E0/3 → Client
```

Then:

```text
G0/1 → TRUSTED
E0/2 → UNTRUSTED
E0/3 → UNTRUSTED
```

Why?

Because legitimate DHCP offers enter through `G0/1`.

---

# 9. Multiple Switches

This becomes even more important when DHCP traffic crosses multiple switches.

Example:

```text
DHCP Server
     |
    SW1
     |
     | Uplink
     |
    SW2
     |
     | Uplink
     |
    SW3
     |
   Client
```

The legitimate DHCP offer travels:

```text
DHCP Server
     ↓
SW1
     ↓
SW2
     ↓
SW3
     ↓
Client
```

Therefore, the relevant uplinks receiving legitimate DHCP offers need to be correctly configured as trusted.

If you miss a trust boundary:

```text
DHCP Server
     ↓
SW1
     ↓
SW2
     X ← incorrectly untrusted
     ↓
SW3
     ↓
Client
```

DHCP can break.

---

# 10. DHCP Snooping Configuration

The configuration described in the lesson has two major parts.

## Step 1 — Enable DHCP Snooping Globally

```cisco
ip dhcp snooping
```

This enables the feature on the switch.

---

## Step 2 — Enable It for Specific VLANs

The lesson's example protects:

```text
VLAN 10
VLAN 20
```

Conceptually:

```cisco
ip dhcp snooping vlan 10
ip dhcp snooping vlan 20
```

Now DHCP Snooping is active for those VLANs.

---

# 11. Configure the Trusted Interface

On the interface where legitimate DHCP offers enter:

```cisco
interface g0/1
 ip dhcp snooping trust
```

The important command is:

```cisco
ip dhcp snooping trust
```

This tells the switch:

> DHCP traffic received through this interface is allowed to contain legitimate DHCP server responses.

---

# 12. Typical Configuration

Putting the pieces together:

```cisco
ip dhcp snooping
ip dhcp snooping vlan 10
ip dhcp snooping vlan 20

interface g0/1
 ip dhcp snooping trust
```

All other ports remain untrusted unless explicitly configured otherwise.

### Mental model

```text
DHCP Snooping
      │
      ├── VLAN 10 protected
      ├── VLAN 20 protected
      │
      └── Ports untrusted by default
               │
               └── Only legitimate DHCP path → TRUST
```

---

# 13. Why You Shouldn't Trust Everything

You might be tempted to configure:

```text
Trust every interface
```

just to make DHCP work.

**Don't do that.**

If every port is trusted:

```text
Port 1 → Trusted
Port 2 → Trusted
Port 3 → Trusted
Port 4 → Trusted
...
```

then an attacker can plug a rogue DHCP server into a user-facing port and potentially send DHCP offers.

You've effectively destroyed the trust boundary.

### Correct approach

```text
Client ports → UNTRUSTED

DHCP server path → TRUSTED
```

**Trust intentionally.**

---

# 14. DHCP Snooping Binding Table

DHCP Snooping has another important function.

The switch builds a **DHCP Snooping Binding Table**.

Think of it as the switch keeping a record of DHCP assignments.

It can associate information such as:

```text
MAC Address
     +
IP Address
     +
VLAN
     +
Interface
```

Conceptually:

| MAC          | IP         | VLAN | Interface |
| ------------ | ---------- | ---: | --------- |
| Client MAC A | 10.0.10.50 |   10 | E0/2      |
| Client MAC B | 10.0.10.51 |   10 | E0/3      |

The lesson describes this as the switch **"keeping receipts."**

That's a very useful mental model.

---

# 15. Why the Binding Table Matters

The binding table becomes useful to other Layer 2 security mechanisms.

Suppose:

```text
Legitimate client:

MAC A → 10.0.10.50
```

Later, another device tries to claim:

```text
MAC B → 10.0.10.50
```

Now the switch has information indicating:

```text
Expected:

10.0.10.50
      ↓
    MAC A

Received:

10.0.10.50
      ↓
    MAC B
```

That mismatch can be useful for detecting spoofing.

This leads directly into the next security feature:

> **Dynamic ARP Inspection (DAI)**

---

# 16. DHCP Snooping → DAI

The progression of Skill 20 is deliberate:

```text
Port Security
      ↓
DHCP Snooping
      ↓
Dynamic ARP Inspection
```

Each feature adds another layer of protection.

### Port Security

Controls:

> **Which MAC addresses can use a port?**

### DHCP Snooping

Controls:

> **Which interfaces can provide DHCP server responses?**

And builds:

> **IP ↔ MAC ↔ VLAN ↔ Interface bindings**

### DAI

Can use those bindings to help determine:

> **Does this device actually own this IP address?**

So don't think of DHCP Snooping as an isolated feature.

It creates information that later security mechanisms can use.

---

# 17. NetworkChuck Coffee Example

Consider a Castle Rysen coffee shop:

```text
                    DHCP Server
                         |
                         |
                    Uplink Port
                         |
                    +---------+
                    | Switch  |
                    +---------+
                     /   |   \
                    /    |    \
                 Admin  Patron Patron
                  PC      PC     PC
```

The security policy should conceptually be:

```text
Uplink toward legitimate DHCP source
             ↓
          TRUSTED

Admin/Patron access ports
             ↓
         UNTRUSTED
```

Now someone connects:

```text
Patron Port
    |
Home Router
    |
Rogue DHCP Server
```

DHCP Snooping prevents DHCP server responses from simply being accepted from that untrusted user-facing interface.

This aligns with the Castle Rysen RFP's requirement for **DHCP Snooping as a core Layer 2 security feature**. 

---

# 18. Troubleshooting DHCP Snooping

One of the most useful practical lessons is:

> **If DHCP stops working after enabling snooping, don't immediately assume the DHCP server is broken.**

Trace the DHCP offer path.

### Troubleshooting process

```text
1. Find the legitimate DHCP server
          ↓
2. Determine the path to the client
          ↓
3. Identify interfaces receiving DHCP offers
          ↓
4. Verify those interfaces are trusted
          ↓
5. Check each switch in the path
```

For example:

```text
DHCP Server
    ↓
SW1
    ↓
SW2
    ↓
SW3
    ↓
Client
```

Check:

```text
SW1 uplink → trusted?
SW2 uplink → trusted?
SW3 uplink → trusted?
```

A single missed trust configuration can break DHCP delivery.

---

# 19. Packet Tracer Caveat

The lesson specifically warns that **Packet Tracer can behave strangely with DHCP Snooping and VLANs**.

Therefore:

> Don't change your understanding of real networking simply to accommodate a Packet Tracer quirk.

If Packet Tracer requires unexpected interfaces to be trusted just to make the lab work, distinguish that from the intended real-world design.

### Real-world principle

```text
Only trust interfaces
that legitimately receive
DHCP server responses.
```

Don't turn every interface into a trusted interface merely because the simulator behaves unexpectedly.

---

# 20. DHCP Snooping vs Port Security

You just learned port security, so connect the two.

| Feature                      | Port Security                 | DHCP Snooping              |
| ---------------------------- | ----------------------------- | -------------------------- |
| Layer                        | Layer 2                       | Layer 2                    |
| Primary identity             | MAC address                   | DHCP traffic / bindings    |
| Main purpose                 | Restrict devices/MACs on port | Block rogue DHCP responses |
| Protects against             | Unauthorized devices/MACs     | Rogue DHCP servers         |
| Uses trusted ports?          | No equivalent concept         | **Yes**                    |
| Creates binding information? | No                            | **Yes**                    |
| Useful to DAI?               | Not the primary source        | **Yes**                    |

### Think:

```text
Port Security
"What device belongs here?"

DHCP Snooping
"Who is allowed to provide DHCP?"

DAI
"Does this IP/MAC relationship make sense?"
```

---

# 21. CCNA Memory Points

### DHCP Snooping

**Purpose:**

> Prevent rogue DHCP servers from providing DHCP responses through untrusted ports.

### Trust rule

> **Trust the port where legitimate DHCP offers enter.**

Not:

> Trust the port where clients need DHCP.

### Default concept

```text
Ports = UNTRUSTED
       ↓
Unless explicitly trusted
```

### Configuration

```cisco
ip dhcp snooping
ip dhcp snooping vlan 10
ip dhcp snooping vlan 20

interface g0/1
 ip dhcp snooping trust
```

### Binding table

```text
MAC
 ↓
IP
 ↓
VLAN
 ↓
Interface
```

### Security chain

```text
Port Security
      ↓
DHCP Snooping
      ↓
Binding Table
      ↓
DAI
```

---

# 22. Important Exam Traps

### ❌ Trap 1

> "Trust the port connected to the client because the client uses DHCP."

**Wrong.**

### ✅ Correct

Trust the interface where the **legitimate DHCP offer enters**.

---

### ❌ Trap 2

> "Trust all switch ports."

**Wrong.**

That defeats the purpose of DHCP Snooping.

---

### ❌ Trap 3

> "DHCP Snooping only blocks DHCP requests."

The lesson's central focus is the protection of **DHCP offer/response traffic from unauthorized sources**.

---

### ❌ Trap 4

> "DHCP Snooping is only for blocking rogue DHCP."

It also creates the **DHCP Snooping Binding Table**, which becomes important for additional Layer 2 security mechanisms such as DAI.

---

# 🧠 Final Mental Model

Remember this picture:

```text
                 LEGITIMATE
                DHCP SERVER
                     │
                     │
                 DHCP OFFER
                     │
                     ▼
              ┌─────────────┐
              │   TRUSTED   │
              │    PORT     │
              └──────┬──────┘
                     │
                  SWITCH
                /    |    \
               /     |     \
             PC      PC    Rogue DHCP
             │       │         │
             │       │         │
          UNTRUSTED UNTRUSTED UNTRUSTED
             │       │         │
             └───────┴─────────┘
                     ↓
             Rogue DHCP Offer
                     ↓
                   BLOCK
```

And the **binding table** is the bonus:

```text
DHCP Snooping
      │
      ├── Blocks rogue DHCP responses
      │
      └── Builds binding table
              │
              ├── MAC
              ├── IP
              ├── VLAN
              └── Interface
                    │
                    ▼
                   DAI
```

### The one sentence to memorize

> **DHCP Snooping establishes trusted DHCP paths, blocks unauthorized DHCP server responses on untrusted ports, and builds IP-to-MAC-to-VLAN-to-interface bindings that can support other Layer 2 security features.**
