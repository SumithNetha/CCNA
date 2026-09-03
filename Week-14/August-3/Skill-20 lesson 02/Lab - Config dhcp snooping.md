# Skill 20 Lab — Configuring DHCP Snooping

## S20-L02 — Lab Notes & Explanation

Your lab demonstrates **DHCP Snooping configuration, trusted interfaces, DHCP bindings, and troubleshooting DHCP behavior in Packet Tracer**.

The important part is that your output shows both the **intended DHCP Snooping operation** and a couple of configuration mistakes that are useful to understand.

---

# 1. Starting State

On `cafe01-sw1`, you first checked the interfaces:

```text
cafe01-sw1#show ip int br

Interface              IP-Address      OK? Method Status  Protocol
Port-channel1          unassigned      YES manual up      up
FastEthernet0/1        unassigned      YES manual up      up
FastEthernet0/2        unassigned      YES manual up      up
FastEthernet0/3        unassigned      YES manual up      up
FastEthernet0/4        unassigned      YES manual down    down
...
```

Important observations:

* `Port-channel1` is up.
* `Fa0/1`, `Fa0/2`, and `Fa0/3` are up.
* Most remaining interfaces are down.
* The switch itself doesn't need an IP address on these Layer 2 interfaces for DHCP Snooping to operate.

---

# 2. Enabling DHCP Snooping

You configured:

```cisco
ip dhcp snooping
```

This enables DHCP Snooping globally on the switch.

Then:

```cisco
ip dhcp snooping vlan 10
ip dhcp snooping vlan 20
```

This enables DHCP Snooping for:

```text
VLAN 10
VLAN 20
```

Your verification confirms:

```text
Switch DHCP snooping is enabled

DHCP snooping is configured on following VLANs:
10,20
```

So your configuration is active.

---

# 3. Trusted Interface

You configured:

```cisco
interface fa0/24
 ip dhcp snooping trust
```

Then later:

```cisco
interface fa0/3
 ip dhcp snooping trust
```

The verification output shows:

```text
Interface                  Trusted
-----------------------    -------
FastEthernet0/3            yes
FastEthernet0/24           yes
```

Therefore, **both Fa0/3 and Fa0/24 are currently trusted DHCP Snooping interfaces**.

---

# 4. What Does `ip dhcp snooping trust` Actually Mean?

This command:

```cisco
ip dhcp snooping trust
```

means:

> **DHCP server responses are permitted to enter through this interface.**

It does **not** mean:

> "This is a trusted user."

It establishes a **DHCP trust boundary**.

Think:

```text
                 DHCP Server
                     |
                     | DHCP Offer
                     ↓
               TRUSTED PORT
                     |
                   Switch
                  /      \
                 /        \
           UNTRUSTED    UNTRUSTED
             Client       Client
```

The source lesson emphasizes that you should trust the interface where legitimate DHCP offers are received, rather than trusting a client port merely because the client needs DHCP.

---

# 5. Your First Important Configuration Mistake

You initially did:

```text
cafe01-sw02(config)#ip dhcp sn tr
                               ^
% Invalid input detected at '^' marker.
```

The reason is that you were in:

```text
(config)#
```

global configuration mode.

But:

```cisco
ip dhcp snooping trust
```

is an **interface-level command**.

You need:

```cisco
conf t
interface fa0/1
ip dhcp snooping trust
```

or:

```cisco
interface range fa0/1-2
ip dhcp snooping trust
```

depending on which interfaces actually need to be trusted.

---

# 6. Your Second Mistake — Interface Range Syntax

You tried:

```text
cafe01-sw02(config)#int fa0/1-2
                            ^
% Invalid input detected at '^' marker.
```

This isn't the correct syntax for selecting an interface range.

You corrected it to:

```cisco
interface range fa0/1-2
```

and Cisco accepted it:

```text
cafe01-sw02(config-if-range)#
```

### Remember

Single interface:

```cisco
interface fa0/1
```

Multiple interfaces:

```cisco
interface range fa0/1-2
```

This is useful because you can apply the same configuration to multiple interfaces simultaneously.

---

# 7. Your DHCP Snooping Verification

You ran:

