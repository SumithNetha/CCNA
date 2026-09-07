# Week 15 — August 12

# Skill 22 — Lesson 04: Configuring DHCP Services on Cisco Devices

## 1. What Is DHCP?

**DHCP — Dynamic Host Configuration Protocol** automatically provides network configuration to clients.

Instead of manually configuring every device with:

```text
IP address
Subnet mask
Default gateway
DNS server
```

DHCP provides these automatically.

### Basic idea

```text
Client
   │
   │ "I need network configuration"
   ▼
DHCP Server
   │
   │ IP + network information
   ▼
Client
```

DHCP is particularly important because many devices depend on it just to become functional on the network.

Examples from the lesson:

* Barista tablets
* Office PCs
* Switch management interfaces

---

# 2. Cisco Router as a DHCP Server

A Cisco router can operate as a **DHCP server**.

This is particularly useful in **small-to-midsize networks**, where the router already:

* Routes between networks
* Knows the relevant subnets
* Sits at the network boundary

The basic configuration concept is:

```text
DHCP Server
     │
     ├── DHCP Pool
     │
     ├── Network
     │
     ├── Default Gateway
     │
     ├── DNS Server
     │
     └── Other DHCP options
```

The DHCP pool defines the address space from which the router can allocate addresses.

---

# 3. DHCP Pool

A **DHCP pool** represents the collection of addresses and configuration parameters available to DHCP clients for a particular network.

The lesson says the pool needs information such as:

* Network
* Default gateway
* DNS server
* Domain name
* Other DHCP options

Conceptually:

```text
              DHCP Pool
                  │
       ┌──────────┼───────────┐
       │          │           │
    Network    Gateway       DNS
       │          │           │
       └──────────┼───────────┘
                  │
                  ▼
               Clients
```

---

# 4. DHCP Excluded Addresses

This is one of the most important configuration concepts in the lesson.

Before creating the DHCP pool, configure **excluded addresses**.

An excluded address is an IP address that DHCP should **not dynamically assign**.

Why?

Because some addresses may already be reserved for devices using static addressing.

Examples:

```text
Router
Server
Printer
Network infrastructure
Other statically configured devices
```

### Example concept

Suppose a subnet contains:

```text
192.168.10.1 ─ Router
192.168.10.2 ─ Server
192.168.10.3 ─ Printer
192.168.10.4 ─ Network device
192.168.10.5+ ─ DHCP clients
```

You don't want the DHCP server accidentally allocating `.2` to a laptop when `.2` belongs permanently to the server.

So those addresses are excluded.

### Real-world practice

Document your excluded ranges.

Don't simply exclude addresses and forget why.

```text
Excluded:
192.168.10.1 → Gateway
192.168.10.2 → Server
192.168.10.3 → Printer
```

That makes future troubleshooting much easier.

---

# 5. DORA — The Heart of DHCP

If you understand **DORA**, you understand the fundamental DHCP process.

**DORA =**

> **D**iscover → **O**ffer → **R**equest → **A**cknowledge

---

## Step 1 — Discover

The client doesn't have an IP address yet.

It essentially asks:

> "Is there a DHCP server that can give me an address?"

The DHCP Discover is sent as a **broadcast**.

```text
Client
  │
  │ DHCP Discover
  │ Broadcast
  ▼
Local Network
  │
  ├── DHCP Server
  ├── Other devices
  └── Other devices
```

The client uses broadcast because it doesn't yet know where the DHCP server is.

---

# 6. Offer

The DHCP server responds with an **Offer**.

Conceptually:

> "I can give you this IP address."

```text
Client
   │
   │ Discover
   ▼
DHCP Server
   │
   │ Offer
   ▼
Client
```

If multiple DHCP servers respond, the client may receive multiple offers.

The lesson describes the client generally selecting the first offer that arrives.

---

# 7. Request

The client chooses an offer and sends a **Request**.

Conceptually:

> "Yes, I want that address."

```text
Client
   │
   │ Request
   ▼
DHCP Server
```

---

# 8. Acknowledge

The server sends an **Acknowledge**, commonly called an **ACK**.

This finalizes the DHCP process.

The client receives not only its IP address but potentially additional network information such as:

* IP address
* Default gateway
* DNS server
* Lease time
* Other DHCP options

So DHCP isn't simply:

> "Here's an IP."

It is more like:

> **"Here's your network identity and information about how to communicate."**

---

# 9. Complete DORA Flow

```text
       DHCP CLIENT                         DHCP SERVER
            │                                   │
            │────── DHCP DISCOVER ─────────────>│
            │                                   │
            │<─────── DHCP OFFER ──────────────│
            │                                   │
            │─────── DHCP REQUEST ─────────────>│
            │                                   │
            │<──── DHCP ACKNOWLEDGE ────────────│
            │                                   │
            ▼
       Network configuration
       is now available
```

### Memorize:

```text
D = Discover
O = Offer
R = Request
A = Acknowledge
```

---

# 10. DHCP Gives More Than an IP Address

This is an important conceptual point.

A DHCP server can provide the client with information such as:

```text
IP Address
Subnet information
Default Gateway
DNS Server
Lease Time
Other DHCP Options
```

Therefore:

```text
DHCP
 │
 ├── "Who am I?"       → IP configuration
 ├── "Where am I?"     → Network/subnet
 ├── "Where's DNS?"    → DNS server
 └── "Where's outside?"→ Default gateway
```

This is why DHCP is fundamental to normal network operation.

---

# 11. Network Devices Can Also Be DHCP Clients

DHCP isn't limited to PCs and laptops.

Cisco network devices can also function as **DHCP clients**.

For example, a switch can obtain an IP address dynamically for a management interface.

The lesson describes using a **Switched Virtual Interface (SVI)** for this purpose.

Conceptually:

```text
DHCP Server
     │
     │
     ▼
Switch SVI
     │
     ▼
Management IP
```

This can be useful for:

* Labs
* Staging environments
* Quick deployments

---

# 12. DHCP for Infrastructure — Production Consideration

Just because a network device **can** use DHCP doesn't mean it should always do so.

For production infrastructure, static management addresses are often preferable because they make:

* Documentation easier
* Monitoring more predictable
* Remote access more predictable
* Troubleshooting easier

So remember the distinction:

```text
DHCP for infrastructure
        │
        ├── Possible
        │
        └── Not necessarily best production design
```

---

# 13. The Big Problem: DHCP Server on Another Subnet

This is one of the most important concepts in the lesson.

Suppose:

```text
VLAN 10
   │
   │ DHCP Discover
   ▼
Router
   │
   X
   │
VLAN 50
   │
DHCP Server
```

Why doesn't it work automatically?

Because the DHCP Discover is a **broadcast**.

And:

> **Routers do not forward broadcasts by default.**

Therefore, a DHCP server located on another subnet won't automatically receive the client's DHCP Discover.

---

# 14. DHCP Relay Agent

The solution is a **DHCP relay agent**.

On Cisco devices, this is commonly configured using:

```cisco
ip helper-address <DHCP-server-IP>
```

The important command from this lesson is:

```text
ip helper-address
```

Its purpose is to tell the router:

> "When DHCP requests arrive on this interface, forward them to the DHCP server over there."

---

# 15. DHCP Relay Example

Imagine:

```text
             VLAN 10
          192.168.10.0/24
                │
              Client
                │
                ▼
         ┌─────────────┐
         │   Router    │
         │             │
         │ ip helper   │
         └──────┬──────┘
                │
                │ Routed traffic
                ▼
         ┌─────────────┐
         │ DHCP Server │
         │ 10.50.0.10  │
         └─────────────┘
```

The client broadcasts locally.

The router receives the DHCP request and relays it toward the centralized DHCP server.

---

# 16. Why the Relay Needs Interface Information

The router doesn't just blindly forward the request.

The relay provides information associated with the interface where the DHCP request was received.

That allows the centralized DHCP server to determine:

> "This request came from the VLAN 10 subnet."

The DHCP server can then select the appropriate address pool/scope.

Conceptually:

```text
Client in VLAN 10
       │
       │ DHCP Discover
       ▼
DHCP Relay
       │
       │ Identifies originating network
       ▼
Central DHCP Server
       │
       │ Select VLAN 10 address pool
       ▼
Offer appropriate address
```

This is what makes **centralized DHCP** practical in larger networks.

---

