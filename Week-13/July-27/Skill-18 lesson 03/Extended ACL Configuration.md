# Extended ACL Configuration — Git Notes

## 1. Extended ACL Overview

An **Extended Access Control List (Extended ACL)** provides much more granular traffic filtering than a Standard ACL.

A Standard ACL mainly asks:

> **Who is sending the traffic?**

An Extended ACL can ask:

> **What protocol is being used, who is sending it, and where is it going?**

The core mental model is:

```text
Extended ACL
     |
     +-- Protocol
     +-- Source
     +-- Destination
     +-- Optional: Port / ICMP type
```

The three fundamental decisions are:

1. **Protocol** — What type of traffic?
2. **Source** — Who is sending it?
3. **Destination** — Where is it going?

Port numbers and ICMP message types can provide additional precision.

---

# 2. Standard ACL vs Extended ACL

| Feature           | Standard ACL     | Extended ACL |
| ----------------- | ---------------- | ------------ |
| Source IP         | ✅                | ✅            |
| Destination IP    | ❌                | ✅            |
| Protocol          | ❌                | ✅            |
| TCP/UDP ports     | ❌                | ✅            |
| ICMP filtering    | Limited          | ✅            |
| Granularity       | Low              | High         |
| Typical placement | Near destination | Near source  |

### Mental model

```text
STANDARD ACL
     ↓
   SOURCE
    "WHO?"
```

```text
EXTENDED ACL
     ↓
PROTOCOL + SOURCE + DESTINATION
     ↓
"WHAT + WHO + WHERE?"
```

---

# 3. Protocol

The first major component of an Extended ACL is the **protocol**.

Common protocols include:

```text
TCP
UDP
ICMP
IP
```

---

## TCP

**TCP (Transmission Control Protocol)** is connection-oriented and provides reliable delivery.

Common TCP-based applications include:

* HTTP
* HTTPS
* SSH
* FTP
* Telnet
* SMTP
* POP3
* IMAP

Example:

```cisco
permit tcp ...
```

This means the ACL is matching TCP traffic.

---

## UDP

**UDP (User Datagram Protocol)** is connectionless and does not provide TCP-style reliability.

Common examples include:

* DNS
* Voice traffic
* Video traffic
* Gaming traffic

Example:

```cisco
permit udp ...
```

---

## ICMP

**ICMP (Internet Control Message Protocol)** is used for network control and diagnostic messaging.

The classic example is:

```text
ping
```

An Extended ACL can specifically permit or deny ICMP traffic.

Example:

```cisco
deny icmp ...
```

This allows you to block ping without necessarily blocking other traffic.

---

## IP

`ip` represents IP traffic broadly.

For example:

```cisco
permit ip any any
```

means:

> Permit IP traffic from any source to any destination.

This includes IP-based traffic such as:

```text
TCP
UDP
ICMP
```

It can therefore be used as a broad catch-all permit after a more specific deny.

---

# 4. Source

After choosing the protocol, identify the **source**.

The source represents:

> **Where is the traffic coming from?**

The source can be:

* A single host
* A network
* `any`

### Specific host

```cisco
host 10.0.18.2
```

### Network

A network can be specified with a wildcard mask.

Example:

```cisco
10.0.18.0 0.0.0.31
```

### Any source

```cisco
any
```

means:

> I don't care which source generated the packet.

---

# 5. Destination

The destination represents:

> **Where is the traffic going?**

Like the source, the destination can be:

* A specific host
* A network
* `any`

Example:

```cisco
host 10.0.18.100
```

means the destination must be exactly:

```text
10.0.18.100
```

---

# 6. Basic Extended ACL Structure

Conceptually, an Extended ACL statement looks like:

```text
ACTION
  ↓
PROTOCOL
  ↓
SOURCE
  ↓
DESTINATION
  ↓
OPTIONAL PORT / TYPE
```

Example:

```cisco
deny icmp host 10.0.18.2 any
```

Read it as:

> **Deny ICMP traffic from host 10.0.18.2 to any destination.**

This is much more precise than a Standard ACL.

---

# 7. Example: Blocking Ping

Suppose:

```text
Source = 10.0.18.2
```

You want to prevent this host from pinging anywhere.

Use:

```cisco
deny icmp host 10.0.18.2 any
```

This means:

```text
Protocol = ICMP
Source   = 10.0.18.2
Destination = any
Action = DENY
```

Importantly, this does **not** mean:

> Block all traffic from 10.0.18.2.

It means:

> Block **ICMP** traffic from 10.0.18.2.

TCP and UDP traffic can still be handled by subsequent ACL rules.

---

# 8. Extended ACLs Can Be Surgical

This is the biggest advantage of Extended ACLs.

You can have different rules for the same source.

For example:

```text
10.0.18.2 → ping → DENY
10.0.18.2 → web → PERMIT
10.0.18.2 → DNS → PERMIT
```

The source is the same, but the protocol/service is different.

This level of control isn't possible with a Standard ACL.

---

# 9. Port Numbers

Extended ACLs can inspect **TCP and UDP port numbers**.

This allows you to filter specific applications/services.

Common examples:

| Application | Protocol | Port |
| ----------- | -------- | ---: |
| HTTP        | TCP      |   80 |
| HTTPS       | TCP      |  443 |
| FTP         | TCP      |   21 |
| Telnet      | TCP      |   23 |
| SSH         | TCP      |   22 |
| SMTP        | TCP      |   25 |
| POP3        | TCP      |  110 |
| IMAP        | TCP      |  143 |
| DNS         | UDP      |   53 |

Knowing the protocol/port mapping is important when translating an application requirement into an ACL rule.

---

# 10. Destination Port vs Source Port

This is one of the most important Extended ACL concepts.

Suppose:

```text
Client → Web Server
```

The client might use:

```text
Source port = random ephemeral port
```

while the server listens on:

```text
Destination port = 80
```

for HTTP.

So if the requirement is:

> Allow a client to access an HTTP server.

You generally match:

```text
TCP
Destination port 80
```

not:

```text
Source port 80
```

### Think about the conversation

```text
CLIENT                         SERVER
10.0.18.2                      10.0.18.100

Source Port       →            Destination Port
random ephemeral               80
```

Therefore:

```text
Client → Server
        TCP
        destination port 80
```

means:

> The client is trying to reach the server's HTTP service.

---

# 11. `eq` — Exact Port Matching

Extended ACLs can use `eq` to match an exact port.

Conceptually:

```cisco
tcp ... eq 80
```

means:

> Match TCP traffic using port 80 at that position.

For a web server, the important idea is:

```text
TCP → destination port 80
```

Similarly:

```text
TCP → destination port 443
```

represents HTTPS.

---

# 12. Common Service Logic

### HTTP

```text
TCP
Destination port 80
```

### HTTPS

```text
TCP
Destination port 443
```

### SSH

```text
TCP
Destination port 22
```

### Telnet

```text
TCP
Destination port 23
```

### DNS

```text
UDP
Destination port 53
```

The important skill is translating:

```text
"Allow HTTPS"
```

into:

```text
TCP
destination port 443
```

---

# 13. ICMP Is Different

ICMP doesn't use TCP or UDP ports.

Therefore, don't think:

```text
ICMP port 80
ICMP port 443
```

That doesn't make sense.

ICMP has **message types**, such as:

```text
echo
echo-reply
```

which are associated with ping.

Extended ACLs can therefore be more specific with ICMP traffic by matching ICMP types.

---

# 14. `ip any any`

A very useful broad rule is:

```cisco
permit ip any any
```

This means:

> Permit IP traffic from any source to any destination.

It can be useful after specific deny rules.

Example:

