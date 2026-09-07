# Week 15 — August 12

# Skill 22 — Lesson 03: Using DNS on Cisco Devices

## 1. What is DNS?

**DNS (Domain Name System)** translates human-readable names into IP addresses.

```text
amazon.com  ──DNS query──>  DNS Server
                              │
                              ▼
                         IP Address
```

Humans prefer names such as:

```text
amazon.com
server.cafe.local
mail.example.com
```

Networks ultimately communicate using IP addresses.

### Core idea

> **DNS = Name → IP address resolution**

If DNS fails, the network itself may still be functioning, but applications that depend on names can appear to be broken.

---

# 2. DNS Ports

DNS commonly uses:

| Protocol |   Port | Typical use                                                                    |
| -------- | -----: | ------------------------------------------------------------------------------ |
| UDP      | **53** | Normal DNS queries                                                             |
| TCP      | **53** | Larger responses and DNS-related operations such as replication/zone transfers |

The lesson emphasizes that most client DNS requests use **UDP/53**.

### Why UDP?

DNS uses a request/response model. If a response doesn't arrive, the client can retry the request.

---

# 3. Important DNS Record Types

You should know these for CCNA:

| Record    | Purpose                                                                    |
| --------- | -------------------------------------------------------------------------- |
| **A**     | Maps hostname → IPv4 address                                               |
| **CNAME** | Alias pointing to another hostname                                         |
| **MX**    | Specifies mail servers for a domain                                        |
| **NS**    | Identifies authoritative name servers                                      |
| **TXT**   | Text information used for verification, email security, and other purposes |

### Example

```text
www.example.com
      │
      ▼
A record
      │
      ▼
192.0.2.10
```

### CNAME example

```text
shop.example.com
       │
     CNAME
       │
       ▼
www.example.com
```

The CNAME doesn't directly provide the IP address; it points to another DNS name.

---

# 4. Why DNS Is So Important

DNS is often treated as a background service, but many applications depend on it.

For example:

```text
User
 │
 ├── Web browsing
 ├── Email
 ├── Cloud applications
 ├── APIs
 └── Collaboration applications
          │
          ▼
        DNS
          │
          ▼
      IP address
```

If DNS becomes unavailable or slow, users may report:

> "The internet is slow."

Even when:

* bandwidth is fine
* interfaces are healthy
* cables are fine
* routing is working
* Wi-Fi is working

### Important troubleshooting lesson

**DNS problems can look like network performance problems.**

The lesson specifically recommends testing DNS early when users report that the Internet is slow.

Useful tool:

```text
nslookup
```

You can compare DNS response behavior against another DNS server.

---

# 5. DNS Caching

DNS servers **cache DNS responses** for a period of time.

### Without caching

Every request might require another DNS lookup.

```text
Client
  │
  ▼
DNS Server
  │
  ▼
Internet DNS hierarchy
```

### With caching

```text
Client
  │
  ▼
DNS Server
  │
  ├── Cached answer → return immediately
  │
  └── No cached answer → perform lookup
```

### Advantage

Caching improves:

* response time
* efficiency
* DNS server workload

### Disadvantage

DNS changes may not appear immediately because old information can remain in caches.

**Key concept:**

> DNS caching improves performance but can delay visibility of DNS changes.

---

# 6. Running DNS Internally

Organizations can operate their own internal DNS infrastructure.

The lesson gives examples such as:

* Windows Server
* Linux
* NAS appliances

An internal DNS server can:

1. Answer internal DNS queries directly.
2. Forward unknown queries to upstream DNS servers.
3. Recursively find answers through the DNS hierarchy.

Example:

```text
Client
   │
   ▼
Internal DNS
   │
   ├── Knows internal hostname?
   │       │
   │       └── Yes → answer
   │
   └── Doesn't know?
           │
           ▼
      Upstream DNS
```

---

# 7. Cisco Devices and DNS

There are **two different concepts** you need to distinguish.

### Cisco device as a DNS client

The Cisco router/switch uses an external DNS server to resolve names.

### Cisco device as a DNS server

The Cisco device can provide DNS service and maintain local hostname-to-IP mappings.

These are **not the same thing**.

---

# 8. Cisco as a DNS Client

Use:

```cisco
ip name-server <DNS-IP>
```

Example:

```cisco
Router(config)# ip name-server 8.8.8.8
```

This tells the Cisco device which DNS server to use for hostname resolution.

Then you can use names instead of manually entering IP addresses in commands where hostname resolution is supported.

Conceptually:

```text
Cisco Router
     │
     │ DNS query
     ▼
DNS Server
     │
     │ IP address
     ▼
Cisco Router
```

---

# 9. Cisco as a Lightweight DNS Server

A Cisco device can also maintain local hostname mappings using:

```cisco
ip host <hostname> <IP-address>
```

Example:

```cisco
Router(config)# ip host WEB-SERVER 192.168.10.10
```

Now the router has a local mapping:

```text
WEB-SERVER → 192.168.10.10
```

This is useful for small environments where deploying a dedicated DNS server would introduce unnecessary complexity.

---

# 10. DNS Service on Cisco

The lesson distinguishes between configuring the Cisco device to **use DNS** and configuring it to **provide DNS service**.

Conceptually:

```text
             Cisco Router
             /          \
            /            \
       DNS client       DNS service
           │                 │
           ▼                 ▼
   External DNS        Local mappings
   ip name-server       ip host
```

For a small network, local mappings can be practical.

For a large enterprise, relying on a single router for DNS would create an unnecessary dependency and larger failure domain.

---

# 11. DHCP + DNS

DNS and DHCP often work together.

The lesson's workflow is:

```text
1. Configure Cisco device to use DNS
             ↓
2. Optionally enable DNS service
             ↓
3. Create local mappings with ip host
             ↓
4. Give clients the DNS server address through DHCP
```

This is an important connection between today's lesson and the **next DHCP lesson**.

For example:

```text
                    Router
                      │
              ┌───────┴────────┐
              │                │
             DHCP             DNS
              │                │
              ▼                ▼
        Client receives   Client resolves
        DNS server IP     hostnames
```

---

# 12. Castle Rysen / NetworkChuck Coffee Application

For a small coffee shop network, the lesson's argument is:

> **Less complexity can be a feature.**

A Cisco router can potentially provide simple local DNS functionality instead of deploying another dedicated server.

Example:

```text
                    Cafe Router
                  /      |       \
                 /       |        \
              POS     Plex       Admin PC
               │        │          │
               └────────┴──────────┘
                        │
                   Local DNS
                        │
               name → IP mappings
```

This can reduce:

* infrastructure complexity
* number of servers
* maintenance requirements
* dependencies

However, as the organization grows, the lesson recommends moving toward **redundant, highly available DNS servers**.

That aligns with the Castle Rysen RFP, which describes an organization that must scale from individual district shops to Fallout Shelters and Central Offices. The RFP specifically includes **DNS, DHCP, and NTP** among the required network services. 

---

# 13. Real-World Troubleshooting

When someone says:

> **"The Internet is slow."**

Don't immediately assume:

```text
Bad Wi-Fi
Bad cable
Bad ISP
Low bandwidth
Duplex problem
```

Also test DNS.

### Basic approach

```text
User reports slow Internet
          │
          ▼
       Test DNS
          │
          ├── Slow/failing DNS
          │        ↓
          │   Investigate resolver
          │
          └── DNS OK
                   ↓
            Continue network
            troubleshooting
```

The lesson specifically calls out:

```text
nslookup
```

as a useful diagnostic tool.

---

# 14. Key Cisco Commands

### Configure a DNS server

```cisco
ip name-server <DNS-IP>
```

Example:

```cisco
ip name-server 192.168.1.53
```

### Create a local hostname mapping

```cisco
ip host <hostname> <IP-address>
```

Example:

```cisco
ip host FILE-SERVER 192.168.10.20
```

### Conceptual workflow

```text
Cisco device
     │
     ├── ip name-server
     │       ↓
     │   Uses external DNS
     │
     └── ip host
             ↓
       Local hostname mapping
```

---

# 🧠 CCNA Exam Takeaways

Memorize these:

1. **DNS translates names to IP addresses.**
2. Normal DNS queries commonly use **UDP/53**.
3. DNS also uses **TCP/53** for cases requiring TCP.
4. **A = IPv4 address**
5. **CNAME = alias**
6. **MX = mail**
7. **NS = name server**
8. **TXT = text/verification/security information**
9. `ip name-server` tells a Cisco device **which DNS server to use**.
10. `ip host` creates a **local hostname-to-IP mapping**.
11. DNS caching improves performance but can delay DNS changes.
12. DNS failures can make a healthy network **appear slow or broken**.
13. Use `nslookup` when troubleshooting DNS from a client.
14. Small networks can use lightweight local DNS functionality.
15. Larger networks should use **redundant/highly available DNS infrastructure**.

---

## 🔗 Today's Mental Model

```text
                 DNS
                  │
       ┌──────────┴──────────┐
       │                     │
    Client                  Server
       │                     │
       │ query               │
       └────────────────────►│
                             │
                         Lookup / Cache
                             │
       IP address             │
       ◄─────────────────────┘


Cisco IOS
   │
   ├── ip name-server
   │       ↓
   │   Use DNS server
   │
   └── ip host
           ↓
      Local name → IP
```

### ⭐ One sentence to remember

> **DNS is the naming system that lets humans and applications use names instead of IP addresses; on Cisco IOS, `ip name-server` points the device to DNS and `ip host` creates local name-to-IP mappings.**

This completes the **Lesson 03 theory** for August 12. The next step in today's plan is **S22-L03 — Using DNS on Cisco Devices lab**, followed by the 5-question quiz, then **Lesson 04 — Configuring DHCP Services**. 