# 17. Centralized DHCP

In an enterprise network, you don't necessarily want every router acting as its own DHCP server.

Instead:

```text
                    Central DHCP
                         │
          ┌──────────────┼──────────────┐
          │              │              │
       VLAN 10         VLAN 20        VLAN 30
          │              │              │
       Relay           Relay          Relay
          │              │              │
       Clients         Clients        Clients
```

A centralized DHCP server can potentially serve many different VLANs.

The DHCP server could be:

* Windows Server
* Linux server
* Dedicated appliance

while routers/L3 switches act as DHCP relay agents.

---

# 18. Three DHCP Roles on Cisco Devices

This is probably the **single most important takeaway** from the lesson.

Cisco devices can participate in DHCP in **three roles**:

| Role                 | Function                                           |
| -------------------- | -------------------------------------------------- |
| **DHCP Server**      | Assigns IP configuration to clients                |
| **DHCP Client**      | Obtains IP configuration from a DHCP server        |
| **DHCP Relay Agent** | Forwards DHCP requests toward a remote DHCP server |

### Visualize it

```text
                 Cisco DHCP Roles
                        │
        ┌───────────────┼────────────────┐
        │               │                │
      SERVER          CLIENT           RELAY
        │               │                │
        ▼               ▼                ▼
    Gives IPs       Gets IPs       Forwards DHCP
```

---

# 19. DHCP Troubleshooting Mindset

When DHCP isn't working, don't randomly change configurations.

Ask questions in sequence.

### Question 1

**Is the client sending DHCP Discover?**

If not, investigate the client/local connection.

### Question 2

**Is the DHCP server responding?**

If there's a Discover but no Offer, investigate the server path/configuration.

### Question 3

**Is the DHCP pool correct?**

Check whether the correct network and available addresses are configured.

### Question 4

**Are excluded addresses correct?**

You may have accidentally excluded too much of the address space.

### Question 5

**Is a DHCP relay/helper address required?**

If the DHCP server is on another subnet, check whether the relay configuration exists.

---

# 20. Troubleshooting Flow

```text
Client cannot obtain IP
          │
          ▼
Is DHCP Discover being sent?
          │
     ┌────┴────┐
    NO        YES
    │           │
    ▼           ▼
Investigate   Does server
client/local   respond?
network          │
             ┌───┴───┐
            NO      YES
             │        │
             ▼        ▼
       Check DHCP   Check pool,
       server/path  exclusions,
                    relay, etc.
```

---

# 21. DHCP in Castle Rysen Coffee

DHCP is explicitly part of the Castle Rysen network requirements.

The RFP requires IP services including:

* NAT
* DHCP
* DNS
* NTP
* SSH

for the district shops. 

This means DHCP isn't an isolated CCNA topic. It's part of the actual network design scenario you're building throughout the course.

For example:

```text
             District Shop
                   │
          ┌────────┴────────┐
          │                 │
     Admin Devices      Patron Devices
          │                 │
          └────────┬────────┘
                   │
                DHCP
                   │
          Automatic addressing
```

As Castle Rysen expands from individual shops to Fallout Shelters and Central Offices, centralized DHCP + relay becomes increasingly relevant.

---

# 22. Cisco DHCP Design: Small vs Large Network

### Small network

A router can provide DHCP directly:

```text
             Router
          DHCP Server
               │
       ┌───────┼───────┐
       │       │       │
      PC      POS    Tablet
```

Simple and practical.

### Larger network

Use centralized DHCP:

```text
             DHCP Server
                  │
       ┌──────────┼──────────┐
       │          │          │
     Relay      Relay      Relay
       │          │          │
     VLAN 10    VLAN 20    VLAN 30
```

This separates the DHCP service from the network devices doing the routing.

---

# 🧠 CCNA Exam Takeaways

Memorize these extremely well:

### DHCP

> **Dynamic Host Configuration Protocol automatically provides network configuration to clients.**

### DORA

```text
D → Discover
O → Offer
R → Request
A → Acknowledge
```

### Broadcast

The initial DHCP Discover is a **broadcast**.

### Router behavior

> **Routers do not forward DHCP broadcasts by default.**

### DHCP Relay

Use:

```cisco
ip helper-address <DHCP-server-IP>
```

