# Three Steps to Class C Subnetting

**Skill 11 - Lesson 03**

---

# Objective

Learn how to divide a Class C (/24) network into multiple smaller subnetworks based on the required number of subnets.

This lesson introduces the complete subnetting process used throughout CCNA.

---

# Why Do We Need Subnetting?

Without subnetting:

```text
192.168.10.0/24
```

* 1 Network
* 254 Usable Hosts

This wastes IP addresses when small networks are needed.

Example:

* Coffee Shop A → 8 devices
* Coffee Shop B → 10 devices
* Coffee Shop C → 6 devices

Instead of giving every location 254 addresses, we divide one large network into many smaller networks.

---

# The Core Concept

> **Subnetting is all about giving up host bits to create more network bits.**

Trade-off:

| More Networks | Fewer Hosts |
| ------------- | ----------- |
| ✅             | ❌           |

Borrowing more host bits:

* Creates more subnetworks
* Reduces the number of hosts per subnet

---

# Binary View of a /24 Network

Original Network:

```text
192.168.10.0/24
```

Subnet Mask:

```text
255.255.255.0
```

Binary:

```text
11111111.11111111.11111111.00000000
```

Meaning:

```text
1 = Network Bit
0 = Host Bit
```

Visualization:

```text
NNNNNNNN.NNNNNNNN.NNNNNNNN.HHHHHHHH
```

---

# Three-Step Subnetting Process

## Step 1 — Determine Required Subnets

Question:

```text
How many subnetworks are required?
```

Formula:

```text
2^x ≥ Required Subnets
```

Where:

* x = Number of borrowed host bits

Example:

Need:

```text
25 Subnets
```

Calculation:

```text
2^4 = 16 ❌

2^5 = 32 ✅
```

Borrow:

```text
5 Host Bits
```

---

## Step 2 — Create the New Subnet Mask

Original:

```text
/24
```

Borrow:

```text
5 bits
```

New Prefix:

```text
24 + 5 = /29
```

Binary:

```text
11111111.11111111.11111111.11111000
```

Decimal:

```text
255.255.255.248
```



# Understanding the Increment

The increment determines where each subnet begins.

Formula:

```text
Increment = 256 − Interesting Octet
```

Example:

```text
Mask

255.255.255.248
```

Interesting Octet:

```text
248
```

Calculation:

```text
256 − 248 = 8
```

Increment:

```text
8
```



## Step 3 — List Subnet Networks

Start:

```text
192.168.10.0
```

Add the increment repeatedly:

```text
0
8
16
24
32
40
48
56
...
```

Subnet Networks:

```text
192.168.10.0
192.168.10.8
192.168.10.16
192.168.10.24
...
```



# Understanding One /29 Subnet

Example:

```text
192.168.10.0/29
```

Addresses:

```text
192.168.10.0
↓

192.168.10.7
```

Breakdown:

| Address                     | Purpose           |
| --------------------------- | ----------------- |
| 192.168.10.0                | Network Address   |
| 192.168.10.1 – 192.168.10.6 | Usable Hosts      |
| 192.168.10.7                | Broadcast Address |

---

# Host Calculation

Formula:

```text
2^(Host Bits) − 2
```

For /29

Remaining Host Bits:

```text
3
```

Calculation:

```text
2³ − 2

8 − 2

6 Usable Hosts
```

Each /29 subnet contains:

* 8 Total Addresses
* 6 Usable Hosts
* 1 Network Address
* 1 Broadcast Address



# Example 1

Network:

```text
192.168.10.0/24
```

Requirement:

```text
25 Subnets
```

Solution:

| Item          | Value           |
| ------------- | --------------- |
| Borrowed Bits | 5               |
| New Prefix    | /29             |
| Subnet Mask   | 255.255.255.248 |
| Increment     | 8               |
| Usable Hosts  | 6               |

Subnet Starts:

```text
0
8
16
24
32
40
48
...
```



# Example 2

Network:

```text
215.1.1.0/24
```

Requirement:

```text
10 Subnets
```

Step 1:

```text
2⁴ = 16

Borrow 4 Bits
```

Step 2:

```text
/24 + 4

=

/28
```

Mask:

```text
255.255.255.240
```

Increment:

```text
256 − 240

=

16
```

Subnet Starts:

```text
215.1.1.0
215.1.1.16
215.1.1.32
215.1.1.48
215.1.1.64
215.1.1.80
215.1.1.96
215.1.1.112
215.1.1.128
215.1.1.144
...
```

Hosts Per Subnet:

```text
2⁴ − 2

=

14
```

---

# Common CIDR Reference