```text
deny icmp host 10.0.18.2 any
permit ip any any
```

The logic becomes:

```text
10.0.18.2 ICMP → DENY
Everything else → PERMIT
```

Without the broader permit, unmatched traffic can eventually hit the ACL's implicit deny.

---

# 15. Implicit Deny Still Applies

Extended ACLs also have the implicit deny behavior.

If an ACL ends without a matching permit, unmatched traffic is denied.

Example:

```text
10 deny icmp host 10.0.18.2 any
```

The effective behavior is approximately:

```text
ICMP from 10.0.18.2 → DENY
Everything else → implicit DENY
```

If the intention is to block only that ICMP traffic while allowing other IP traffic:

```text
10 deny icmp host 10.0.18.2 any
20 permit ip any any
```

---

# 16. First Match Wins

Extended ACLs are processed:

> **Top to bottom, one rule at a time.**

The first matching entry determines the action.

Example:

```text
10 permit ip any any
20 deny tcp host 10.0.18.2 any eq 80
```

The deny rule will never be reached because:

```text
permit ip any any
```

already matches the traffic.

Therefore, more specific rules should generally be placed before broader rules.

Correct:

```text
10 deny tcp host 10.0.18.2 any eq 80
20 permit ip any any
```

---

# 17. Reading an Extended ACL Like a Sentence

A useful troubleshooting technique is to read each rule as a sentence.

For example:

```text
deny tcp host 10.0.18.2 host 10.0.18.100 eq 443
```

Read it as:

> **Deny TCP from host 10.0.18.2 to host 10.0.18.100 on destination port 443.**

Break it down:

```text
deny
 ↓
tcp
 ↓
source = 10.0.18.2
 ↓
destination = 10.0.18.100
 ↓
destination port = 443
```

If the sentence doesn't describe the intended policy, the ACL probably doesn't either.

---

# 18. Common Extended ACL Mistakes

## Mistake 1 — Wrong source

A typo in the source address can cause the ACL to match the wrong traffic.

---

## Mistake 2 — Wrong destination

The ACL may technically be valid but affect a different server/network than intended.

---

## Mistake 3 — Wrong protocol

For example:

```text
deny udp
```

will not block TCP traffic.

---

## Mistake 4 — Wrong port

HTTPS is:

```text
TCP 443
```

not UDP 443.

---

## Mistake 5 — Putting the port on the wrong side

For client-to-server web traffic:

```text
client source port = ephemeral
server destination port = 80/443
```

Therefore, the service port normally belongs to the **destination side** of the ACL entry.

---

## Mistake 6 — Forgetting implicit deny

A rule such as:

```text
deny icmp host X any
```

can unintentionally deny everything else if there isn't an appropriate permit afterward.

---

## Mistake 7 — Incorrect rule order

A broad rule can prevent a later specific rule from ever being evaluated.

---

# 19. Extended ACL Placement

The general rule is:

> **Place Extended ACLs as close to the source as possible.**

Why?

Extended ACLs are highly specific.

They can identify:

```text
Source
Destination
Protocol
Port
```

Therefore, unwanted traffic can be stopped early.

Example:

```text
CLIENT
  |
  | unwanted traffic
  v
[EXTENDED ACL]
  |
  X  DROP
```

Instead of allowing unwanted traffic to travel across the network before finally blocking it.

---

# 20. Standard vs Extended Placement

This is a major CCNA comparison.

### Standard ACL

```text
SOURCE ────────────────> DESTINATION
                          ^
                          |
                    ACL preferably here
```

Generally:

> **Standard ACL → close to destination**

Because Standard ACLs only know the source.

### Extended ACL

```text
SOURCE
  ^
  |
 ACL preferably here
  |
  +──────────────> DESTINATION
```

Generally:

> **Extended ACL → close to source**

Because Extended ACLs can make precise decisions.

---

# 21. Named Extended ACLs

Named ACLs make large policies easier to understand and maintain.