when the DHCP server is on another subnet.

### Three roles

```text
Server → gives configuration
Client → receives configuration
Relay  → forwards requests
```

### DHCP Pool

Defines the address space and configuration parameters clients can receive.

### Exclusions

Reserve addresses that should **not** be dynamically allocated.

### Production infrastructure

Network devices *can* use DHCP, but static management addressing may be preferable for predictable management and documentation.

---

# 🔥 The Most Important Mental Model

Think of DHCP as two different architectures.

### Architecture 1 — Local DHCP

```text
Client
  │
  │ Broadcast
  ▼
Router
(DHCP Server)
  │
  ▼
IP configuration
```

### Architecture 2 — Centralized DHCP

```text
Client
  │
  │ Broadcast
  ▼
L3 Router/Switch
(DHCP Relay)
  │
  │ ip helper-address
  ▼
Central DHCP Server
  │
  ▼
Correct subnet/scope
  │
  ▼
Client gets configuration
```

If you understand **why Architecture 2 needs a relay**, you understand one of the most important concepts in this lesson.

---

## ⭐ One-Sentence Summary

> **DHCP automatically provides network configuration through the DORA process; a Cisco device can act as a DHCP server, client, or relay agent, with `ip helper-address` allowing DHCP broadcasts from one subnet to reach a centralized DHCP server on another subnet.**

This completes the **DHCP theory for August 12**. Your study plan has the **S22-L04 DHCP lab** as the hands-on task for today; tomorrow starts with the DHCP quiz and then moves into **SSH configuration**. 


# Example
Absolutely. Let's take **one realistic Castle Rysen Coffee example** and walk through DHCP from the moment a laptop connects until it gets a usable IP address. This will make **DHCP server, DORA, DHCP pool, excluded addresses, default gateway, DNS, and DHCP relay** fit together.

The Castle Rysen RFP requires DHCP and DNS as core IP services for the district shops. 

# Example: Castle Rysen Coffee District Shop

Imagine we have one coffee shop.

The shop has:

* 15 humans
* Employee/admin devices
* Customer devices
* A router
* A switch
* Internet connectivity
* A DNS server
* DHCP running on the router

For simplicity, we'll start with **one subnet**.

```text
                    INTERNET
                       │
                       │
                ┌──────┴──────┐
                │    Router   │
                │             │
                │ G0/0        │
                │ 192.168.10.1│
                └──────┬──────┘
                       │
                       │
                    Switch
                 ┌────┼────┐
                 │    │    │
                PC1  PC2  Laptop
```

Our network is:

```text
Network:        192.168.10.0/24
Router:         192.168.10.1
DNS Server:     192.168.10.10
DHCP Server:    Router
```

---

# 1. What happens when a new laptop joins?

Imagine a customer's laptop connects to Wi-Fi.

At this moment, the laptop doesn't know:

```text
IP address       ?
Subnet mask      ?
Default gateway  ?
DNS server       ?
```

It needs DHCP.

So the laptop essentially says:

> "I need network configuration."

This starts **DORA**.

```text
D = Discover
O = Offer
R = Request
A = Acknowledge
```

---

# 2. Step 1 — DHCP Discover

The laptop doesn't know the DHCP server's IP address.

It can't say:

```text
"Hey 192.168.10.1, give me an IP."
```

because it doesn't have an IP configuration yet.

So it sends a **DHCP Discover broadcast**.

Conceptually:

```text
Laptop
   │
   │ DHCP Discover
   │ Broadcast
   ▼
Switch
   │
   ├───────────────┐
   │               │
   ▼               ▼
Router          Other devices
```

The important point:

> The client uses a broadcast because it doesn't yet know where the DHCP server is.

---

# 3. Router receives the Discover

Our router is configured as the DHCP server.

It has a DHCP pool:

```text
192.168.10.0/24
```

The router looks at the available addresses.

Suppose we reserve:

```text
192.168.10.1  → Router
192.168.10.10 → DNS server
192.168.10.20 → Printer
```

These should not be dynamically assigned.

So we exclude them.

Conceptually:

```text
192.168.10.1     Router       EXCLUDED
192.168.10.2     Available
192.168.10.3     Available
...
192.168.10.10    DNS Server   EXCLUDED
...
192.168.10.20    Printer      EXCLUDED
...
192.168.10.21    Available
```

