# Skill 20 — Layer 2 Security

## Lesson 01: Configuring Switch Port Security

> **Core idea:** Port security places boundaries on an access port by controlling **which MAC addresses, and how many MAC addresses, are allowed to use that port.**

---

## 1. Why Port Security Matters

A switch is naturally designed to learn MAC addresses and forward Ethernet frames. By default, an access port is relatively trusting:

* Plug in a PC → it works.
* Plug in another switch → it works.
* Plug in an unauthorized device → it may work.
* Plug in a rogue AP → it may potentially provide an unintended path into the network.

For an enterprise network, this can be dangerous.

### Example

Suppose:

```text
Admin PC
   |
   | E0/3
   |
Switch
```

The policy might be:

> **E0/3 should have exactly one authorized device.**

Without port security, someone could disconnect the PC and connect:

```text
             +-- PC
             |
Switch -- Small Switch
             |
             +-- Laptop
             |
             +-- Rogue Device
```

Port security allows us to establish a restriction on that switch port.

---

# 2. What Is Port Security?

**Port Security** is a Cisco switch feature that restricts the number of **secure MAC addresses** allowed on a switch port.

Conceptually:

```text
Normal switch
    ↓
Learns MAC addresses freely

Port Security
    ↓
MAC learning + restrictions
    ↓
"Only these MAC addresses are allowed here."
```

The important point is:

> **Port security operates using MAC addresses at Layer 2.**

It doesn't authenticate the user. It establishes a MAC-address-based boundary on the switchport.

---

# 3. Default Port-Security Behavior

When port security is enabled, the lesson describes the basic behavior as:

* **Maximum secure MAC addresses:** 1
* The switch can learn the first MAC address it sees.
* That MAC becomes the authorized/secure MAC address.
* A different MAC address appearing afterward causes a violation.

Example:

```text
E0/3
 │
 └── Admin PC
       MAC = AAAA.BBBB.CCCC
```

The switch learns:

```text
E0/3 → AAAA.BBBB.CCCC
```

Later:

```text
E0/3
 │
 └── Unauthorized PC
       MAC = 1111.2222.3333
```

The switch sees:

```text
Expected: AAAA.BBBB.CCCC
Received: 1111.2222.3333
                  ↓
          Port-security violation
```

---

# 4. Port-Security Violation

A **violation** occurs when a device sends traffic through a secured port using a MAC address that isn't permitted by the port-security policy.

The default violation action is:

## `shutdown`

The port is placed into:

```text
err-disabled
```

### What happens?

```text
Unauthorized MAC
       ↓
Violation detected
       ↓
Port-security action
       ↓
Port becomes error-disabled
       ↓
Traffic stops
```

Even reconnecting the original authorized device doesn't automatically bring the port back.

The lesson demonstrates recovering the interface with:

```cisco
interface e0/3
 shutdown
 no shutdown
```

---

# 5. Port-Security Violation Modes

This is an important CCNA topic.

Cisco provides three major violation modes:

| Mode         | Unauthorized traffic | Logging | Port state     |
| ------------ | -------------------- | ------- | -------------- |
| **Protect**  | Dropped              | No      | Stays up       |
| **Restrict** | Dropped              | Yes     | Stays up       |
| **Shutdown** | Dropped              | Yes     | Error-disabled |

### Easy way to remember

```text
PROTECT
↓
Drop quietly

RESTRICT
↓
Drop + report

SHUTDOWN
↓
Drop + report + kill port
```

---

## 6. Protect Mode

With:

```text
protect
```

unauthorized traffic is dropped.

The switch:

* Drops traffic from unauthorized MAC addresses
* Does not log the violation
* Does not shut down the interface

### Behavior

```text
Unauthorized MAC
       ↓
     DROP
       ↓
Port remains UP
```

### Advantage

No interruption to the port.

### Disadvantage

You don't get visibility into the violation through logging.

---

# 7. Restrict Mode

With:

```text
restrict
```

unauthorized traffic is still dropped, but the switch also records/logs the violation.

```text
Unauthorized MAC
       ↓
     DROP
       ↓
Violation logged
       ↓
Port remains UP
```

This provides a useful middle ground.

### Why?

You get:

**Security + visibility + availability**

without immediately taking the port offline.

The lesson specifically presents **restrict** as a practical favorite because you can know that something unusual happened without necessarily taking the legitimate connection down.

---

# 8. Shutdown Mode

This is the **default** violation mode.

```text
Unauthorized MAC
       ↓
Violation
       ↓
Traffic dropped
       ↓
Violation logged
       ↓
Port → error-disabled
```

This is the most aggressive response.

### Advantage

Strong enforcement.

### Disadvantage

Operational impact.

For example, if someone accidentally changes a cable in a remote branch:

```text
Cable moved
   ↓
Different MAC
   ↓
Violation
   ↓
Port shutdown
   ↓
User loses connectivity
   ↓
Support required
```

