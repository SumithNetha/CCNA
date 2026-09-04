# Week 14 — August 4

# Lab Notes: Configuring Dynamic ARP Inspection (DAI)

### Device: `cafe01-sw1`

This lab builds directly on **DHCP Snooping**. The switch already has a DHCP Snooping binding, and we use that information to deploy and verify DAI.

---

## 1. Verify the DHCP Snooping Binding

First, the existing DHCP Snooping binding was checked:

```text
cafe01-sw1#show ip dhcp snooping binding

MacAddress          IpAddress        Lease(sec)  Type           VLAN  Interface
-----------------   ---------------  ----------  -------------  ----  -----------------
00:0D:BD:A2:AB:06   10.0.18.2        0           dhcp-snooping  10    FastEthernet0/3

Total number of bindings: 1
```

### What this tells us

The switch has learned:

```text
MAC Address:  00:0D:BD:A2:AB:06
IP Address:   10.0.18.2
VLAN:         10
Interface:    Fa0/3
```

So the switch has a legitimate IP-to-MAC association that DAI can use.

### Important relationship

```text
DHCP Snooping
      │
      ▼
Binding Table
      │
      │ 10.0.18.2 ↔ 00:0D:BD:A2:AB:06
      ▼
DAI
```

This is the foundation of the DAI configuration.

---

# 2. Enable DAI on VLANs

Entered:

```text
cafe01-sw1(config)#ip arp inspection vlan 10,20
```

This enables Dynamic ARP Inspection for:

```text
VLAN 10
VLAN 20
```

### Verify

```text
cafe01-sw1(config)#do show ip arp inspection
```

Initially:

```text
Vlan     Configuration    Operation
----     -------------    ---------
10       Enabled          Inactive
20       Enabled          Inactive
```

### Important observation

The VLANs are configured for DAI, but the **Operation** status initially shows:

```text
Inactive
```

So:

```text
Configuration = Enabled
Operation     = Inactive
```

These are not the same thing.

---

# 3. Why Were VLANs Initially Inactive?

The next part of the lab configures trusted interfaces.

You entered:

```text
cafe01-sw1(config)#interface range fa0/1-2
```

Then:

```text
cafe01-sw1(config-if-range)#ip arp inspection trust
```

This makes:

```text
Fa0/1 → Trusted
Fa0/2 → Trusted
```

After that, the verification output changed:

```text
Vlan     Configuration    Operation
----     -------------    ---------
10       Enabled          Active
20       Enabled          Active
```

So the important progression in this lab was:

```text
DAI configured
     ↓
Operation inactive
     ↓
Configure trusted interfaces
     ↓
DAI operation active
```

---

# 4. Why Trust Fa0/1–2?

The command:

```text
ip arp inspection trust
```

changes the DAI trust state of an interface.

The lab uses:

```text
interface range fa0/1-2
ip arp inspection trust
```

Therefore:

```text
Fa0/1 ── Trusted
Fa0/2 ── Trusted
```

while the other interfaces remain untrusted.

---

# 5. Verify Interface Trust

The command:

```text
show ip arp inspection interfaces
```

was used, abbreviated in the lab as:

```text
show ip arp ins int
```

Output:

```text
Interface        Trust State     Rate(pps)    Burst Interval
---------------  -----------     ---------    --------------
Fa0/1            Trusted                15                 1
Fa0/2            Trusted                15                 1
Fa0/3            Untrusted              15                 1
Fa0/4            Untrusted              15                 1
...
Fa0/24           Untrusted              15                 1
Gig0/1           Untrusted              15                 1
Gig0/2           Untrusted              15                 1
```

### What we see

| Interface    | Trust State   |
| ------------ | ------------- |
| Fa0/1        | **Trusted**   |
| Fa0/2        | **Trusted**   |
| Fa0/3–Fa0/24 | **Untrusted** |
| Gi0/1        | **Untrusted** |
| Gi0/2        | **Untrusted** |

This is an important DAI design concept:

```text
Trusted interfaces
       ↓
ARP inspection trust boundary

Untrusted interfaces
       ↓
ARP inspection performed
```

---

# 6. ARP Rate Limiting

Notice this column:

```text
Rate(pps)
```

All interfaces currently show:

```text
15
```

That means the configured/default rate shown by this device is:

```text
15 ARP packets per second
```

The output also shows:

```text
Burst Interval = 1
```

This matters because DAI doesn't only validate ARP information; it also controls the rate of ARP traffic on untrusted interfaces.

For a normal endpoint this may be reasonable, while special/high-density interfaces such as a WAP may require additional consideration.

---

# 7. DAI Validation Options

The next step was:

```text
cafe01-sw1(config)#ip arp inspection validate ?
```

The switch presented:

```text
dst-mac    Validate destination MAC address
ip         Validate IP address
src-mac    Validate source MAC address
```

Therefore DAI can validate three categories:

```text
Source MAC
Destination MAC
IP Address
```

---

# 8. Source MAC Validation

```text
ip arp inspection validate src-mac
```

