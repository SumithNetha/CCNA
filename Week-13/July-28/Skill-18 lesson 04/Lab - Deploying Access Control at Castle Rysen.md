## Lab Notes — Deploying Access Control at Castle Rysen

Your configuration is implementing **two separate access-control policies**: one for **Patron → Admin/Plex traffic**, and one for **SSH/Telnet management access**.

---

# 1. Extended ACL — `CAFE_FILTER`

### Create the Extended ACL

```cisco
conf t
ip access-list extended CAFE_FILTER
```

This enters:

```text
(config-ext-nacl)#
```

From here you can add:

* `permit`
* `deny`
* `remark`
* sequence numbers
* `no`

---

## 2. Patron and Admin Networks

From your interface configuration:

```text
Gi0/0.10 = 10.0.18.1
Gi0/0.20 = 10.0.18.33
```

The ACL uses:

```text
10.0.18.32 0.0.0.31
```

as the **Patron VLAN**.

This corresponds to:

```text
10.0.18.32/27
```

Range:

```text
Network:    10.0.18.32
Usable:     10.0.18.33 - 10.0.18.62
Broadcast:  10.0.18.63
```

The Admin network is:

```text
10.0.18.0 0.0.0.31
```

which corresponds to:

```text
10.0.18.0/27
```

Range:

```text
Network:    10.0.18.0
Usable:     10.0.18.1 - 10.0.18.30
Broadcast:  10.0.18.31
```

So the ACL is essentially:

```text
Patrons: 10.0.18.32/27
Admin:   10.0.18.0/27
```

---

# 3. Permit Plex Traffic

You initially entered:

```cisco
permit tcp 10.0.18.32 0.0.0.31 10.0.18.0 0.0.0.31 443
```

and received:

```text
% Invalid input detected
```

The reason is that Extended ACL syntax requires you to specify how the port should be matched.

You discovered this with:

```cisco
?
```

which showed:

```text
eq
established
gt
lt
neq
range
```

For an exact port, use:

```cisco
eq
```

Therefore:

```cisco
permit tcp 10.0.18.32 0.0.0.31 10.0.18.0 0.0.0.31 eq 443
```

---

## 4. Final Plex Permit Entries

Your completed ACL contains:

```cisco
10 permit tcp 10.0.18.32 0.0.0.31 10.0.18.0 0.0.0.31 eq 443
20 permit tcp 10.0.18.32 0.0.0.31 10.0.18.0 0.0.0.31 eq 32400
30 permit tcp 10.0.18.32 0.0.0.31 10.0.18.0 0.0.0.31 eq 32469
40 permit udp 10.0.18.32 0.0.0.31 10.0.18.0 0.0.0.31 eq 1900
50 permit udp 10.0.18.32 0.0.0.31 10.0.18.0 0.0.0.31 eq 5353
```

Notice your first attempt used:

```text
32461
```

but you removed it:

```cisco
no permit tcp ... eq 32461
```

and correctly replaced it with:

```cisco
30 permit tcp ... eq 32469
```

This matches the RFP requirement for Plex access. 

---

# 5. Deny Other Patron → Admin Traffic

After the five specific permits:

```cisco
60 deny ip 10.0.18.32 0.0.0.31 10.0.18.0 0.0.0.31
```

This is the critical security boundary.

It means:

```text
10.0.18.32/27
        |
        |  Anything else
        v
10.0.18.0/27
        |
       DENY
```

Because the five Plex entries come first, the permitted Plex traffic gets through.

Everything else from Patron → Admin hits sequence 60.

---

# 6. Restore Other Connectivity

You then added:

```cisco
70 permit ip any any
```

This is important.

Without it, traffic that doesn't match the five Plex permits or the explicit deny would eventually encounter the **implicit deny**.

With:

```cisco
70 permit ip any any
```

the design becomes:

```text
Plex required traffic
        ↓
      ALLOW
        ↓
Other Patron → Admin traffic
        ↓
      DENY
        ↓
Patron → other destinations
        ↓
      ALLOW
```

