# Configuring IPv6 Overlay at Castle Rysen Cafe

This lesson is where IPv6 moves from **address formatting** into **actual network deployment**. The main idea is simple:

> **Don't replace IPv4. Add IPv6 alongside it and transition gradually.**

Castle Rysen already has a functioning IPv4 network, so the goal is to introduce IPv6 without breaking what already works.

---

# 1. What Is an IPv6 Overlay?

An **IPv6 overlay** in this lesson means running IPv6 **alongside the existing IPv4 network**.

You are not doing:

```text
IPv4 network
     ↓
DELETE IPv4
     ↓
Build IPv6 network
```

Instead:

```text
Existing IPv4 network
        +
       IPv6
        ↓
IPv4 + IPv6 coexist
        ↓
Gradual IPv6 transition
```

This is realistic because organizations generally cannot replace every IPv4 device simultaneously.

Some devices may still have limited or no IPv6 support, while newer infrastructure can support both.

So the practical strategy is:

> **Keep IPv4 working while introducing IPv6.**

---

# 2. Why IPv6 Subnetting Is Different

IPv4 subnetting often focuses heavily on **conserving addresses**.

You repeatedly ask:

* How many hosts do I need?
* How many subnets?
* What subnet mask?
* What is the increment?
* What is the usable range?
* What is the broadcast address?

IPv6 has such a huge address space that the design philosophy changes.

The lesson's key phrase is:

> **"In IPv4, we subnet to conserve. In IPv6, we subnet to organize."**

That's an important mindset shift.

You're still designing a structured network, but you're generally not trying to squeeze every possible host address out of a subnet.

---

# 3. Why `/64` Is So Important

Most IPv6 LANs commonly use a **/64 prefix**.

IPv6 has:

```text
128 bits total
```

With a `/64`:

```text
64 bits → network prefix
64 bits → interface ID
```

Conceptually:

```text
|<--------- 64 bits --------->|<--------- 64 bits --------->|
        Network Prefix                 Interface ID
```

For example:

```text
2001:db8:1:1::/64
```

The `/64` identifies the subnet.

---

# 4. A `/48` Gives You Huge Subnetting Space

Suppose an organization receives:

```text
2001:db8:1::/48
```

IPv6 addresses have eight hextets:

```text
2001 : db8 : 1 : 0000 : 0000 : 0000 : 0000 : 0000
  1     2    3     4      5      6      7      8
```

A `/48` means the first **three hextets** provide the allocated 48-bit prefix.

That leaves the fourth hextet as a convenient place to create `/64` subnet identifiers.

```text
2001:db8:1:SUBNET::/64
             ↑
        fourth hextet
```

Since the fourth hextet contains 16 bits:

```text
2^16 = 65,536
```

So a `/48` can provide:

```text
65,536 × /64 subnets
```

This is why IPv6 subnet allocation feels very different from IPv4.

---

# 5. Castle Rysen IPv6 Addressing Design

The lesson uses the documentation prefix:

```text
2001:db8:1::/48
```

The fourth hextet is then used to identify different VLANs/subnets.

For example:

### VLAN 10

```text
2001:db8:1:1::/64
```

### VLAN 20

```text
2001:db8:1:2::/64
```

This gives a very predictable structure:

```text
2001:db8:1:1::/64
           ↑
        VLAN subnet
```

and:

```text
2001:db8:1:2::/64
           ↑
        VLAN subnet
```

The exact numbering scheme is less important than having a **consistent, understandable addressing plan**.

---

# 6. Don't Confuse Decimal VLAN Numbers With Hexadecimal

This is an easy place to make mistakes.

IPv6 hextets are written in **hexadecimal**.

Therefore:

```text
Decimal 10 = Hex A
Decimal 20 = Hex 14
```

So if you wanted to directly represent VLAN numbers in the fourth hextet:

```text
VLAN 10 → :a:
VLAN 20 → :14:
```

However, the lesson deliberately uses:

```text
VLAN 10 → 2001:db8:1:1::/64
VLAN 20 → 2001:db8:1:2::/64
```