| CIDR | Mask            | Increment | Usable Hosts |
| ---- | --------------- | --------- | ------------ |
| /25  | 255.255.255.128 | 128       | 126          |
| /26  | 255.255.255.192 | 64        | 62           |
| /27  | 255.255.255.224 | 32        | 30           |
| /28  | 255.255.255.240 | 16        | 14           |
| /29  | 255.255.255.248 | 8         | 6            |
| /30  | 255.255.255.252 | 4         | 2            |

---

# Real-World Uses

### /29

Small branch offices

* Small coffee shops
* Retail stores
* Small offices
* IoT networks

### /30

Point-to-point links

```text
Router A
     │
     │
Router B
```

Only two usable IP addresses are needed.


# Example 3

Network:

```text
215.1.1.0/24
```

Requirement:

```text
Need 10 Subnets
```

---

## Step 1 — Determine Borrowed Bits

Formula:

```text
2^x ≥ Required Subnets
```

Calculation:

```text
2^1 = 2
2^2 = 4
2^3 = 8
2^4 = 16 ✅
```

Need to borrow:

```text
4 Host Bits
```

---

## Step 2 — Create the New Subnet Mask

Original Prefix:

```text
/24
```

Borrowed Bits:

```text
4
```

New Prefix:

```text
24 + 4 = /28
```

Binary Mask:

```text
11111111.11111111.11111111.11110000
```

Decimal Mask:

```text
255.255.255.240
```

---

## Step 3 — Find the Increment

Formula:

```text
Increment = 256 − Interesting Octet
```

Calculation:

```text
256 − 240 = 16
```

Increment:

```text
16
```

This means every subnet starts **16 IP addresses** after the previous one.

---

## List the First 10 Subnets

| Subnet | Network Address | First Host  | Last Host   | Broadcast   |
| ------ | --------------- | ----------- | ----------- | ----------- |
| 1      | 215.1.1.0       | 215.1.1.1   | 215.1.1.14  | 215.1.1.15  |
| 2      | 215.1.1.16      | 215.1.1.17  | 215.1.1.30  | 215.1.1.31  |
| 3      | 215.1.1.32      | 215.1.1.33  | 215.1.1.46  | 215.1.1.47  |
| 4      | 215.1.1.48      | 215.1.1.49  | 215.1.1.62  | 215.1.1.63  |
| 5      | 215.1.1.64      | 215.1.1.65  | 215.1.1.78  | 215.1.1.79  |
| 6      | 215.1.1.80      | 215.1.1.81  | 215.1.1.94  | 215.1.1.95  |
| 7      | 215.1.1.96      | 215.1.1.97  | 215.1.1.110 | 215.1.1.111 |
| 8      | 215.1.1.112     | 215.1.1.113 | 215.1.1.126 | 215.1.1.127 |
| 9      | 215.1.1.128     | 215.1.1.129 | 215.1.1.142 | 215.1.1.143 |
| 10     | 215.1.1.144     | 215.1.1.145 | 215.1.1.158 | 215.1.1.159 |

---

## Host Calculation

Remaining Host Bits:

```text
8 − 4 = 4
```

Formula:

```text
2^HostBits − 2
```

Calculation:

```text
2^4 − 2

16 − 2

= 14 Usable Hosts
```

Each subnet contains:

* **16 Total Addresses**
* **14 Usable Host Addresses**
* **1 Network Address**
* **1 Broadcast Address**

---

## Final Summary

| Item                    | Value           |
| ----------------------- | --------------- |
| Original Network        | 215.1.1.0/24    |
| Required Subnets        | 10              |
| Borrowed Bits           | 4               |
| New Prefix              | /28             |
| Subnet Mask             | 255.255.255.240 |
| Increment               | 16              |
| Actual Subnets Created  | 16              |
| Usable Hosts per Subnet | 14              |

### Remember

```text
Need 10 Subnets
        │
        ▼
Borrow 4 Bits
        │
        ▼
/24 → /28
        │
        ▼
Mask = 255.255.255.240
        │
        ▼
Increment = 16
        │
        ▼
Subnets Start At:
0, 16, 32, 48, 64, 80, 96, 112, 128, 144...
```


---

# Key Formulas

### Number of Subnets

```text
2^Subnet Bits
```

### Usable Hosts

```text
2^Host Bits − 2
```

### Increment

```text
256 − Interesting Octet
```

### New Prefix

```text
Original Prefix + Borrowed Bits
```

---

# Key Takeaways

* Subnetting divides one large network into smaller subnetworks.
* Borrow host bits to create additional subnet bits.
* More subnet bits create more subnetworks but reduce available hosts.
* The increment determines where each subnet starts.
* Every subnet reserves:

  * First IP → Network Address
  * Last IP → Broadcast Address
* The same three-step process works for every Class C subnetting problem.


