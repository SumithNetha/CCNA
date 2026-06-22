notes.md

# Week 7 Day 1 – NAT Overload (PAT)

Date: June 15

## Topic

Configuring NAT Overload

---

## What is NAT Overload

NAT Overload allows multiple private IP addresses to share one public IP address.

Router differentiates traffic using source ports.

PAT = Port Address Translation

Example:

192.168.1.10:5000
↓

216.0.5.2:30001

192.168.1.20:5000
↓

216.0.5.2:30002

Same public IP.
Different ports.

---

## Dynamic NAT vs PAT

Dynamic NAT

Private → Public

192.168.1.10
↓

216.0.5.50

PAT

Private → Shared Public

192.168.1.10:5000
↓

216.0.5.2:30001

---

## Why PAT Exists

IPv4 public addresses are limited.

Without PAT:

# 100 Devices

100 Public IPs

With PAT:

# 100 Devices

1 Public IP

---

## Interface Method

Command:

ip nat inside source list 1 interface g0/1 overload

Meaning:

Translate inside devices
using interface IP
and distinguish using ports.

---

## Pool Method

Command:

ip nat inside source list 1 pool PUBLIC overload

Router can spread sessions across multiple public IPs.

---

## Why Interface Method is Popular

ISP public addresses change.

Router automatically uses current interface IP.

No manual updates.

---

## Verification

show ip nat translations

show ip nat statistics

show access-lists

show ip interface brief

---

## Key Takeaways

PAT = Many → One

Uses ports

Most common NAT deployment

Foundation of real internet access
