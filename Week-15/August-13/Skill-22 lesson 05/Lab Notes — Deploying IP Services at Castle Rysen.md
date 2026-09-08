# Lab Notes — Deploying IP Services at Castle Rysen

## 1. FO-RT01: DNS configuration

You configured:

```cisco
ip domain-name castlerysen.local
ip host cafe1.castlerysen.local 172.16.0.2
ip name-server 1.1.1.1
```

These three commands have different purposes.

### `ip domain-name`

```cisco
ip domain-name castlerysen.local
```

This establishes the router's DNS domain:

```text
FO-RT01.castlerysen.local
```

More importantly, this is also commonly required when generating RSA keys for SSH.

---

### `ip host`

```cisco
ip host cafe1.castlerysen.local 172.16.0.2
```

This creates a **static hostname-to-IP mapping** on the router.

Think:

```text
cafe1.castlerysen.local
        ↓
    172.16.0.2
```

So if FO-RT01 needs to communicate with:

```text
cafe1.castlerysen.local
```

it can resolve that name locally without querying an external DNS server.

This is essentially a tiny local DNS database on the router.

---

### `ip name-server`

```cisco
ip name-server 1.1.1.1
```

This tells the router:

> "If you don't know the hostname locally, ask 1.1.1.1."

So the conceptual resolution order is:

```text
                  DNS lookup
                      |
             +--------+--------+
             |                 |
       Local ip host       External DNS
             |                 |
      cafe1.castlerysen    1.1.1.1
```

This is why you can have both:

```cisco
ip host ...
ip name-server ...
```

They aren't redundant.

---

# 2. Why did you create Loopback1?

You configured:

```cisco
interface loopback 1
 ip address 1.1.1.1 255.255.255.255
```

A loopback interface is a **logical interface**, not a physical port.

Unlike:

```text
FastEthernet
GigabitEthernet
Serial
```

it doesn't depend on a physical cable.

Your physical topology might look like:

```text
             FO-RT01
          /     |      \
       VLANs   WAN    FO-RT02
```

But Loopback1 exists logically inside the router:

```text
FO-RT01
   |
   +--- Loopback1
          1.1.1.1/32
```

The `/32` is important.

```text
1.1.1.1 255.255.255.255
```

means:

```text
1.1.1.1/32
```

There is exactly **one host address** in that prefix.

---

# 3. Why use a Loopback for services?

This is one of the most important real-world networking concepts in this lesson.

Suppose you use the physical interface:

```text
FO-RT01 Serial0/0/0
172.16.0.1
```

as your NTP server address.

If that physical interface goes down:

```text
FO-RT01 ----X---- FO-RT02
```

clients lose access to that service address.

A loopback is much more stable.

For example:

```text
FO-RT01 Loopback1
1.1.1.1/32
```

If OSPF advertises that address, another router can potentially reach:

```text
1.1.1.1
```

through whatever available path exists.

That's the reason loopbacks are heavily used for:

* Router IDs
* Management
* NTP
* DNS
* SNMP
* Routing protocol endpoints
* Service addresses

---

# 4. Your OSPF Router ID changed to 1.1.1.1

This output is particularly important:

```text
Routing Protocol is "ospf 1"
Router ID 1.1.1.1
```

Before creating Loopback1, the router apparently did not have a manually configured router ID.

After you created:

```cisco
interface loopback 1
 ip address 1.1.1.1 255.255.255.255
```

OSPF selected:

```text
1.1.1.1
```

as the router ID.

Then you explicitly advertised it:

```cisco
router ospf 1
 network 1.1.1.1 0.0.0.0 area 0
```

So now:

```text
Loopback1
1.1.1.1/32
     |
     +---- OSPF Router ID
     |
     +---- OSPF advertised network
```

That's a very useful relationship to understand.

### Important distinction

The command:

```cisco
network 1.1.1.1 0.0.0.0 area 0
```

does **not create** the IP address.

The address already exists because of:

```cisco
interface loopback 1
ip address 1.1.1.1 255.255.255.255
```

The OSPF command tells OSPF:

> "Activate OSPF on the interface matching this address and advertise this connected network."

---

# 5. Your routing table confirms the Loopback

You got:

```text
1.0.0.0/32 is subnetted, 1 subnets
C       1.1.1.1/32 is directly connected, Loopback1
```

`C` means:

**Connected**

So FO-RT01 knows:

```text
1.1.1.1/32 → Loopback1
```

