# Three Steps to Class B and Class A Subnetting

**Skill 11 – Lesson 04**

---

# Objective

Learn how to subnet **Class B** and **Class A** networks using the **same three-step process** used for Class C subnetting.

> **Important:** The subnetting process never changes. Only the size of the network and the octet where the increment appears changes.

---

# Default Network Classes

| Class   | Default Network | Default Prefix | Default Mask  |
| ------- | --------------- | -------------- | ------------- |
| Class A | 10.0.0.0        | /8             | 255.0.0.0     |
| Class B | 172.16.0.0      | /16            | 255.255.0.0   |
| Class C | 192.168.1.0     | /24            | 255.255.255.0 |

---

# The Three-Step Process

Every subnetting problem follows the same steps:

### Step 1

Determine how many subnet bits are required.

```text
2^x ≥ Required Subnets
```

---

### Step 2

Borrow host bits.

```text
New Prefix

=

Default Prefix + Borrowed Bits
```

---

### Step 3

Find the increment.

```text
Increment

=

256 − Interesting Octet
```

Use the increment to list all subnet networks.

---

# Class B Subnetting

Default Network:

```text
172.16.0.0/16
```

Binary Layout:

```text
Network      Network       Host         Host
11111111 . 11111111 . 00000000 . 00000000
```

Diagram:

```text
172 .16 .0 .0
 N    N   H   H
```

A Class B network has **16 host bits** (last two octets).

---

# Example 1

Network:

```text
172.16.0.0/16
```

Requirement:

```text
100 Subnets
```

---

## Step 1 – Borrow Bits

Formula:

```text
2^x ≥ 100
```

Calculation:

```text
2^6 = 64 ❌

2^7 = 128 ✅
```

Borrow:

```text
7 Bits
```

---

## Step 2 – New Mask

Original Prefix:

```text
/16
```

Borrow:

```text
7 Bits
```

New Prefix:

```text
/23
```

Binary Mask:

```text
11111111.11111111.11111110.00000000
```

Decimal Mask:

```text
255.255.254.0
```

---

## Step 3 – Find Increment

Interesting Octet:

```text
254
```

Calculation:

```text
256 − 254

=

2
```

Increment:

```text
2
```

Subnet Networks:

```text
172.16.0.0

172.16.2.0

172.16.4.0

172.16.6.0

172.16.8.0
...
```

---

## Hosts Per Subnet

Host Bits Remaining:

```text
9
```

Formula:

```text
2^9 − 2
```

Result:

```text
510 Usable Hosts
```

---

# Oreo Rule

Each subnet reserves:

* First Address → Network Address
* Last Address → Broadcast Address

Everything between them is usable.

Example:

Subnet:

```text
172.16.2.0/23
```

| Address                   | Purpose      |
| ------------------------- | ------------ |
| 172.16.2.0                | Network      |
| 172.16.2.1 – 172.16.3.254 | Usable Hosts |
| 172.16.3.255              | Broadcast    |

### Important

Addresses ending in:

```text
.x.x.0
```

or

```text
.x.x.255
```

are **not always reserved**.

They are only reserved if they are the **network** or **broadcast** address of that subnet.

---

# Example 2

Network:

```text
172.20.0.0/16
```

Requirement:

```text
500 Subnets
```

---

## Step 1

```text
2^8 = 256 ❌

2^9 = 512 ✅
```

Borrow:

```text
9 Bits
```

---

## Step 2

Original Prefix:

```text
/16
```

Borrow:

```text
9
```

New Prefix:

```text
/25
```

Subnet Mask:

```text
255.255.255.128
```

---

## Step 3

Interesting Octet:

```text
128
```

Increment:

```text
256 − 128

=

128
```

Subnet Networks:

```text
172.20.0.0

172.20.0.128

172.20.1.0

172.20.1.128

172.20.2.0

172.20.2.128
...
```

### Octet Rollover

When an octet exceeds **255**, it resets to **0** and the previous octet increases by one.

Example:

```text
172.20.0.128

+128

=

172.20.1.0
```

This is called **octet rollover**.

---

# Class A Subnetting

Default Network:

```text
10.0.0.0/8
```

Binary Layout:

```text
Network       Host       Host       Host
11111111 . 00000000 . 00000000 . 00000000
```

Diagram:

```text
10 .0 .0 .0

 N  H  H  H
```

A Class A network has **24 host bits**.

---

# Example 3

Network:

```text
10.0.0.0/8
```

Requirement:

```text
1000 Subnets
```

---

## Step 1

```text
2^10 = 1024 ✅
```

Borrow:

```text
10 Bits
```

---

## Step 2

Original Prefix:

```text
/8
```

Borrow:

```text
10
```

New Prefix:

```text
/18
```

Mask:

```text
255.255.192.0
```

---

## Step 3

Interesting Octet:

```text
192
```

Calculation:

```text
256 − 192

=

64
```

Increment:

```text
64
```

Subnet Networks:

```text
10.0.0.0

10.0.64.0

10.0.128.0

10.0.192.0

10.1.0.0

10.1.64.0
...
```

### Why Does It Become 10.1.0.0?

After:

```text
10.0.192.0
```

Adding another **64** gives:

```text
10.0.256.0
```

Since an octet cannot exceed **255**, it rolls over:

```text
10.1.0.0
```

This is called **octet rollover**.

---

# Quick Comparison

| Network Class | Default Prefix | Common Increment Location |
| ------------- | -------------- | ------------------------- |
| Class C       | /24            | 4th Octet                 |
| Class B       | /16            | 3rd or 4th Octet          |
| Class A       | /8             | 2nd, 3rd or 4th Octet     |

---

# Memory Tips

### Class C

Increment usually changes the **4th octet**.

Example:

```text
192.168.1.0

192.168.1.16

192.168.1.32
```

---

### Class B

Increment usually changes the **3rd octet**.

Example:

```text
172.16.0.0

172.16.2.0

172.16.4.0
```

---

### Class A

Increment may change the **2nd or 3rd octet**.

Example:

```text
10.0.0.0

10.0.64.0

10.0.128.0

10.1.0.0
```

---

# Key Formulas

### Number of Subnets

```text
2^Borrowed Bits
```

### Usable Hosts

```text
2^Host Bits − 2
```

### New Prefix

```text
Default Prefix + Borrowed Bits
```

### Increment

```text
256 − Interesting Octet
```

---

# Key Takeaways

* Class A, B, and C use the **same subnetting process**.
* The only difference is the **default prefix** and the **octet where the increment appears**.
* When an octet exceeds **255**, perform **octet rollover**.
* Do **not** assume addresses ending in **.0** or **.255** are always reserved.
* Always identify the **network** and **broadcast** addresses for the current subnet before deciding whether an address is usable.
* Trust the same three-step method regardless of network class.

---

## Folder Structure

```text
week-07/
└── june-17/
    ├── notes.md
    ├── subnetting-class-c.md
    ├── subnetting-class-b-a.md
    └── practice.md
```