The important lesson is:

> **Use an addressing scheme that is easy for your team to understand and support.**

Don't create a complicated scheme simply because it looks clever.

---

# 7. Configuring IPv6 on a Cisco Router

The configuration is surprisingly similar to IPv4.

For IPv4, you might use:

```text
interface GigabitEthernet0/0
 ip address 192.168.1.1 255.255.255.0
```

For IPv6:

```text
interface GigabitEthernet0/0
 ipv6 address 2001:db8:1:1::1/64
```

The important difference is:

```text
ip address
```

becomes:

```text
ipv6 address
```

---

# 8. The Most Important Command: `ipv6 unicast-routing`

Before the router can actually route IPv6 traffic, enable IPv6 unicast routing globally:

```text
ipv6 unicast-routing
```

This is configured from global configuration mode:

```text
enable
configure terminal
ipv6 unicast-routing
```

Then configure the interfaces/subinterfaces.

---

# 9. Why `ipv6 unicast-routing` Matters

This is one of the most important things to remember from today's lesson.

You could configure:

```text
interface GigabitEthernet0/0
 ipv6 address 2001:db8:1:1::1/64
```

and the interface could appear to have an IPv6 address.

But if you forget:

```text
ipv6 unicast-routing
```

the router will not perform IPv6 routing between networks.

So think:

```text
IPv6 address configured
        ≠
IPv6 routing enabled
```

You need both.

### Mental checklist

```text
1. Enable IPv6 routing
       ↓
ipv6 unicast-routing

2. Configure IPv6 address
       ↓
ipv6 address <address>/<prefix>

3. Enable interface
       ↓
no shutdown
```

---

# 10. Castle Rysen VLAN Example

Suppose Castle Rysen Cafe has two VLANs.

### VLAN 10

Network:

```text
2001:db8:1:1::/64
```

Router gateway:

```text
2001:db8:1:1::1/64
```

### VLAN 20

Network:

```text
2001:db8:1:2::/64
```

Router gateway:

```text
2001:db8:1:2::1/64
```

If router-on-a-stick is being used, the configuration could conceptually look like:

```text
ipv6 unicast-routing

interface GigabitEthernet0/0.10
 encapsulation dot1Q 10
 ipv6 address 2001:db8:1:1::1/64

interface GigabitEthernet0/0.20
 encapsulation dot1Q 20
 ipv6 address 2001:db8:1:2::1/64
```

The important lesson-specific concepts are:

* IPv6 address assigned to each VLAN gateway
* `/64` used for each LAN
* Fourth hextet used to identify the subnet
* `ipv6 unicast-routing` enabled globally

---

# 11. IPv6 Addresses Automatically Get Link-Local Addresses

One of the interesting things you'll see when verifying the configuration is an address beginning with:

```text
FE80::
```

You may not have manually configured it.

That's because IPv6 interfaces use **link-local addresses** for communication on the local link.

For example, after configuring:

```text
ipv6 address 2001:db8:1:1::1/64
```

you might see something like:

```text
FE80::...
2001:DB8:1:1::1
```

The `FE80::` address wasn't necessarily manually configured.

That's normal IPv6 behavior.

---

# 12. Verify the Configuration

The lesson specifically highlights:

```text
show ipv6 interface brief
```

This is an important command for checking IPv6 interfaces.

Example conceptually:

```text
Router# show ipv6 interface brief

Interface              IPv6-Address
GigabitEthernet0/0.10   2001:DB8:1:1::1
                        FE80::...
GigabitEthernet0/0.20   2001:DB8:1:2::1
                        FE80::...
```

You're looking for:

* IPv6 addresses
* Link-local addresses
* Interface status

---

# 13. Why the Link-Local Address Matters

Don't think:

> "I didn't configure that address, so something is wrong."

Instead:

```text
IPv6 interface
      │
      ├── Global/other IPv6 address
      │
      └── FE80:: link-local address
```

The lesson deliberately introduces this because the **next lesson goes deeper into IPv6 address types**, particularly link-local addressing.

---

# 14. IPv4 + IPv6 at Castle Rysen