Conceptually:

```cisco
ip access-list extended WEB-FILTER
```

Then rules can be added individually.

For example:

```text
10 deny ...
20 permit ...
30 permit ...
```

A name such as:

```text
WEB-FILTER
```

communicates more information than simply:

```text
access-list 101
```

---

# 22. Sequence Numbers

Named Extended ACLs can use sequence numbers.

Example:

```text
10 deny icmp host 10.0.18.2 any
20 permit tcp any any eq 443
30 permit ip any any
```

You can modify an individual entry rather than rebuilding the entire ACL.

For example:

```cisco
no 20
```

removes sequence 20.

This is particularly useful as ACLs become larger.

---

# 23. Remarks

Use `remark` to document the purpose of rules.

Example:

```cisco
remark Block PC1 ping traffic
```

Remarks do not affect packet forwarding.

They exist for:

* Documentation
* Troubleshooting
* Change management
* Future administrators

A good ACL should explain **why** a security rule exists, not merely what it does.

---

# 24. Extended ACL Design Process

Before writing the command, identify:

### Step 1 — Action

```text
permit or deny?
```

### Step 2 — Protocol

```text
TCP?
UDP?
ICMP?
IP?
```

### Step 3 — Source

```text
Which host/network?
```

### Step 4 — Destination

```text
Which host/network?
```

### Step 5 — Port/type

If required:

```text
80?
443?
22?
53?
ICMP echo?
```

### Step 6 — Rule order

Ask:

> Could an earlier rule already match this traffic?

### Step 7 — Default behavior

Ask:

> What happens to traffic that doesn't match?

Remember:

```text
Implicit deny
```

---

# 25. Example Design Exercise

Requirement:

> PC1 should not be able to ping the server, but it should still be able to access other IP services.

Translate the requirement:

```text
Action       = deny
Protocol     = ICMP
Source       = PC1
Destination  = Server
```

Conceptually:

```cisco
deny icmp host PC1 host SERVER
```

Then allow other IP traffic if appropriate:

```cisco
permit ip any any
```

The ACL now expresses:

```text
PC1 → ICMP → Server = DENY
Everything else     = PERMIT
```

This is much more precise than a Standard ACL.

---

# 26. Example: Allow HTTPS but Not HTTP

Requirement:

> A source should be allowed to access HTTPS but not HTTP.

Translate:

```text
HTTPS = TCP 443
HTTP  = TCP 80
```

The ACL logic becomes:

```text
TCP → destination port 80  → DENY
TCP → destination port 443 → PERMIT
```

The key is that the ACL isn't simply saying:

> "This host is denied."

It is saying:

> "This host is denied **this particular application/service**."

That is the power of Extended ACLs.

---

# 27. Example: Restrict SSH

Requirement:

> Only authorized administrators should access a device using SSH.

Translate:

```text
Protocol = TCP
Destination service = SSH
Port = 22
Source = authorized administrator network
```

Conceptually:

```text
Admin network
     |
     | TCP/22
     v
Network device
     |
   PERMIT
```

Other sources attempting TCP/22 can be denied.

---

# 28. Extended ACL and Business Requirements

Extended ACLs are particularly useful when a requirement says things like:

```text
Allow HTTPS
Deny HTTP
Allow SSH
Deny Telnet
Allow DNS
Block ping
Allow access to this server
Block access to that server
```

These requirements contain **protocol, destination, or service information**, making Extended ACLs appropriate.

---

# 29. Troubleshooting Extended ACLs

When traffic is unexpectedly denied, work through the policy systematically.

### 1. Is the ACL applied?

An ACL that exists but isn't applied won't filter the traffic.

### 2. Is it applied in the correct direction?

```text
in
out
```

Think from the router's perspective.

### 3. What is the actual source IP?

Make sure the packet has the source address you expect.

### 4. What is the destination IP?

Make sure the destination matches the ACL.