```cisco
show ip dhcp snooping
```

and received:

```text
Switch DHCP snooping is enabled
DHCP snooping is configured on following VLANs:
10,20
Insertion of option 82 is enabled
Option 82 on untrusted port is not allowed
Verification of hwaddr field is enabled
```

Let's break this down.

---

## DHCP Snooping is enabled

```text
Switch DHCP snooping is enabled
```

Global feature:

```cisco
ip dhcp snooping
```

is active.

---

## VLANs 10 and 20 are protected

```text
DHCP snooping is configured on following VLANs:
10,20
```

Equivalent configuration:

```cisco
ip dhcp snooping vlan 10
ip dhcp snooping vlan 20
```

---

# 8. Option 82

Your output says:

```text
Insertion of option 82 is enabled
```

**DHCP Option 82** is DHCP Relay Agent Information.

It can provide additional information about where a DHCP request originated, such as the switch/interface through which the client request arrived.

For this particular lab, the most important thing is simply recognizing that Packet Tracer shows Option 82 as enabled.

---

# 9. Option 82 on Untrusted Port

Your switch says:

```text
Option 82 on untrusted port is not allowed
```

This reinforces the trust-boundary concept.

Untrusted interfaces are not supposed to be sources of unauthorized DHCP infrastructure traffic.

---

# 10. DHCP Snooping Binding Table

You then ran:

```cisco
show ip dhcp snooping binding
```

and got:

```text
MacAddress          IpAddress        Lease(sec)  Type           VLAN  Interface
------------------  ---------------  ----------  -------------  ----  -----------------
00:0D:BD:A2:AB:06   10.0.18.2        0           dhcp-snooping  10    FastEthernet0/3

Total number of bindings: 1
```

This is one of the most important outputs in the lab.

The switch has learned:

```text
MAC:
00:0D:BD:A2:AB:06

        ↓

IP:
10.0.18.2

        ↓

VLAN:
10

        ↓

Interface:
Fa0/3
```

So the switch has effectively recorded:

> **MAC `00:0D:BD:A2:AB:06` received IP `10.0.18.2` through VLAN 10 on Fa0/3.**

---

# 11. Compare This With the PC

Your PC showed:

```text
IPv4 Address....................: 10.0.18.2
Subnet Mask.....................: 255.255.255.224
Default Gateway.................: 10.0.18.1
```

The MAC shown in the DHCP Snooping table is:

```text
00:0D:BD:A2:AB:06
```

And the PC's IPv6 link-local address:

```text
FE80::20D:BDFF:FEA2:AB06
```

also contains the same underlying interface identifier:

```text
20D:BDFF:FEA2:AB06
```

So the binding table corresponds to the PC you were testing.

---

# 12. The DHCP Lease Flow

Your first test was:

```text
ipconfig
```

The PC already had:

```text
10.0.18.2/27
Gateway: 10.0.18.1
```

Then:

```text
ipconfig /release
```

removed the DHCP configuration:

```text
IP Address:      0.0.0.0
Subnet Mask:     0.0.0.0
Gateway:         0.0.0.0
DNS:             0.0.0.0
```

Then:

```text
ipconfig /renew
```

successfully obtained:

```text
IP Address:      10.0.18.2
Subnet Mask:     255.255.255.224
Default Gateway: 10.0.18.1
DNS Server:      1.1.1.1
```

This demonstrates that DHCP is functioning through the configured trust boundary.

---

# 13. The Interesting Part — DHCP Request Failed

You then released the address again:

```text
C:\>ipconfig /release
```

and attempted:

```text
C:\>ipconfig /renew
DHCP request failed.
```

It failed multiple times:

```text
DHCP request failed.
DHCP request failed.
DHCP request failed.
```

Then you tried again:

```text
C:\>ipconfig /renew
```

and it succeeded:

```text
IP Address:      10.0.18.2
Subnet Mask:     255.255.255.224
Default Gateway: 10.0.18.1
DNS Server:      1.1.1.1
```

---

# 14. Why This Is Important

This is exactly where you should apply the troubleshooting mindset from the lesson.

Don't immediately conclude:

> "The DHCP server is broken."

Instead investigate:

```text
Client
  ↓
DHCP Discover
  ↓
Switch
  ↓
DHCP Server
  ↓
DHCP Offer
  ↓
Switch
  ↓
Client
```

Ask:

> **Where is the DHCP Offer coming from?**

Then verify the interface receiving that offer is trusted.

The lesson specifically warns that Packet Tracer can behave inconsistently with DHCP Snooping and VLANs, so you should not change the real-world security design merely to accommodate a simulator behavior.

---

# 15. Important Problem in Your Current Configuration

Your final output shows:

```text
FastEthernet0/3    yes
FastEthernet0/24   yes
```

But your binding table says:

```text
10.0.18.2 → Fa0/3
```

So Fa0/3 appears to be the **client-facing interface** for the PC.

If that's true, then this configuration:

```cisco
interface fa0/3
 ip dhcp snooping trust
```

would be questionable in a real production design.

Why?

Because you've made the client-facing interface trusted.

That means you're saying:

```text
Fa0/3
   ↓
"I trust DHCP server traffic coming from here."
```

But if Fa0/3 is:

```text
Switch → Client PC
```

you generally **do not want it trusted**.

---

# 16. What the Real-World Design Should Look Like

Assuming:

```text
Fa0/3 = Client
Fa0/24 = Uplink toward legitimate DHCP source
```

then the conceptual configuration should be:

```text
Fa0/3
Client
  ↓
UNTRUSTED


Fa0/24
Uplink
  ↓
TRUSTED
```

So:

```cisco
interface fa0/24
 ip dhcp snooping trust

interface fa0/3
 no ip dhcp snooping trust
```

**However**, don't blindly change your Packet Tracer lab if the lesson's topology specifically requires Fa0/3 to be trusted to accommodate its simulated DHCP path. First identify where the DHCP server actually sits in your topology.

The lesson's rule remains:

> **Trust the path that legitimately carries DHCP server responses.**

---

# 17. Why the Binding Table Can Still Exist

An important point:

The fact that:

```text
Fa0/3 → binding exists
```

does **not** mean Fa0/3 should necessarily be trusted.

DHCP Snooping is specifically designed to learn client bindings from DHCP transactions.

So you can have:

```text
Fa0/3 = UNTRUSTED
       ↓
Client sends DHCP request
       ↓
Legitimate DHCP response returns through trusted path
       ↓
Switch learns binding
       ↓
Fa0/3 → MAC/IP/VLAN binding
```

That's perfectly consistent with the security model.

---

# 18. Why the Binding Table Is Valuable

Your binding:

```text
00:0D:BD:A2:AB:06
        ↓
10.0.18.2
        ↓
VLAN 10
        ↓
Fa0/3
```

becomes useful to subsequent security mechanisms.

The next lesson is **DAI — Dynamic ARP Inspection**.

DAI can use DHCP Snooping information to validate IP/MAC relationships.

So your Skill 20 progression is becoming:

```text
                LAYER 2 SECURITY
                       │
          ┌────────────┼────────────┐
          ↓            ↓            ↓
    Port Security DHCP Snooping    DAI
          │            │            │
        MAC          DHCP        ARP/IP-MAC
      control       control       validation
                       │
                       ↓
                Binding Table
```

---

# 19. Commands You Used

## Enable DHCP Snooping

```cisco
ip dhcp snooping
```

## Enable on VLANs

```cisco
ip dhcp snooping vlan 10
ip dhcp snooping vlan 20
```

or:

```cisco
ip dhcp snooping vlan 10,20
```

## Trust an interface

```cisco
interface fa0/24
 ip dhcp snooping trust
```

## Trust a range

```cisco
interface range fa0/1-2
 ip dhcp snooping trust
```

## Verify configuration

```cisco
show ip dhcp snooping
```

## Verify DHCP bindings

```cisco
show ip dhcp snooping binding
```

## Verify interface status

```cisco
show ip interface brief
```

---

# 20. Your Lab in One Diagram

Based on your outputs, we can represent the tested portion as:

```text
                         DHCP Infrastructure
                                |
                                |
                            Fa0/24
                           TRUSTED
                                |
                         +-------------+
                         |  cafe01-sw1 |
                         +-------------+
                                |
                              Fa0/3
                            TRUSTED*
                                |
                              PC
                         10.0.18.2/27
                         MAC:
                    00:0D:BD:A2:AB:06
                                |
                         VLAN 10

* Verify whether Fa0/3 is actually the
  legitimate DHCP-offer path in the lab.
```

The critical real-world correction is that **if Fa0/3 is merely the client access port, it should normally remain untrusted**.

---

# 21. Lab Troubleshooting Lessons

You encountered three useful situations:

### ① Wrong configuration mode

```text
(config)#
ip dhcp snooping trust
```

❌ Incorrect.

Use interface configuration mode:

```text
(config-if)#
ip dhcp snooping trust
```

---

### ② Incorrect interface-range syntax

```text
int fa0/1-2
```

❌ Incorrect.

Use:

```text
interface range fa0/1-2
```

✅ Correct.

---

### ③ DHCP intermittently failed

```text
DHCP request failed.
```

Then later succeeded.

Don't immediately blame DHCP.

Check:

```text
DHCP server
     ↓
DHCP offer path
     ↓
Every switch
     ↓
Trusted uplinks
     ↓
Client
```

And remember the lesson's Packet Tracer warning.

---

# 22. Git-Ready Lab Notes

````markdown
# S20-L02 — Configuring DHCP Snooping

## Objective

Configure DHCP Snooping to protect VLANs from rogue DHCP servers and verify the DHCP Snooping binding table.

## Configuration

### Enable DHCP Snooping globally

```cisco
ip dhcp snooping
````

### Enable DHCP Snooping for VLANs

```cisco
ip dhcp snooping vlan 10,20
```

### Configure trusted DHCP path

```cisco
interface fa0/24
 ip dhcp snooping trust
```

### Configure multiple interfaces

```cisco
interface range fa0/1-2
 ip dhcp snooping trust
```

## Verification

```cisco
show ip dhcp snooping
show ip dhcp snooping binding
```

Example:

```text
Switch DHCP snooping is enabled

DHCP snooping is configured on following VLANs:
10,20

Interface                  Trusted
-----------------------    -------
FastEthernet0/3            yes
FastEthernet0/24           yes
```

Binding table:

```text
MacAddress          IpAddress     VLAN  Interface
------------------  ------------  ----  ----------------
00:0D:BD:A2:AB:06   10.0.18.2     10    FastEthernet0/3
```

## Key Concepts

* DHCP Snooping creates a trust boundary for DHCP traffic.
* DHCP Snooping is enabled globally and then applied to specific VLANs.
* Interfaces are untrusted by default.
* Only interfaces that legitimately receive DHCP server responses should be trusted.
* Client-facing access ports should normally remain untrusted.
* DHCP Snooping builds a binding table containing MAC, IP, VLAN, and interface information.
* The binding table can support Dynamic ARP Inspection (DAI).
* Packet Tracer may behave inconsistently with DHCP Snooping; simulator behavior should not replace real-world design principles.

## Troubleshooting

If DHCP stops working:

1. Locate the legitimate DHCP server.
2. Trace the DHCP Offer path.
3. Identify every switch interface receiving the legitimate Offer.
4. Verify those interfaces are trusted.
5. Check each switch along the path.
6. Verify the DHCP Snooping binding table.

## Important Command Difference

`ip dhcp snooping trust` is an interface-level command.

Correct:

```cisco
interface fa0/24
 ip dhcp snooping trust
```

For multiple interfaces:

```cisco
interface range fa0/1-2
 ip dhcp snooping trust
```

## Security Principle

Trust the **DHCP offer source/path**, not the client simply because the client requires DHCP.

```

### Bottom line

Your lab successfully demonstrated the key DHCP Snooping mechanism: **DHCP Snooping is enabled for VLANs 10/20, the switch has trusted interfaces, and the switch successfully built a binding for `10.0.18.2` on VLAN 10/Fa0/3.** The biggest thing I would flag in your lab is **Fa0/3 being marked trusted even though the binding shows the PC on Fa0/3**—verify the topology before treating that as a correct production configuration. :contentReference[oaicite:0]{index=0}
```