Therefore, security policy must consider operational requirements as well.

---

# 9. Sticky MAC Addresses

One of the most important concepts in the lesson is **sticky MAC learning**.

Without sticky MAC, learned secure MAC addresses are associated with the running configuration and don't necessarily persist across a reboot.

### Sticky MAC

Sticky learning allows the switch to:

1. Dynamically learn the MAC address.
2. Treat it as a secure MAC.
3. Place the learned address into the configuration so it can persist.

Conceptually:

```text
Device connects
      ↓
Switch learns MAC
      ↓
Sticky learning
      ↓
MAC becomes secure
      ↓
Configuration can preserve it
```

This avoids manually entering every MAC address.

---

# 10. Static vs Sticky Concept

Think about the distinction like this:

### Static secure MAC

You manually specify:

```text
"This exact MAC is allowed."
```

### Sticky secure MAC

You tell the switch:

```text
"Learn the MAC dynamically and remember it."
```

This is particularly useful for access ports where the expected device is known but manually entering MAC addresses for every port would be cumbersome.

---

# 11. Maximum Number of Secure MAC Addresses

The default behavior discussed in the lesson is effectively:

```text
1 port
↓
1 secure MAC
```

But not every access port has only one legitimate device.

### Example: IP Phone + PC

```text
             +-- PC
             |
Switch ------ IP Phone
```

The switchport can legitimately see **two MAC addresses**:

```text
IP Phone MAC
PC MAC
```

Therefore, the maximum number of secure MAC addresses can be increased where the topology requires it.

### Key point

Don't blindly configure:

```text
maximum = 1
```

on every port.

Understand what devices are legitimately expected on that port first.

---

# 12. MAC Address Aging

**Aging** controls how long a learned secure MAC address remains associated with the port before it ages out.

This is useful when the network changes.

For example:

```text
Employee A
   ↓
E0/3
   ↓
Employee moves
   ↓
Old MAC remains
```

Without appropriate aging behavior, stale MAC information can remain associated with the port.

Aging can therefore help environments where:

* Users move desks
* Devices are replaced
* Ports aren't used for long periods
* Endpoint assignments change

---

# 13. Port Security vs Normal MAC Learning

This distinction is fundamental.

### Normal switch MAC learning

```text
Frame arrives
     ↓
Source MAC learned
     ↓
CAM/MAC table updated
     ↓
Switch forwards traffic
```

### Port security

```text
Frame arrives
     ↓
Source MAC examined
     ↓
Is MAC permitted?
    / \
  YES  NO
   ↓    ↓
Forward  Violation
```

So:

> **Port security doesn't replace MAC learning; it puts restrictions around which MAC addresses can be learned/used on a port.**

---

# 14. Real-World NetworkChuck Coffee Example

Imagine the Castle Rysen Coffee admin area:

```text
                 ADMIN VLAN
                     |
                 Access Switch
                     |
        +------------+------------+
        |                         |
      E0/3                      E0/4
        |                         |
    Admin PC                  Admin PC
```

You could establish a policy such as:

```text
E0/3 → one authorized MAC
E0/4 → one authorized MAC
```

Now imagine someone connects a small switch:

```text
                   E0/3
                     |
               Small Switch
                /    |    \
              PC    PC   Rogue AP
```

Port security can detect additional unauthorized MAC addresses and apply the configured violation action.

This supports the Castle Rysen requirement to protect the network from internal threats. The RFP explicitly calls for **Layer 2 security features including port security, DHCP Snooping, and DAI**. 

---

# 15. Port Security and Rogue Access Points

A rogue AP is particularly concerning.

Example:

```text
Corporate Switch
       |
       | Access Port
       |
   Rogue AP
      )))))
   Wireless users
```

An unauthorized AP could potentially provide access to a network that wasn't intended to be exposed through that physical port.

Port security can help by restricting which MAC addresses are permitted on the port.

**Important:** Port security is not a complete rogue-AP detection/prevention system. It is one Layer 2 control that can reduce unauthorized endpoint/device connections.

---

# 16. Port Security Configuration Concepts

A typical Cisco port-security configuration involves:

### Enable port security

```cisco
interface e0/3
 switchport port-security
```

### Set maximum number of MAC addresses

```cisco
switchport port-security maximum 1
```

### Configure a violation mode

For example:

```cisco
switchport port-security violation restrict
```

### Enable sticky learning

```cisco
switchport port-security mac-address sticky
```

A complete example:

```cisco
interface e0/3
 switchport mode access
 switchport port-security
 switchport port-security maximum 1
 switchport port-security mac-address sticky
 switchport port-security violation restrict
```

### What this means

```text
E0/3
 │
 ├── Access port
 ├── Port security enabled
 ├── Maximum 1 secure MAC
 ├── MAC learned using sticky learning
 └── Unauthorized MAC → dropped + logged
```