### 5. What protocol is being used?

```text
TCP?
UDP?
ICMP?
```

### 6. What port is actually being used?

Especially important for application filtering.

### 7. Which rule matches first?

Remember:

> **First match wins.**

### 8. Is the traffic falling into the implicit deny?

If nothing permits it, it can be denied at the end.

---

# 30. Common Protocol and Port Reference

| Service | Protocol | Port / Type |
| ------- | -------- | ----------- |
| HTTP    | TCP      | 80          |
| HTTPS   | TCP      | 443         |
| FTP     | TCP      | 21          |
| Telnet  | TCP      | 23          |
| SSH     | TCP      | 22          |
| SMTP    | TCP      | 25          |
| POP3    | TCP      | 110         |
| IMAP    | TCP      | 143         |
| DNS     | UDP      | 53          |
| Ping    | ICMP     | Echo        |

The key is not memorizing ports randomly.

Associate:

```text
Application
    ↓
Protocol
    ↓
Port
```

For example:

```text
HTTPS
 ↓
TCP
 ↓
443
```

---

# 31. Standard ACL vs Extended ACL — Final Comparison

```text
STANDARD ACL
─────────────
Source IP
    ↓
Simple
    ↓
Less precise
    ↓
Near destination
```

```text
EXTENDED ACL
─────────────
Protocol
    +
Source
    +
Destination
    +
Port / ICMP type
    ↓
Precise
    ↓
Near source
```

---

# 32. Key Commands

### Create named Extended ACL

```cisco
ip access-list extended NAME
```

### Permit TCP

```cisco
permit tcp ...
```

### Deny TCP

```cisco
deny tcp ...
```

### Permit UDP

```cisco
permit udp ...
```

### Deny UDP

```cisco
deny udp ...
```

### Permit ICMP

```cisco
permit icmp ...
```

### Deny ICMP

```cisco
deny icmp ...
```

### Permit all IP traffic

```cisco
permit ip any any
```

### Match a host

```cisco
host X.X.X.X
```

### Match any source/destination

```cisco
any
```

### Add a remark

```cisco
remark DESCRIPTION
```

### Remove sequence

```cisco
no <sequence-number>
```

### Apply to interface

```cisco
ip access-group NAME in
```

or:

```cisco
ip access-group NAME out
```

### Verify

```cisco
show access-lists
```

---

# 33. The Three-Question Mental Model

Whenever you see an Extended ACL, immediately identify:

```text
1. WHAT?
   ↓
Protocol

2. WHO?
   ↓
Source

3. WHERE?
   ↓
Destination
```

Then ask:

```text
4. WHICH SERVICE?
   ↓
Port / ICMP type
```

For example:

```text
deny tcp host 10.0.18.2 host 10.0.18.100 eq 443
```

Break it down:

```text
WHAT?     TCP
WHO?      10.0.18.2
WHERE?    10.0.18.100
SERVICE?  HTTPS / TCP 443
ACTION?   DENY
```

---

# 34. Critical Takeaways

### Remember these:

1. **Extended ACLs provide granular traffic control.**
2. They can match **protocol, source, and destination**.
3. They can additionally match **TCP/UDP ports**.
4. They can match specific **ICMP traffic/types**.
5. `ip` is a broad match for IP traffic.
6. **First match wins.**
7. The ACL still has an **implicit deny** at the end.
8. Put **specific rules before broad rules**.
9. For client-to-server application traffic, the service port is normally the **destination port**.
10. Extended ACLs are generally placed **as close to the source as possible**.
11. Named ACLs and sequence numbers make ACLs easier to maintain.
12. Always read an ACL entry as a sentence: **action + protocol + source + destination + optional service**.

---

## ⭐ One-Line Memory

> **Extended ACL = WHAT protocol + WHO source + WHERE destination + WHICH port/type; first match wins, implicit deny remains, and place it close to the source.**
