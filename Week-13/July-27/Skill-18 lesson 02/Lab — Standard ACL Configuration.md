# Week 13 — July 27

## Lab Notes — Standard ACL Configuration

Source: your uploaded Packet Tracer lab output. 

This lab demonstrates the **complete lifecycle of a named Standard ACL**: creation → modification → verification → application → testing → removal.

---

## 1. Lab Topology / Addressing

From the router configuration:

| Item           | Value           |
| -------------- | --------------- |
| Router         | `cafe01-RT01`   |
| Admin VLAN     | VLAN 10         |
| Admin subnet   | `10.0.18.0/27`  |
| Admin gateway  | `10.0.18.1`     |
| PC1            | `10.0.18.2`     |
| PC2            | `10.0.18.3`     |
| Patron VLAN    | VLAN 20         |
| Patron subnet  | `10.0.18.32/27` |
| Patron gateway | `10.0.18.33`    |
| ACL            | `PC1_filter`    |
| ACL type       | Standard        |

PC1's Packet Tracer output confirms:

```text
IPv4 Address: 10.0.18.2
Subnet Mask: 255.255.255.224
Default Gateway: 10.0.18.1
```



---

# 2. Initial ACL

The router already had a numbered ACL:

```cisco
access-list 1 permit 10.0.18.0 0.0.0.63
```

This is **ACL 1**, used by NAT:

```cisco
ip nat inside source list 1 interface GigabitEthernet0/1 overload
```

So don't confuse:

```text
ACL 1
```

with:

```text
PC1_filter
```

They are two different ACLs.

### ACL 1

```text
10 permit 10.0.18.0 0.0.0.63
```

The wildcard `0.0.0.63` corresponds to `/26`.

---

# 3. Creating the Named Standard ACL

The lab creates:

```cisco
ip access-list standard PC1_filter
```

This moves you into:

```text
(config-std-nacl)#
```

Now you're editing the ACL itself.

---

# 4. First Version of the ACL

The ACL eventually becomes:

```cisco
10 deny host 10.0.18.2
20 permit 10.0.18.0 0.0.0.255
30 permit any
```

Then sequence 20 is removed:

```cisco
no 20
```

Leaving:

```cisco
10 deny host 10.0.18.2
30 permit any
```

This is much cleaner.

---

# 5. Adding Another Host

The lab demonstrates inserting a rule using a specific sequence number:

```cisco
12 deny host 10.0.18.3
```

The ACL becomes:

```text
10 deny host 10.0.18.2
12 deny host 10.0.18.3
30 permit any
```

Notice something important:

```text
10
12
30
```

The sequence numbers don't have to be consecutive.

This lets you insert rules later.

---

# 6. Removing a Specific ACL Entry

To remove sequence 12:

```cisco
no 12
```

Result:

```text
10 deny host 10.0.18.2
30 permit any
```

This is the advantage of sequence numbers.

You don't have to delete and recreate the entire ACL just to remove one rule.

---

# 7. ACL Remarks

The lab also demonstrates:

```cisco
remark "don't deny--this is only for commenting the filter group"
```

A remark is documentation.

It does **not** permit or deny traffic.

Think:

```text
remark = documentation
deny/permit = actual policy
```

One detail visible in your output: the remark appears in `show running-config`, but it isn't displayed by `show access-lists` in this Packet Tracer output. 

---

# 8. Final ACL Before Application

The useful ACL is:

```cisco
ip access-list standard PC1_filter
 10 deny host 10.0.18.2
 30 permit any
```

Meaning:

```text
Source = 10.0.18.2
        ↓
      DENY

Anything else
        ↓
     PERMIT
```

This is the classic Standard ACL example:

> **Block one host while allowing everyone else.**

---

# 9. Why `permit any` Is Critical

If you only configured:

```cisco
10 deny host 10.0.18.2
```

then traffic from every other source would eventually encounter the implicit:

```text
deny any
```

Therefore:

```text
PC1
 ↓
DENY

Everyone else
 ↓
implicit DENY
```

But your final ACL has:

```cisco
10 deny host 10.0.18.2
30 permit any
```

So:

```text
PC1       → DENY
Everyone else → PERMIT
```

---

# 10. Applying the ACL

The lab enters:

```cisco
interface GigabitEthernet0/0.10
```

This is the **Admin VLAN 10 subinterface**.

Then:

```cisco
ip access-group PC1_filter in
```

Therefore:

```text
PC1
10.0.18.2
   |
   | traffic enters router
   v
Gi0/0.10
   |
   | INBOUND ACL
   v
PC1_filter
```

The direction is:

```text
IN
```

because traffic from PC1 is **entering the router through Gi0/0.10**.

---

# 11. The Most Important Mental Model

Don't think:

> "PC1 is inbound."

Think:

> **"The packet is entering the router through Gi0/0.10."**

### Inbound

```text
          ROUTER
            ↑
            |
       Gi0/0.10
            ↑
           PC1
```

### Outbound

```text
          ROUTER
            |
            ↓
       Gi0/0.10
            ↓
           PC1
```

The `in`/`out` keyword describes the packet's direction **relative to the interface/router**, not the user's intention.

---

# 12. Testing the ACL

The lab uses:

```text
C:\>ping -t 10.0.18.1
```

PC1:

```text
10.0.18.2
```

Gateway:

```text
10.0.18.1
```

Since the ACL says:

```cisco
deny host 10.0.18.2
```

the packet from PC1 matches sequence 10.

Therefore the router denies it.

---

# 13. ACL Match Counters

This is an excellent part of your lab.

You got:

```text
10 deny host 10.0.18.2 (18 match(es))
```

Then:

```text
(19 match(es))
```

Then:

```text
(20 match(es))
```

Then:

```text
(24 match(es))
```

Those counters show that traffic is actually hitting that ACL entry.

So:

```text
PC1 sends traffic
       ↓
ACL evaluates packet
       ↓
Source = 10.0.18.2
       ↓
Sequence 10 matches
       ↓
DENY
       ↓
match counter increases
```

This makes `show access-lists` extremely useful for troubleshooting.

---

# 14. Why You Saw `Destination host unreachable`

While the ACL was applied, your continuous ping eventually produced:

```text
Reply from 10.0.18.1: Destination host unreachable.
```

The important observation is that this occurred while the ACL was filtering traffic.

After the ACL was removed, successful replies resumed:

```text
Reply from 10.0.18.1: bytes=32 time<1ms TTL=255
```

So the lab demonstrates the immediate operational effect of applying/removing an ACL. 

---

# 15. Important Mistake in the Lab: Removing the ACL

You attempted:

```cisco
cafe01-RT01(config)#no ip access-group PC1_filter in
```

and received:

```text
% Invalid input detected
```

### Why?

You were at:

```text
(config)#
```

But `ip access-group` is an **interface-level command**.

You need to enter the interface first:

```cisco
conf t
interface GigabitEthernet0/0.10
no ip access-group PC1_filter in
```

Then the ACL is detached from the interface.

---

# 16. Don't Confuse These Two Commands

### Remove ACL from an interface

```cisco
interface GigabitEthernet0/0.10
no ip access-group PC1_filter in
```

This means:

> **Stop applying this ACL here.**

### Delete the ACL itself

```cisco
no ip access-list standard PC1_filter
```

This means:

> **Delete the ACL configuration.**

These are different operations.

---

# 17. The Correct Production Workflow

Your lab demonstrates why you should separate these operations.

### To completely remove an ACL safely:

```cisco
conf t
interface GigabitEthernet0/0.10
no ip access-group PC1_filter in
exit
no ip access-list standard PC1_filter
end
```

Conceptually:

```text
ACL applied
    ↓
Detach ACL
    ↓
Verify
    ↓
Delete ACL
```

Don't casually delete a security policy before determining where it is being used.