The DHCP server might therefore offer:

```text
192.168.10.2
```

to the laptop.

---

# 4. Step 2 — DHCP Offer

The router responds:

> "I can give you 192.168.10.2."

But it doesn't just offer an IP.

It can also provide information such as:

```text
IP address:       192.168.10.2
Subnet mask:      255.255.255.0
Default gateway:  192.168.10.1
DNS server:       192.168.10.10
Lease time:       configured duration
```

So:

```text
                 DHCP SERVER
                     │
                     │ OFFER
                     │
                     ▼
Laptop ◄──── 192.168.10.2
             255.255.255.0
             Gateway 192.168.10.1
             DNS 192.168.10.10
```

The laptop now knows what address it could use.

But the process isn't finished.

---

# 5. Step 3 — DHCP Request

The laptop decides:

> "Yes, I want 192.168.10.2."

It sends a DHCP Request.

```text
Laptop
   │
   │ DHCP Request
   │
   ▼
Router
```

This is essentially the client saying:

> "I accept that offer."

---

# 6. Step 4 — DHCP Acknowledge

The router sends a DHCP ACK.

```text
Router
   │
   │ DHCP ACK
   ▼
Laptop
```

The laptop can now configure itself.

```text
IP address       = 192.168.10.2
Subnet mask      = 255.255.255.0
Default gateway  = 192.168.10.1
DNS server       = 192.168.10.10
```

DORA is complete.

---

# 7. Now the Laptop Actually Has an Identity

Before DHCP:

```text
Laptop
IP = ???
Gateway = ???
DNS = ???
```

After DHCP:

```text
Laptop
   │
   ├── IP:       192.168.10.2
   ├── Mask:     255.255.255.0
   ├── Gateway:  192.168.10.1
   └── DNS:      192.168.10.10
```

This is what the lesson means when it says DHCP is more than simply handing out an IP.

It gives the client its **network identity and direction**.

---

# 8. What Does the Default Gateway Do?

Now the laptop wants to access something outside its local subnet.

For example:

```text
google.com
```

First, it needs DNS.

The laptop asks:

```text
"What IP address belongs to google.com?"
```

It sends the DNS query to:

```text
192.168.10.10
```

Once it gets an IP address, the laptop determines whether that destination is local or remote.

If it's remote, it sends the traffic to:

```text
192.168.10.1
```

That's the default gateway.

So the DHCP configuration has effectively told the laptop:

```text
Your address:
192.168.10.2

Your local network:
192.168.10.0/24

Your way out:
192.168.10.1

Your DNS server:
192.168.10.10
```

---

# 9. Now Let's Configure the Cisco Router

Suppose we want:

```text
Network:       192.168.10.0/24
Gateway:       192.168.10.1
DNS:           192.168.10.10
DHCP clients:  192.168.10.2 - 192.168.10.254
```

First, exclude addresses.

```cisco
Router(config)# ip dhcp excluded-address 192.168.10.1
Router(config)# ip dhcp excluded-address 192.168.10.10
Router(config)# ip dhcp excluded-address 192.168.10.20
```

Now create the DHCP pool:

```cisco
Router(config)# ip dhcp pool CAFE_CLIENTS
```

We are now inside the DHCP pool configuration.

Tell it which network it serves:

```cisco
Router(dhcp-config)# network 192.168.10.0 255.255.255.0
```

Tell clients their default gateway:

```cisco
Router(dhcp-config)# default-router 192.168.10.1
```

Tell clients their DNS server:

```cisco
Router(dhcp-config)# dns-server 192.168.10.10
```

So the configuration is:

```cisco
ip dhcp excluded-address 192.168.10.1
ip dhcp excluded-address 192.168.10.10
ip dhcp excluded-address 192.168.10.20

ip dhcp pool CAFE_CLIENTS
 network 192.168.10.0 255.255.255.0
 default-router 192.168.10.1
 dns-server 192.168.10.10
```

---

# 10. What Does Each Command Actually Do?

This is important. Don't memorize the commands without understanding them.

### `ip dhcp excluded-address`

```cisco
ip dhcp excluded-address 192.168.10.10
```