So Internet access isn't accidentally destroyed.

---

# 7. Verify the ACL

You used:

```cisco
do show acc
```

and obtained:

```text
Extended IP access list CAFE_FILTER
    10 permit tcp ... eq 443
    20 permit tcp ... eq 32400
    30 permit tcp ... eq 32469
    40 permit udp ... eq 1900
    50 permit udp ... eq 5353
    60 deny ip ...
    70 permit ip any any
```

This is the correct logical order.

---

# 8. Applying the ACL to the Correct Interface

You first removed an existing ACL:

```cisco
int gi0/0.10
no ip access-group Demo in
```

Then you applied `CAFE_FILTER` to:

```cisco
int gi0/0.20
```

with:

```cisco
ip access-group CAFE_FILTER in
```

Your configuration therefore became:

```text
Gi0/0.20
10.0.18.33
Patron VLAN
      │
      │ inbound
      ▼
CAFE_FILTER
```

This is significant because the ACL should inspect the traffic **when it enters the router from the Patron VLAN**.

---

# 9. ACL Match Counters — Very Important

Your output eventually showed:

```text
60 deny ip ... (52 match(es))
70 permit ip any any
```

and later:

```text
60 deny ip ... (44 match(es))
```

and:

```text
30 permit tcp ... eq 32469 (1 match(es))
```

These counters show that traffic is actually hitting the ACL entries.

For example:

```text
30 permit ... eq 32469 (1 match(es))
```

means at least one packet matched that ACE.

Similarly:

```text
60 deny ... (52 match(es))
```

means 52 packets have matched the deny ACE.

This is extremely useful when troubleshooting.

---

# 10. Your Ping Tests

You tested:

```text
ping 10.0.18.6
```

and initially got:

```text
Request timed out.
```

That makes sense for traffic that isn't explicitly permitted between the Patron and Admin networks.

Why?

Ping uses:

```text
ICMP
```

Your ACL permits:

```text
TCP 443
TCP 32400
TCP 32469
UDP 1900
UDP 5353
```

but does **not** permit ICMP from Patron → Admin.

Therefore:

```text
Patron
  |
  | ICMP
  X
Admin
```

is denied by sequence 60.

---

# 11. Ping to `10.0.18.33`

You tested:

```text
ping 10.0.18.33
```

and received replies.

`10.0.18.33` is the router's own interface address on the Patron subnet.

So this verifies that the Patron device can reach its local Layer-3 gateway:

```text
Patron PC
   |
   | VLAN 20
   |
10.0.18.33
Router
```

It does **not** prove that Patron → Admin traffic is permitted.

That's an important troubleshooting distinction.

---

# 12. Ping to `10.0.18.6`

You eventually saw successful replies:

```text
Reply from 10.0.18.6
```

followed by:

```text
Reply from 10.0.18.33: Destination host unreachable.
```

and approximately:

```text
Sent = 76
Received = 48
Lost = 28
37% loss
```

This is consistent with the ACL counter activity changing as traffic was tested, but **the pasted output alone doesn't establish the exact reason for the later intermittent behavior**. The ACL clearly is filtering Patron → Admin traffic, but the output doesn't give enough evidence to attribute every packet loss event solely to the ACL.

For your lab notes, the important demonstrated behavior is:

> ACL filtering was active and its ACE match counters increased during testing.

---

# 13. Telnet Testing

You tested several ports against:

```text
10.0.18.6
```

### TCP 443

```text
telnet 10.0.18.6 443
```

Result:

```text
Trying 10.0.18.6 ...Open
[Connection ... closed by foreign host]
```

This is significant.

The ACL permitted:

```text
TCP/443
```

and the connection reached the destination.

The remote host then closed the connection.

So:

```text
Patron
   |
TCP 443
   |
   ▼
ACL → PERMIT
   |
   ▼
10.0.18.6
   |
   └── closes connection
```

The ACL isn't necessarily the thing closing the session.

---

### UDP 1900

You attempted:

```text
telnet 10.0.18.6 1900
```

and got:

```text
Connection timed out
```