---

# 18. Recreating the ACL

After deleting it:

```cisco
no ip access-list standard PC1_filter
```

you recreate it:

```cisco
ip access-list standard PC1_filter
```

Initially:

```text
Standard IP access list PC1_filter
```

contains no rules.

Then you add:

```cisco
10 deny host 10.0.18.2
30 permit any
```

The final output confirms:

```text
Standard IP access list PC1_filter
    10 deny host 10.0.18.2
    30 permit any
```



---

# 19. One Subtle Detail: Your Admin Subnet

PC1 has:

```text
10.0.18.2/27
```

Therefore the actual Admin VLAN 10 subnet is:

```text
Network:    10.0.18.0/27
Mask:       255.255.255.224
Broadcast:  10.0.18.31
Usable:     10.0.18.1 - 10.0.18.30
```

So:

```text
10.0.18.2
```

is correctly inside VLAN 10.

Your DHCP configuration confirms:

```cisco
ip dhcp pool ADMIN-10
 network 10.0.18.0 255.255.255.224
 default-router 10.0.18.1
```



---

# 20. Important Wildcard Mask Observation

For the `/27` Admin network:

```text
Subnet mask:
255.255.255.224
```

the corresponding wildcard is:

```text
0.0.0.31
```

So if you wanted to match the entire Admin subnet, you'd use:

```cisco
permit 10.0.18.0 0.0.0.31
```

Your NAT ACL is different:

```cisco
access-list 1 permit 10.0.18.0 0.0.0.63
```

because `0.0.0.63` represents `/26` and covers:

```text
10.0.18.0 - 10.0.18.63
```

which encompasses both your `/27` Admin and `/27` Patron networks.

That's a useful detail to notice when reading your running configuration.

---

# 21. What This Lab Actually Taught You

### ACL creation

```cisco
ip access-list standard PC1_filter
```

### Host matching

```cisco
deny host 10.0.18.2
```

### Sequence numbers

```cisco
10 deny ...
12 deny ...
30 permit ...
```

### Removing an individual rule

```cisco
no 12
```

### Documentation

```cisco
remark "..."
```

### Allowing everything else

```cisco
permit any
```

### Applying an ACL

```cisco
ip access-group PC1_filter in
```

### Checking the ACL

```cisco
show access-lists
```

### Removing an ACL from an interface

```cisco
no ip access-group PC1_filter in
```

### Deleting the ACL itself

```cisco
no ip access-list standard PC1_filter
```

---

# 22. Commands Worth Memorizing

```cisco
! Create
ip access-list standard PC1_filter

! Match one host
10 deny host 10.0.18.2

! Allow everyone else
30 permit any

! Apply to interface
interface GigabitEthernet0/0.10
ip access-group PC1_filter in

! Verify
show access-lists

! Remove individual rule
ip access-list standard PC1_filter
no 10

! Remove ACL from interface
interface GigabitEthernet0/0.10
no ip access-group PC1_filter in

! Delete ACL
no ip access-list standard PC1_filter
```

---

## Final mental model

```text
             STANDARD ACL
                  |
                  v
             SOURCE IP
                  |
          +-------+-------+
          |               |
       MATCH            NO MATCH
          |               |
          v               v
    PERMIT / DENY    Next ACL entry
                          |
                          v
                   Nothing matches
                          |
                          v
                    IMPLICIT DENY
```

And for placement:

```text
STANDARD ACL
     |
     | Can inspect
     v
SOURCE IP ONLY
     |
     | Therefore
     v
Place generally
near DESTINATION
```

### The biggest lesson from your actual lab

**Creating an ACL is the easy part. Safely controlling its scope is the real skill.**

You demonstrated all four pieces:

```text
BUILD
  ↓
MODIFY
  ↓
APPLY
  ↓
VERIFY / REMOVE
```

And your match counters plus ping test showed that the ACL was **actually affecting live packet forwarding**, not merely sitting in the configuration. 