---

# 17. Verification

After configuring port security, you need to verify what the switch actually learned.

Useful Cisco commands include:

```cisco
show port-security
```

and:

```cisco
show port-security interface e0/3
```

You can also inspect the MAC address table:

```cisco
show mac address-table
```

For an interface-level investigation, you can use:

```cisco
show interfaces e0/3
```

### What you're looking for

You want to determine:

* Is port security enabled?
* What is the maximum secure MAC count?
* How many secure MAC addresses are currently learned?
* What violation mode is configured?
* Has a violation occurred?
* Is the interface operational or error-disabled?

---

# 18. Error-Disabled Recovery

If shutdown mode causes a violation:

```text
Port Security Violation
        ↓
    Error-disabled
```

The lesson demonstrates manually recovering the port:

```cisco
interface e0/3
 shutdown
 no shutdown
```

The important operational lesson is:

> **Don't confuse an error-disabled port with a normal interface-down condition.**

You need to determine **why** the interface became disabled before simply bringing it back up.

Otherwise:

```text
Violation
 ↓
no shutdown
 ↓
Same unauthorized device
 ↓
Another violation
 ↓
Port disabled again
```

---

# 19. Security vs Operations

A major real-world lesson here is that security controls have operational consequences.

### Shutdown

**Security:** Strong
**Availability:** Lower after violation
**Support impact:** Higher

### Restrict

**Security:** Strong enforcement of the MAC restriction
**Availability:** Port remains up
**Visibility:** Violation is logged

### Protect

**Security:** Traffic restriction
**Availability:** Port remains up
**Visibility:** Low

So the right choice depends on the environment.

For a highly controlled port:

```text
Shutdown
```

may be appropriate.

For an environment where accidental endpoint changes are common:

```text
Restrict
```

may provide a better operational balance.

---

# 20. Key Terminology

| Term               | Meaning                                                                             |
| ------------------ | ----------------------------------------------------------------------------------- |
| **Port Security**  | Layer 2 switch feature that restricts MAC addresses on a port                       |
| **Secure MAC**     | MAC address permitted by port-security policy                                       |
| **Sticky MAC**     | Dynamically learned MAC that is retained through configuration                      |
| **Maximum MAC**    | Maximum number of secure MAC addresses permitted                                    |
| **Violation**      | Unauthorized MAC exceeds the port-security policy                                   |
| **Protect**        | Drop unauthorized traffic without logging                                           |
| **Restrict**       | Drop unauthorized traffic and log the violation                                     |
| **Shutdown**       | Drop traffic, log violation, and error-disable the port                             |
| **Error-disabled** | Cisco state in which an interface has been disabled because of a detected condition |
| **Aging**          | Mechanism controlling how long learned secure MAC information remains               |

---

# 21. CCNA Exam Memory Table

### Violation Modes

```text
              Traffic   Log   Port
Protect        DROP      NO    UP
Restrict       DROP      YES   UP
Shutdown       DROP      YES   ERR-DISABLED
```

### Memorize this:

> **Protect = Drop**
> **Restrict = Drop + Report**
> **Shutdown = Drop + Report + Disable**

---

# 22. Most Important Takeaways

1. **Port security operates at Layer 2 using MAC addresses.**
2. It restricts **how many MAC addresses** can use a switchport.
3. A basic configuration can allow **one secure MAC address**.
4. A different unauthorized MAC can trigger a **security violation**.
5. **Shutdown is the default violation action** described in the lesson.
6. Shutdown places the interface into **error-disabled** state.
7. **Protect** drops unauthorized traffic silently.
8. **Restrict** drops unauthorized traffic and provides logging/visibility.
9. **Sticky MAC** dynamically learns a MAC and preserves it through configuration.
10. The maximum secure MAC count can be increased for legitimate multi-device scenarios such as **IP phone + PC**.
11. **Aging** helps prevent stale secure MAC information from remaining indefinitely.
12. Port security is a **boundary**, not a complete network-security solution.
13. In the Castle Rysen design, port security is one component of a larger Layer 2 security strategy that also includes **DHCP Snooping and DAI**. 

---

## 🔥 The mental model to remember

```text
                 SWITCHPORT
                     │
             ┌───────▼───────┐
             │ Port Security │
             └───────┬───────┘
                     │
              Check source MAC
                     │
             ┌───────┴───────┐
             │               │
          Allowed        Unauthorized
             │               │
          Forward         Violation
                             │
                 ┌───────────┼───────────┐
                 │           │           │
              Protect     Restrict    Shutdown
                 │           │           │
                Drop      Drop+Log   Drop+Log
                                        │
                                   Error-disabled
```

### One-sentence summary

> **Switch port security turns a normally trusting Layer 2 access port into an explicitly controlled port by limiting permitted MAC addresses and defining what the switch should do when that policy is violated.**