directly.

---

# 6. FO-RT02 did the same thing

You configured:

```cisco
interface loopback 2
 ip address 2.2.2.2 255.255.255.255
```

and:

```cisco
router ospf 1
 network 2.2.2.2 0.0.0.0 area 0
```

Therefore FO-RT02 now has:

```text
Loopback2
2.2.2.2/32
```

and OSPF uses:

```text
Router ID = 2.2.2.2
```

Your output confirms:

```text
Routing Process "ospf 1" with ID 2.2.2.2
```

So the architecture is becoming:

```text
              OSPF Area 0

       FO-RT01              FO-RT02
       RID 1.1.1.1          RID 2.2.2.2
           |                     |
     Lo1 1.1.1.1           Lo2 2.2.2.2
```

---

# 7. The OSPF adjacency messages

You have messages such as:

```text
%OSPF-5-ADJCHG:
Process 1, Nbr 2.2.2.2
on FastEthernet0/0.40
from LOADING to FULL
```

`FULL` is the important state.

It means the two OSPF routers have successfully synchronized their LSDBs for that adjacency.

You also saw:

```text
Nbr 216.0.5.2 on Serial0/0/0
from LOADING to FULL
```

So OSPF has successfully formed adjacencies with the relevant neighbors.

---

# 8. NTP: FO-RT01 as the time source

You configured:

```cisco
clock timezone IST 5 30
clock set 4:40:40 sept 10 2026
ntp master 1
```

The important distinction is:

### `clock set`

Sets the router's current clock.

```cisco
clock set 4:40:40 sept 10 2026
```

This is manually setting the time.

### `clock timezone`

Changes how the time is displayed.

```cisco
clock timezone IST 5 30
```

It means:

```text
UTC + 5:30
```

### `ntp master 1`

Turns the router into an **NTP master**.

Your output:

```text
*~127.127.1.1   .LOCL.   0
```

is showing the router's local clock as the NTP reference.

---

# 9. Why does NTP show `127.127.1.1`?

This is a Cisco NTP concept that often looks strange initially.

You aren't actually connecting to another device at:

```text
127.127.1.1
```

It represents a **local reference clock** used by the Cisco implementation.

Your output:

```text
*~127.127.1.1
 .LOCL.
 st 0
```

basically means:

> FO-RT01 is using its own local clock as its NTP reference.

So:

```text
FO-RT01
   |
   +--- Local clock
           |
           +--- NTP master
```

---

# 10. FO-RT02 is also configured as NTP master

You did:

```cisco
ntp master 1
```

on FO-RT02 too.

So currently you have:

```text
FO-RT01
NTP master
    |
    |


FO-RT02
NTP master
```

This is important because the intended architecture from the Castle Rysen lesson is generally:

```text
             Fallout Shelter
              NTP Master
                  |
          +-------+-------+
          |               |
       Cafe 1           Cafe 2
       NTP client       NTP client
```

Rather than making every router an NTP master.

So if the exercise is asking you to implement the intended hierarchy, **FO-RT01/FO-RT02 should be the authoritative time sources, while the cafe routers should synchronize from them.**

---

# 11. Your cafe NTP configuration

You configured:

```cisco
ntp server 1.1.1.1
```

That means:

> Cafe01 should synchronize its clock from the NTP server at 1.1.1.1.

And because:

```text
1.1.1.1
```

is FO-RT01's Loopback1, the intended architecture is:

```text
              FO-RT01
             1.1.1.1
           NTP Master
                |
                |
             OSPF/WAN
                |
                |
          cafe01-RT01
             NTP client
```

That's exactly why the Loopback becomes useful.

---

# 12. But your cafe output shows a problem

You got:

```text
~1.1.1.1       .INIT.   16
```

while you also have:

```text
~172.16.0.1    127.127.1.1   1
```

This tells us something important.

The cafe router is **not successfully synchronizing with 1.1.1.1**.

The key fields are:

```text
1.1.1.1
.INIT.
st 16
reach 0
```

### `st 16`

Stratum 16 means:

> This source is currently considered unsynchronized/unusable.

### `reach 0`

The cafe has not successfully received NTP responses from that server.

### `.INIT.`

The NTP association hasn't successfully initialized.

So:

```text
ntp server 1.1.1.1
```

being present in the configuration does **not** mean synchronization is working.

This is a very important troubleshooting lesson.

---

# 13. Why is `172.16.0.1` appearing?

Your cafe router has:

```text
Serial0/1/0
172.16.0.2
```

and FO-RT01 has:

```text
Serial0/0/0
172.16.0.1
```

So:

```text
FO-RT01                         cafe01
172.16.0.1  ----------------  172.16.0.2
```

The output:

```text
~172.16.0.1
127.127.1.1
1
```

shows the cafe is receiving NTP from:

```text
172.16.0.1
```

at stratum 1.

This is actually a strong clue that somewhere in your configuration/topology, the cafe is learning NTP from the physical FO-RT01 address as well.

But your displayed running configuration only shows:

```cisco
ntp server 1.1.1.1
```

So there may be a pre-existing NTP association/configuration or simulator behavior involved.

---

# 14. The huge NTP offset is also a clue

You have:

```text
offset 631244926.00
```

That's enormous.

This is because your simulated devices initially had wildly different clocks.

For example, you saw:

```text
FO-RT01
Wed Mar 24 1993
```

and later manually changed it to:

```text
Thu Sep 10 2026
```

while the cafe showed:

```text
Wed Sep 2 2026
```

So your Packet Tracer environment has devices starting with different simulated dates/times.

NTP is supposed to correct that difference.

---

# 15. DHCP Option 42

You attempted:

```cisco
ip dhcp pool ADMIN-10
 option 42 ip 10.0.18.1
```

and Packet Tracer responded:

```text
%This version of PT does not support options other than 43 and 150
```

This is **not a syntax mistake**.

Your command is conceptually valid:

```text
DHCP Option 42
      ↓
NTP Server
```

The problem is that this version of Packet Tracer doesn't implement DHCP Option 42.

So don't interpret:

```text
This version of PT does not support...
```

as:

> "Option 42 is invalid in real Cisco networking."

It means:

> "This Packet Tracer implementation doesn't support this DHCP option."

That's an important distinction between **Cisco IOS capability** and **Packet Tracer simulation capability**.

---

# 16. What Option 42 would accomplish

Without Option 42, a DHCP client might receive:

```text
IP address
10.0.18.10

Subnet mask
255.255.255.224

Default gateway
10.0.18.1

DNS
1.1.1.1
```

But it doesn't automatically know:

```text
Which NTP server should I use?
```

DHCP Option 42 can provide:

```text
NTP Server = 10.0.18.1
```

Conceptually:

```text
                    DHCP
                     |
        +------------+------------+
        |            |            |
       IP          Gateway        DNS
   10.0.18.10      .1          1.1.1.1
                     |
                     |
                    NTP
```

However, your Packet Tracer version won't let you configure Option 42.

---

# 17. One thing I would change in your design

You currently have:

```cisco
ntp server 1.1.1.1
```

and also earlier configured:

```cisco
ntp master 1
```

on the cafe router.

Those represent two different roles.

### NTP server/master

```text
"Get time from me."
```

### NTP client

```text
"Get time from someone else."
```

For the intended Castle Rysen hierarchy, think:

```text
          FO-RT01
        NTP Master
        1.1.1.1
             |
             |
       +-----+-----+
       |           |
     Cafe 1      Cafe 2
     Client      Client
```

So the cafe should normally be an NTP client rather than independently acting as an NTP master.

---

# 18. The entire service architecture you're building

Your work is actually tying several CCNA topics together:

```text
                         CASTLE RYSEN

                    Fallout Shelter
                    ┌───────────────┐
                    │    FO-RT01    │
                    │               │
                    │ Loopback      │
                    │ 1.1.1.1/32   │
                    │               │
                    │ DNS           │
                    │ NTP Master    │
                    │ OSPF          │
                    └───────┬───────┘
                            │
                         OSPF/WAN
                            │
                    ┌───────┴───────┐
                    │ cafe01-RT01   │
                    │               │
                    │ DHCP          │
                    │ DNS client    │
                    │ NTP client    │
                    │ SSH           │
                    │ NAT           │
                    │ ACL           │
                    └───────────────┘
```

And that is the bigger point of this lesson: **you're no longer configuring isolated commands. You're building an IP-services architecture.**

The most important troubleshooting item from your output is therefore:

> **Cafe01 has `ntp server 1.1.1.1` configured, but `show ntp associations` shows `1.1.1.1` at stratum 16 with reach 0, so it is not successfully synchronized to that Loopback yet.**

That is the next thing to troubleshoot—not the DHCP Option 42 error, which is simply a Packet Tracer limitation. 