However, remember:

**Telnet tests TCP connections.**

Your ACL permits UDP/1900, not TCP/1900.

Therefore, Telnet is **not an appropriate test for the UDP/1900 ACE**.

This is an important lab lesson.

---

### UDP 5353

Same situation:

```text
telnet 10.0.18.6 5353
```

is testing TCP/5353, while your ACL allows:

```text
UDP/5353
```

So the Telnet timeout doesn't prove that the UDP/5353 ACL entry is failing.

---

### TCP 32469

You tested:

```text
telnet 10.0.18.6 32469
```

and got:

```text
Connection refused by remote host
```

This indicates the TCP connection reached the destination far enough for the remote host to refuse it; again, that is different from an ACL silently dropping the traffic.

---

# 14. Standard ACL — `ADMIN_LIMIT`

The second part of the lab protects remote device management.

You created:

```cisco
ip access-list standard ADMIN_LIMIT
```

Then attempted:

```cisco
permit 10.0.18.0 0.0.31
```

which failed.

You corrected it to:

```cisco
permit 10.0.18.0 0.0.0.31
```

This permits:

```text
10.0.18.0/27
```

Then:

```cisco
permit 10.0.16.0 0.0.0.127
```

permits:

```text
10.0.16.0/25
```

So your Standard ACL became:

```text
ADMIN_LIMIT

10 permit 10.0.18.0 0.0.0.31
20 permit 10.0.16.0 0.0.0.127
```

---

# 15. Applying `ADMIN_LIMIT` to VTY

You configured:

```cisco
line vty 0 4
access-class ADMIN_LIMIT in
```

This is the key command for the management restriction.

The traffic flow is:

```text
Remote Management Device
          |
          | SSH / Telnet
          ↓
       VTY Lines
          |
          ↓
 ADMIN_LIMIT ACL
          |
     ┌────┴────┐
     │         │
  Allowed    Denied
     │         │
     ▼         X
 Device
```

Only the permitted source networks can establish remote VTY sessions.

---

# 16. Final Lab Configuration

### Extended ACL

```cisco
ip access-list extended CAFE_FILTER
 permit tcp 10.0.18.32 0.0.0.31 10.0.18.0 0.0.0.31 eq 443
 permit tcp 10.0.18.32 0.0.0.31 10.0.18.0 0.0.0.31 eq 32400
 permit tcp 10.0.18.32 0.0.0.31 10.0.18.0 0.0.0.31 eq 32469
 permit udp 10.0.18.32 0.0.0.31 10.0.18.0 0.0.0.31 eq 1900
 permit udp 10.0.18.32 0.0.0.31 10.0.18.0 0.0.0.31 eq 5353
 deny ip 10.0.18.32 0.0.0.31 10.0.18.0 0.0.0.31
 permit ip any any
```

Applied:

```cisco
interface GigabitEthernet0/0.20
 ip access-group CAFE_FILTER in
```

### Management ACL

```cisco
ip access-list standard ADMIN_LIMIT
 permit 10.0.18.0 0.0.0.31
 permit 10.0.16.0 0.0.0.127
```

Applied:

```cisco
line vty 0 4
 access-class ADMIN_LIMIT in
```

---

## 🧠 What this lab actually taught you

```text
                ACL DEPLOYMENT
                     │
        ┌────────────┴────────────┐
        │                         │
  User Traffic              Device Access
        │                         │
 Extended ACL              Standard ACL
        │                         │
Patron → Admin             Who can manage?
        │                         │
Specific ports             Source subnet
        │                         │
Interface ACL              VTY access-class
```

### The most important operational lesson

**Don't just check whether an ACL exists. Check whether it is doing what you intended.**

Your:

```cisco
show access-lists
```

output showed **match counters increasing**, which is evidence that packets were actually being evaluated against the configured ACEs.

And your lab demonstrated an especially useful troubleshooting distinction:

> **A connection being refused or timing out is not automatically proof that an ACL caused it.**

You have to consider the **protocol, port, ACL match, destination service, and return path** together.
