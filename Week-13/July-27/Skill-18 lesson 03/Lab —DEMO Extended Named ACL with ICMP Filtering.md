# Lab Notes — Extended Named ACL with ICMP Filtering

**Topic:** Extended ACL — ICMP filtering, sequence numbers, ACL application, exceptions, and verification
**Device:** `cafe01-RT01`
**Environment:** Cisco Packet Tracer
**Source:** Your lab output and topology 

---

## 1. Lab Objective

Configure an **Extended Named ACL** on `cafe01-RT01` to control ICMP traffic from PC1.

Initial requirement:

> Deny PC1 (`10.0.18.2`) from pinging Server (`10.0.16.11`).

Then modify the ACL to:

> Deny PC1 from pinging **any destination**, but create an exception allowing PC1 to ping `10.0.16.11`.

---

## 2. Relevant Topology

From the topology:

```text
PC1
10.0.18.2/27
Gateway: 10.0.18.1
   |
   | VLAN 10
   |
cafe01-SW01
   |
   |
cafe01-RT01
   |
   | Routing
   |
10.0.16.0/23
   |
Server
10.0.16.11
```

### Important addresses

| Device               | IP             | Network         |
| -------------------- | -------------- | --------------- |
| PC1                  | `10.0.18.2/27` | `10.0.18.0/27`  |
| RT01 VLAN 10 gateway | `10.0.18.1`    | VLAN 10         |
| RT01 VLAN 20         | `10.0.18.33`   | `10.0.18.32/27` |
| Server               | `10.0.16.11`   | `10.0.16.0/23`  |
| RT01 WAN             | `172.16.0.2`   | `172.16.0.0/30` |

---

## 3. Verify Router Interfaces

Command:

```cisco
show ip interface brief
```

Relevant output:

```text
GigabitEthernet0/0.10    10.0.18.1
GigabitEthernet0/0.20    10.0.18.33
Serial0/1/0              172.16.0.2
```

Both VLAN subinterfaces are:

```text
up/up
```

This confirms the router's VLAN interfaces are operational.

---

## 4. Create a Named Extended ACL

Enter configuration mode:

```cisco
configure terminal
```

Create the ACL:

```cisco
ip access-list extended Demo
```

The prompt changes to:

```text
cafe01-RT01(config-ext-nacl)#
```

This indicates that you are configuring the **Extended Named ACL `Demo`**.

---

## 5. Initial ACL — Block One Specific Ping

Configured:

```cisco
deny icmp host 10.0.18.2 host 10.0.16.11 echo
```

Breakdown:

```text
deny
 ↓
ICMP
 ↓
source = 10.0.18.2
 ↓
destination = 10.0.16.11
 ↓
ICMP type = echo
```

Therefore:

```text
PC1 ── ICMP Echo ──X──> Server
10.0.18.2             10.0.16.11
```

Only **ICMP Echo requests** from PC1 to that specific server are denied.

---

## 6. Add a Permit for Other IP Traffic

You then configured:

```cisco
permit ip any any
```

The complete ACL became:

```cisco
10 deny icmp host 10.0.18.2 host 10.0.16.11 echo
20 permit ip any any
```

The logic is:

```text
PC1 → Server → ICMP Echo
             ↓
           DENY

Everything else
             ↓
           PERMIT
```

This `permit ip any any` is important because otherwise unmatched IP traffic eventually reaches the ACL's implicit deny.

---

## 7. Verify the ACL

Command:

```cisco
show ip access-lists
```

Output:

```text
Extended IP access list Demo
    10 deny icmp host 10.0.18.2 host 10.0.16.11 echo
    20 permit ip any any
```

This confirms the ACL exists and contains the expected entries.

---

## 8. Apply the ACL to the Interface

Creating an ACL does **not** automatically activate it.

You applied it to VLAN 10:

```cisco
interface gigabitEthernet0/0.10
ip access-group Demo in
```

The important part is:

```text
Demo in
```

Meaning:

> Apply ACL `Demo` to packets entering `Gi0/0.10`.

Traffic direction:

```text
PC1
 |
 | packet
 ↓
Gi0/0.10
 |
 ↓
ACL Demo
 |
 ↓
Router
```

Because PC1 belongs to VLAN 10, this is an appropriate location for filtering traffic originating from PC1.

---

## 9. Verify ACL Match Counters

After generating traffic:

```cisco
show ip access-lists
```

you obtained:

```text
10 deny icmp host 10.0.18.2 host 10.0.16.11 echo (56 match(es))
20 permit ip any any
```

The:

```text
(56 match(es))
```

means packets have matched sequence 10.

### Very important troubleshooting concept

ACL counters provide evidence that traffic is actually hitting a particular ACL entry.

---

## 10. Change the ACL Scope

You then removed sequence 10:

```cisco
no 10
```

and replaced it with:

```cisco
10 deny icmp host 10.0.18.2 any echo
```

Now the policy became:

```text
PC1 → ICMP Echo → ANY destination
              ↓
            DENY
```

This is much broader than the original rule.

### Original

```cisco
deny icmp host 10.0.18.2 host 10.0.16.11 echo
```

Only:

```text
PC1 → 10.0.16.11
```

was denied.

### Modified

```cisco
deny icmp host 10.0.18.2 any echo
```

Now:

```text
PC1 → ANY destination
```

is denied for ICMP Echo.

---

## 11. Why `any` Is Powerful

Compare:

```cisco
host 10.0.16.11
```

with:

```cisco
any
```

`host` means exactly one IP address.

```text
host 10.0.16.11
      ↓
10.0.16.11 only
```

`any` means any address.

```text
any
 ↓
10.0.0.0 → 255.255.255.255
```

So:

```cisco
deny icmp host 10.0.18.2 any echo
```

means:

> PC1 cannot send ICMP Echo requests to any destination.

---

## 12. Create an Exception

After creating the broad deny, you added:

```cisco
5 permit icmp host 10.0.18.2 host 10.0.16.11 echo
```

The final ACL became:

```cisco
5  permit icmp host 10.0.18.2 host 10.0.16.11 echo
10 deny icmp host 10.0.18.2 any echo
20 permit ip any any
```

This is the most important part of the lab.

---

## 13. Why Sequence 5 Is Before Sequence 10

ACLs use **first-match processing**.

For:

```text
PC1 → 10.0.16.11 → ICMP Echo
```

the router checks:

```text
Sequence 5
   ↓
MATCH
   ↓
PERMIT
```

It stops processing.

It never reaches:

```text
Sequence 10
```

Therefore the ping works.

---

For:

```text
PC1 → some other destination → ICMP Echo
```

sequence 5 doesn't match.

The router continues:

```text
Sequence 10
   ↓
MATCH
   ↓
DENY
```

Therefore:

```text
PC1 → Server 10.0.16.11     → PERMIT
PC1 → Other destinations     → DENY
```

---

3# 14. Final ACL Logic

Your final ACL:

```cisco
ip access-list extended Demo
 5 permit icmp host 10.0.18.2 host 10.0.16.11 echo
 10 deny icmp host 10.0.18.2 any echo
 20 permit ip any any
```

Can be visualized as:

```text
                 PC1
             10.0.18.2
                  |
                  |
             ICMP Echo
                  |
                  v
           +--------------+
           | Sequence 5   |
           | PC1 → Server |
           +--------------+
              /       \
            YES        NO
             |          |
          PERMIT        v
                    +-----------+
                    | Sequence 10|
                    | PC1 → ANY |
                    +-----------+
                       /     \
                     YES      NO
                      |        |
                    DENY       v
                           Sequence 20
                           permit IP any any
```

---

## 15. PC1 Configuration

PC1 showed:

```text
IPv4 Address:     10.0.18.2
Subnet Mask:      255.255.255.224
Default Gateway:  10.0.18.1
```

Therefore:

```text
Network:    10.0.18.0/27
PC1:        10.0.18.2
Gateway:    10.0.18.1
```

PC1 is correctly positioned in VLAN 10.

---

## 16. Server Configuration

Server/PC2 showed:

```text
IPv4 Address:     10.0.16.11
Subnet Mask:      255.255.255.128
Default Gateway:  10.0.16.1
```

So the lab traffic being tested is:

```text
10.0.18.2
    |
    | ICMP Echo
    ↓
10.0.16.11
```

---

## 17. Ping Testing

You tested:

```text
ping -t 10.0.16.11
```

The initial test showed replies such as:

```text
Reply from 10.0.16.11
```

After the ACL changes, you also observed periods of:

```text
Reply from 10.0.18.1: Destination host unreachable.
```

and later successful replies from:

```text
10.0.16.11
```

Your captured test therefore demonstrates that the lab was actively generating traffic while the ACL was being modified and verified. The supplied output alone does **not** establish a single definitive cause for the intermittent packet loss/unreachable responses, so that behavior should not be attributed solely to the ACL without further topology/interface verification. 

---

## 18. ACL Match Counters After Testing

The final verification showed:

```text
5  permit icmp host 10.0.18.2 host 10.0.16.11 echo (12 match(es))
10 deny icmp host 10.0.18.2 any echo (123 match(es))
20 permit ip any any (24 match(es))
```

This demonstrates that different traffic was hitting different ACL entries.

Especially important:

```text
Sequence 5 → 12 matches
Sequence 10 → 123 matches
Sequence 20 → 24 matches
```

The counters are cumulative for the ACL entries during the lab session.

---

## 19. Useful Commands Learned

### Check interfaces

```cisco
show ip interface brief
```

### Create named Extended ACL

```cisco
ip access-list extended Demo
```

### Remove an ACL entry

```cisco
no 10
```

### Add a numbered ACL entry

```cisco
5 permit icmp host 10.0.18.2 host 10.0.16.11 echo
```

### Apply ACL inbound

```cisco
interface gi0/0.10
ip access-group Demo in
```

### Verify ACL

```cisco
show ip access-lists
```

### Verify while inside configuration mode

```cisco
do show ip access-lists
```

---

## 20. What This Lab Demonstrated

| Concept                | What you practiced                        |
| ---------------------- | ----------------------------------------- |
| Named Extended ACL     | `ip access-list extended Demo`            |
| Sequence numbers       | `5`, `10`, `20`                           |
| ICMP filtering         | `icmp`                                    |
| ICMP Echo              | `echo`                                    |
| Host matching          | `host 10.0.18.2`                          |
| Any destination        | `any`                                     |
| ACL application        | `ip access-group Demo in`                 |
| ACL direction          | `in`                                      |
| First-match processing | Sequence 5 before 10                      |
| Exception              | Permit specific traffic before broad deny |
| Match counters         | `(12 match(es))`                          |
| Verification           | `show ip access-lists`                    |
| Testing                | `ping -t`                                 |

---

## 21. The Core Lesson

The most important configuration from this lab is:

```cisco
ip access-list extended Demo
 5 permit icmp host 10.0.18.2 host 10.0.16.11 echo
 10 deny icmp host 10.0.18.2 any echo
 20 permit ip any any
```

Think of it as:

```text
RULE 5
PC1 → Server → PING
       ↓
     ALLOW
       ↓
FIRST MATCH → STOP


RULE 10
PC1 → ANY → PING
       ↓
      DENY


RULE 20
Other IP traffic
       ↓
     ALLOW
```

### ⭐ Lab takeaway

> **A specific permit placed before a broader deny creates an ACL exception.**

That is the key skill demonstrated by this lab.