Means:

> DHCP should not dynamically allocate this address.

---

### `ip dhcp pool`

```cisco
ip dhcp pool CAFE_CLIENTS
```

Creates/selects the DHCP pool.

Think:

> "I'm defining the DHCP service for this network."

---

### `network`

```cisco
network 192.168.10.0 255.255.255.0
```

Defines the subnet from which the pool operates.

---

### `default-router`

```cisco
default-router 192.168.10.1
```

Tells clients:

> "Use 192.168.10.1 as your default gateway."

---

### `dns-server`

```cisco
dns-server 192.168.10.10
```

Tells clients:

> "Use 192.168.10.10 for DNS."

---

# 11. What If We Have Multiple VLANs?

Now let's make this more realistic.

The Castle Rysen RFP requires district shops to separate **patron devices from administrative devices**. 

So suppose we have:

```text
VLAN 10 = ADMIN
VLAN 20 = PATRONS
```

And:

```text
VLAN 10
192.168.10.0/24

VLAN 20
192.168.20.0/24
```

Our router/L3 device has:

```text
192.168.10.1 → VLAN 10 gateway
192.168.20.1 → VLAN 20 gateway
```

We could have:

```text
             Router
          ┌─────┴─────┐
          │           │
      VLAN 10       VLAN 20
      ADMIN         PATRONS
         │             │
       PCs          Customers
```

Now imagine the DHCP server is the router itself.

We can create **two DHCP pools**:

```cisco
ip dhcp pool ADMIN
 network 192.168.10.0 255.255.255.0
 default-router 192.168.10.1
 dns-server 192.168.10.10

ip dhcp pool PATRONS
 network 192.168.20.0 255.255.255.0
 default-router 192.168.20.1
 dns-server 192.168.10.10
```

Now:

```text
ADMIN client
     │
     ▼
192.168.10.0/24 pool
     │
     ▼
192.168.10.x


PATRON client
     │
     ▼
192.168.20.0/24 pool
     │
     ▼
192.168.20.x
```

The router determines which pool applies based on the client's network/interface.

---

# 12. Now Change One Thing: Centralized DHCP

Imagine Castle Rysen grows.

Instead of every router being a DHCP server, the company creates a centralized DHCP server:

```text
                 Central DHCP
                 10.100.0.10
                      │
                      │
                 Core Network
                      │
          ┌───────────┴───────────┐
          │                       │
      District A              District B
       Router                   Router
          │                       │
       Clients                 Clients
```

This is where **DHCP relay** becomes important.

---

# 13. Why Does DHCP Relay Exist?

Suppose a client in:

```text
192.168.20.0/24
```

needs DHCP.

It sends:

```text
DHCP Discover
```

as a broadcast.

But the DHCP server lives at:

```text
10.100.0.10
```

Different subnet.

The router will **not forward the broadcast by default**.

So:

```text
Client
  │
  │ Broadcast
  ▼
Router
  │
  X
  │
  X ── DHCP server never receives it
```

That's the problem.

---

# 14. `ip helper-address` Solves It

On the router interface for the client network, configure:

```cisco
ip helper-address 10.100.0.10
```

Now:

```text
Client
  │
  │ DHCP Discover
  │ Broadcast
  ▼
Router
  │
  │ DHCP Relay
  │
  │ ip helper-address 10.100.0.10
  ▼
Central DHCP Server
```

The router receives the broadcast and relays the DHCP request toward the centralized server.

---

# 15. Why Does the Server Know Which Pool to Use?

Imagine the central DHCP server has:

```text
Pool A:
192.168.10.0/24

Pool B:
192.168.20.0/24

Pool C:
192.168.30.0/24
```

A client from VLAN 20 sends a DHCP request.

The relay provides information identifying the network where the request originated.

The DHCP server can therefore determine:

> "This request came from 192.168.20.0/24."

So it chooses:

```text
Pool B
```

and offers an address such as:

```text
192.168.20.50
```

---

# 16. Complete Enterprise Example

Now put everything together:

```text
                         INTERNET
                            │
                            │
                       ┌────┴────┐
                       │  Router │
                       │ / L3 SW │
                       └────┬────┘
                            │
                       DHCP Relay
                            │
                     ip helper-address
                            │
                            ▼
                  ┌─────────────────┐
                  │ Central DHCP    │
                  │ 10.100.0.10     │
                  └─────────────────┘


          District Shop
                │
        ┌───────┴────────┐
        │                │
     VLAN 10          VLAN 20
      ADMIN            PATRON
        │                │
      PC                  Laptop
        │                │
        └───────┬────────┘
                │
             Switch
                │
             Router
```

A customer laptop joins VLAN 20.

### DORA:

```text
1. DISCOVER
Laptop → broadcast

2. OFFER
Relay → DHCP Server
DHCP Server → offer 192.168.20.50

3. REQUEST
Laptop → requests 192.168.20.50

4. ACK
DHCP Server → confirms
```

Laptop ends up with:

```text
IP:       192.168.20.50
Mask:     255.255.255.0
Gateway:  192.168.20.1
DNS:      configured DNS server
```

That's the complete story.

---

# 17. How to Troubleshoot This Example

Suppose the customer's laptop says:

> **"No Internet — couldn't get an IP address."**

Don't immediately blame the ISP.

Walk through DHCP.

### Step 1

Is the laptop sending Discover?

```text
Laptop → DHCP Discover
```

If not:

```text
Check client/Wi-Fi/VLAN/local connectivity
```

---

### Step 2

Does the router receive it?

If not:

```text
Check VLAN
Check trunk
Check interface
Check local connectivity
```

---

### Step 3

Is the DHCP server reachable?

If centralized:

```text
Check ip helper-address
Check routing
Check connectivity to DHCP server
```

---

### Step 4

Does the DHCP server have an appropriate pool?

For a VLAN 20 client:

```text
192.168.20.0/24
```

The server needs the appropriate scope/pool.

---

### Step 5

Are too many addresses excluded?

Suppose you accidentally excluded:

```text
192.168.20.1 - 192.168.20.253
```

Only `.254` remains.

Your DHCP service will quickly run out of usable addresses.

---

# 18. The Three Cisco DHCP Roles — One Example

Using our Castle Rysen network:

### Server

Router directly assigns addresses:

```text
Router
   │
   │ DHCP
   ▼
Client
```

**Router = DHCP Server**

---

### Client

A Cisco switch obtains its management address dynamically:

```text
DHCP Server
     │
     ▼
Switch SVI
     │
     ▼
Gets management IP
```

**Switch = DHCP Client**

---

### Relay

The DHCP server lives elsewhere:

```text
Client
  │
  ▼
Router
  │
  │ ip helper-address
  ▼
Central DHCP Server
```

**Router = DHCP Relay Agent**

---

# 🔥 The Whole Lesson in One Scenario

Remember this story:

> A new customer walks into Castle Rysen Coffee and connects their laptop to the Wi-Fi.

The laptop has no IP configuration.

### ① Discover

```text
"I need an IP!"
```

Broadcast.

### ② Offer

```text
DHCP:
"I can give you 192.168.20.50."
```

### ③ Request

```text
Laptop:
"Yes, I'll take 192.168.20.50."
```

### ④ Acknowledge

```text
DHCP:
"Confirmed. Here's your configuration."
```

The laptop receives:

```text
IP       → 192.168.20.50
Mask     → 255.255.255.0
Gateway  → 192.168.20.1
DNS      → DNS server address
```

If the DHCP server is on another network:

```text
Client
   ↓
Broadcast
   ↓
Router
   ↓
ip helper-address
   ↓
Central DHCP Server
```

And **that is DHCP**.

---

## 🧠 What I want you to be able to explain without notes

If I ask you:

> **"A laptop in VLAN 20 isn't getting an IP. Explain what happens and troubleshoot it."**

You should be able to say:

> "The client begins with a DHCP Discover broadcast. The DHCP server should respond with an Offer, the client sends a Request, and the server sends an Acknowledge. If the DHCP server is on another subnet, the broadcast won't cross the router, so I need a DHCP relay configured with `ip helper-address` on the client-facing Layer 3 interface. Then I'd verify the DHCP pool, excluded addresses, relay configuration, routing, and whether the client is actually generating the Discover."

**If you can explain that from memory, you've understood this lesson rather than just memorized DORA.**
