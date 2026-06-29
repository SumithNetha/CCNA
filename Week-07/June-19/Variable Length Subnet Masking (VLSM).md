# Variable Length Subnet Masking (VLSM)

## Definition

Variable Length Subnet Masking (VLSM) is a subnetting technique that allows multiple subnet masks of different sizes to be used within the same network. Each subnet receives only the number of IP addresses it actually requires, reducing address wastage.

Unlike Fixed Length Subnet Masking (FLSM), where every subnet has the same size, VLSM creates subnets based on individual host requirements.

---

# Why VLSM is Needed

Using the same subnet size for every department wastes IPv4 addresses.

Example:

| Department | Hosts Required |
|------------|---------------:|
| Sales | 100 |
| HR | 40 |
| Management | 12 |
| WAN Link | 2 |

Using FLSM:

- Every subnet becomes /25
- Every subnet gets 126 usable hosts
- Hundreds of addresses are wasted

Using VLSM:

| Department | Prefix | Usable Hosts |
|------------|--------|-------------:|
| Sales | /25 | 126 |
| HR | /26 | 62 |
| Management | /28 | 14 |
| WAN | /30 | 2 |

Result:
- Minimal address wastage
- Efficient IPv4 utilization
- Easier network expansion

---

# Advantages

- Efficient use of IPv4 addresses
- Reduces address wastage
- Supports networks of different sizes
- Improves scalability
- Reduces broadcast domains
- Standard practice in enterprise networks
- Required knowledge for CCNA and CCNP

---

# FLSM vs VLSM

| Feature | FLSM | VLSM |
|---------|------|------|
|Subnet Size|Fixed|Variable|
|Subnet Mask|One mask|Multiple masks|
|Address Efficiency|Poor|Excellent|
|Scalability|Low|High|
|Address Waste|High|Low|

---

# Rules of VLSM

1. Sort host requirements from largest to smallest.
2. Allocate the largest subnet first.
3. Use the smallest subnet that satisfies the host requirement.
4. Do not overlap subnets.
5. Start the next subnet immediately after the previous broadcast address.

---

# Host Requirement Reference

| Hosts Needed | Prefix | Usable Hosts |
|--------------|--------|-------------:|
|254|/24|254|
|126|/25|126|
|62|/26|62|
|30|/27|30|
|14|/28|14|
|6|/29|6|
|2|/30|2|

---

# Example

Network:

192.168.1.0/24

Requirements:

| Department | Hosts |
|------------|-------|
| Sales |100|
| HR |50|
| Management |20|
| WAN |2|

Allocation:

| Department | Network | Prefix |
|------------|---------|--------|
| Sales |192.168.1.0|/25|
| HR |192.168.1.128|/26|
| Management |192.168.1.192|/27|
| WAN |192.168.1.224|/30|

---

# Common Mistakes

- Starting with the smallest subnet
- Choosing a subnet larger than required
- Creating overlapping subnets
- Forgetting future growth requirements

---

# Real-World Applications

- Department subnetting
- Server networks
- Guest Wi-Fi
- Voice VLANs
- Management VLANs
- WAN point-to-point links
- Enterprise network design
- Data center addressing

---

# Key Takeaways

- VLSM allows different subnet sizes within the same network.
- Allocate subnets from largest to smallest.
- Choose the smallest subnet that satisfies the host requirement.
- VLSM minimizes IPv4 address wastage.
- VLSM is the standard subnetting method used in modern enterprise networks.