Conceptually, the switch checks the source MAC information in the ARP packet.

This helps detect situations where an attacker attempts to use an unexpected MAC address.

---

# 9. Destination MAC Validation

```text
ip arp inspection validate dst-mac
```

This validates the destination MAC information contained in the ARP packet.

This provides an additional layer of ARP inspection.

---

# 10. IP Validation

```text
ip arp inspection validate ip
```

This validates the IP address information in the ARP packet.

This is particularly important for detecting forged IP/MAC relationships.

---

# 11. Why These Validations Matter

Suppose DHCP Snooping has learned:

```text
IP:  10.0.18.2
MAC: 00:0D:BD:A2:AB:06
```

An attacker attempts to claim:

```text
10.0.18.2
       ↓
Attacker's MAC
```

DAI can compare the information being advertised against what the switch knows.

```text
DHCP Snooping Binding
          │
          ▼
10.0.18.2 ↔ 00:0D:BD:A2:AB:06
          │
          ▼
       DAI check
          │
      ┌───┴────┐
      │        │
    Match   Mismatch
      │        │
    Allow     Drop
```

---

# 12. Current Validation Status

Before configuring the validation options, the `show ip arp inspection` output showed:

```text
Source Mac Validation      : Disabled
Destination Mac Validation : Disabled
IP Address Validation      : Disabled
```

So at this stage:

```text
DAI VLAN protection → Enabled
Interface trust      → Configured
Validation options   → Disabled
```

The lab has therefore reached the point where the individual validation mechanisms can be configured.

---

# 13. Understanding the Complete Lab

Your configuration so far can be visualized as:

```text
                     cafe01-sw1
                         │
             ┌───────────┴───────────┐
             │                       │
          Fa0/1                   Fa0/2
         TRUSTED                 TRUSTED
             │                       │
             └───────────┬───────────┘
                         │
                 DAI-enabled VLANs
                    VLAN 10, 20
                         │
            ┌────────────┴────────────┐
            │                         │
         Fa0/3...                  Other ports
        UNTRUSTED                  UNTRUSTED
            │
            ▼
       ARP inspection
            │
            ▼
   Compare ARP information
   against legitimate bindings
```

---

# 14. Important Lab Commands

### Check DHCP Snooping bindings

```text
show ip dhcp snooping binding
```

### Enable DAI for VLANs

```text
ip arp inspection vlan 10,20
```

### Check DAI status

```text
show ip arp inspection
```

### Enter interfaces

```text
interface range fa0/1-2
```

### Trust interfaces

```text
ip arp inspection trust
```

### Check interface trust and rate

```text
show ip arp inspection interfaces
```

### See available validation options

```text
ip arp inspection validate ?
```

### Validation options

```text
src-mac
dst-mac
ip
```

---

# 15. What Happened in This Lab?

### Before DAI

```text
DHCP Snooping binding exists
```

```text
10.0.18.2
    ↕
00:0D:BD:A2:AB:06
```

### Enable DAI

```text
ip arp inspection vlan 10,20
```

Result:

```text
VLAN 10 → Enabled / Inactive
VLAN 20 → Enabled / Inactive
```

### Trust Fa0/1–2

```text
interface range fa0/1-2
ip arp inspection trust
```

Result:

```text
VLAN 10 → Enabled / Active
VLAN 20 → Enabled / Active
```

### Verify interfaces

```text
Fa0/1 → Trusted
Fa0/2 → Trusted

Everything else → Untrusted
```

### Check validation capabilities

```text
src-mac
dst-mac
ip
```

---

# 🧠 Key Lab Takeaways

1. **DAI protects against ARP spoofing/poisoning.**

2. **DHCP Snooping provides the IP/MAC binding information that DAI can use.**

3. DAI is enabled per VLAN:

```text
ip arp inspection vlan 10,20
```

4. Interfaces can be explicitly trusted:

```text
ip arp inspection trust
```

5. Interfaces are otherwise shown as **Untrusted**.

6. The lab's Fa0/1 and Fa0/2 were configured as trusted.

7. DAI supports validation of:

```text
Source MAC
Destination MAC
IP Address
```

8. DAI also has an **ARP packet rate limit**, shown in:

```text
show ip arp inspection interfaces
```

9. Always verify DAI after configuration:

```text
show ip arp inspection
show ip arp inspection interfaces
```

10. **Do not confuse "Configuration Enabled" with "Operation Active."** Your lab demonstrated this directly.

---

## 🔥 The most important relationship from today's lab

```text
            DHCP Snooping
                  │
                  ▼
        DHCP Snooping Binding
                  │
                  │
                  ▼
                 DAI
                  │
          ┌───────┼────────┐
          ▼       ▼        ▼
       Source   Dest.      IP
        MAC      MAC     Validation
          │       │        │
          └───────┼────────┘
                  ▼
            ARP Decision
             /          \
          Valid        Invalid
            │             │
          Allow          Drop
```

**This is the core of the August 4 DAI lab.**
