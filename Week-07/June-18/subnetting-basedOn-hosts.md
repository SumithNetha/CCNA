# Subnetting Based on Host Requirements

## Objective

Subnet a network based on the **number of hosts required per subnet** rather than the number of subnets.

**Memory Trick:** **Save the Host**

---

# Difference from Previous Method

## Subnet-Based Subnetting

Question:
> How many subnets do I need?

Focus:
- Borrow host bits.
- Create more networks.

---

## Host-Based Subnetting

Question:
> How many hosts are needed in each subnet?

Focus:
- Preserve host bits.
- Ensure enough IP addresses for devices.

---

# Key Concept

When a subnetting problem specifies the number of hosts:

- Protect enough host bits.
- Remaining bits become network bits.
- Build the new subnet mask.

Think:

```
Hosts First
↓

Save Host Bits
↓

Subnet What's Left
```

---

# Formula

```
2^HostBits - 2 ≥ Required Hosts
```

Subtract 2 because every subnet reserves:

- Network Address
- Broadcast Address

---

# Host Bit Reference Table

| Host Bits | Total Addresses | Usable Hosts | Prefix | Subnet Mask |
|------------|-----------------|--------------|--------|-----------------|
| 2 | 4 | 2 | /30 | 255.255.255.252 |
| 3 | 8 | 6 | /29 | 255.255.255.248 |
| 4 | 16 | 14 | /28 | 255.255.255.240 |
| 5 | 32 | 30 | /27 | 255.255.255.224 |
| 6 | 64 | 62 | /26 | 255.255.255.192 |
| 7 | 128 | 126 | /25 | 255.255.255.128 |
| 8 | 256 | 254 | /24 | 255.255.255.0 |
| 9 | 512 | 510 | /23 | 255.255.254.0 |
| 10 | 1024 | 1022 | /22 | 255.255.252.0 |
| 11 | 2048 | 2046 | /21 | 255.255.248.0 |
| 12 | 4096 | 4094 | /20 | 255.255.240.0 |

---

# Steps

1. Read the question carefully.
2. Identify the required number of hosts.
3. Find the minimum host bits using powers of two.
4. Preserve (save) those host bits.
5. Remaining bits become network bits.
6. Calculate the new prefix length.
7. Determine the subnet mask.
8. Calculate the block size (increment).
9. List subnet ranges.

---

# Example 1

## Requirement

```
Network: 172.16.10.0

Hosts Needed: 50
```

### Step 1

```
2^5 - 2 = 30 ❌
2^6 - 2 = 62 ✅
```

Host Bits = **6**

### Step 2

```
Prefix

32 - 6 = /26
```

### Step 3

Subnet Mask

```
255.255.255.192
```

### Step 4

Increment

```
256 - 192 = 64
```

### Subnets

```
172.16.10.0

172.16.10.64

172.16.10.128

172.16.10.192
```

### Hosts per Subnet

```
62 Usable Hosts
```

---

# Example 2

## Requirement

```
Network: 172.16.0.0

Hosts Needed: 500
```

### Host Bits

```
2^8 - 2 = 254 ❌

2^9 - 2 = 510 ✅
```

Host Bits = **9**

### Prefix

```
32 - 9 = /23
```

### Mask

```
255.255.254.0
```

### Increment

```
256 - 254 = 2
```

Third Octet:

```
172.16.0.0

172.16.2.0

172.16.4.0

172.16.6.0
```

### Hosts

```
510 Usable Hosts
```

---

# Example 3

## Requirement

```
Network: 10.0.0.0

Hosts Needed: 2000
```

### Host Bits

```
2^10 - 2 = 1022 ❌

2^11 - 2 = 2046 ✅
```

Host Bits = **11**

### Prefix

```
32 - 11 = /21
```

### Mask

```
255.255.248.0
```

### Increment

```
256 - 248 = 8
```

Third Octet

```
10.0.0.0

10.0.8.0

10.0.16.0

10.0.24.0
```

### Hosts

```
2046 Usable Hosts
```

---

# Quick Decision Guide

```
Question asks:

Number of Subnets?

↓

Borrow Host Bits


Question asks:

Number of Hosts?

↓

Save Host Bits
```

---

# Real-World Usage

Network administrators usually design networks based on:

- PCs
- Laptops
- Servers
- Printers
- IP Phones
- Cameras
- Wireless Devices

Instead of creating subnets with exactly the required number of hosts, allocate extra addresses for future expansion to avoid redesigning the IP addressing scheme later.

---

# Common Mistakes

❌ Forgetting to subtract 2 for Network and Broadcast addresses.

❌ Using the subnet-count method when the question asks for hosts.

❌ Choosing a subnet that is too small.

❌ Forgetting to calculate the increment after determining the subnet mask.

❌ Not leaving room for future growth.

---

# Exam Tips

- Read the question carefully before solving.
- If it mentions **hosts**, preserve host bits.
- Memorize the Host Bit Reference Table.
- Practice calculating subnet masks and increments without a calculator.
- Always verify that usable hosts meet or exceed the requirement.

---

# Key Takeaways

- Host-based subnetting starts with the required number of devices.
- Use **2ⁿ − 2 ≥ Required Hosts** to determine host bits.
- Preserve those host bits and subnet the remaining bits.
- Build the subnet mask and calculate the increment.
- Plan IP addressing with room for future growth.

---

## Memory Trick

```
Need Hosts?

↓

SAVE THE HOST


Need Subnets?

↓

BORROW THE HOST BITS
```