The overall architecture now looks something like:

```text
                Castle Rysen Cafe
                       │
             ┌─────────┴─────────┐
             │                   │
           IPv4                 IPv6
             │                   │
             └─────────┬─────────┘
                       │
                Existing Network
                       │
                VLANs / Subnets
```

IPv4 isn't being removed.

IPv6 is being introduced alongside it.

This is the practical transition strategy the lesson is demonstrating.

---

# 15. Addressing Strategy

The lesson's Castle Rysen example can be visualized as:

```text
2001:db8:1::/48
       │
       ├── :1: → VLAN 10
       │        2001:db8:1:1::/64
       │
       └── :2: → VLAN 20
                2001:db8:1:2::/64
```

Then the router's gateway addresses:

```text
VLAN 10
2001:db8:1:1::1/64

VLAN 20
2001:db8:1:2::1/64
```

This is a **clean, predictable addressing hierarchy**.

---

# 16. Important Concepts From This Lesson

| Concept                     | Meaning                                         |
| --------------------------- | ----------------------------------------------- |
| IPv6 overlay                | IPv6 operates alongside existing IPv4           |
| Dual-stack concept          | IPv4 and IPv6 can coexist                       |
| IPv6 address size           | 128 bits                                        |
| Common LAN prefix           | `/64`                                           |
| Example allocation          | `/48`                                           |
| `/48` → `/64`               | 65,536 possible `/64` subnets                   |
| Fourth hextet               | Convenient place for subnet identification      |
| `ipv6 address`              | Assigns IPv6 address to an interface            |
| `ipv6 unicast-routing`      | Enables IPv6 routing on Cisco router            |
| `FE80::/10`                 | Link-local address range                        |
| `show ipv6 interface brief` | Verify IPv6 interfaces                          |
| Main design principle       | IPv6 subnetting is primarily about organization |

---

# 17. Commands to Remember

### Enable IPv6 routing

```text
ipv6 unicast-routing
```

### Enter interface

```text
interface GigabitEthernet0/0
```

### Assign IPv6 address

```text
ipv6 address 2001:db8:1:1::1/64
```

### Enable interface

```text
no shutdown
```

### Verify

```text
show ipv6 interface brief
```

---

# 18. What You Need to Remember for CCNA

If you only memorize **five things** from this lesson, make them these:

### 1. IPv6 overlay

```text
IPv4 + IPv6
```

IPv6 can be introduced without immediately removing IPv4.

### 2. `/64` is the common IPv6 LAN size

```text
/64 = 64-bit network prefix + 64-bit interface ID
```

### 3. `/48` gives substantial subnetting space

```text
/48 → /64
```

leaves 16 bits for subnetting:

```text
2^16 = 65,536 /64 subnets
```

### 4. The fourth hextet is useful for subnet organization

```text
2001:db8:1:1::/64
            ↑
        subnet ID
```

### 5. Don't forget:

```text
ipv6 unicast-routing
```

Without it, the Cisco router won't actually route IPv6 traffic.

---

## The Big Picture

Today's lesson is less about memorizing IPv6 commands and more about understanding **how IPv6 is introduced into an existing network**:

```text
              EXISTING CASTLE RYSEN NETWORK
                         │
                 ┌───────┴───────┐
                 │               │
                IPv4            IPv6
                 │               │
                 │        2001:db8:1::/48
                 │               │
                 │        ┌──────┴──────┐
                 │        │             │
                 │      /64           /64
                 │        │             │
                 │   VLAN 10        VLAN 20
                 │        │             │
                 │   ...:1::/64    ...:2::/64
                 │
                 └──── Coexists ──────┘
```

**Core takeaway:** IPv6 deployment doesn't have to mean destroying the IPv4 network. At Castle Rysen, IPv6 is layered onto the existing infrastructure using a structured `/48 → /64` addressing strategy, with the fourth hextet used to organize subnets and `ipv6 unicast-routing` enabled on the Cisco router. The automatically appearing `FE80::` address is your introduction to **IPv6 link-local addressing**, which is the focus of the next lesson.
