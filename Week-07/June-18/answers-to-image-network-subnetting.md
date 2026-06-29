# Network Subnetting Practice Solutions

## Objective

Practice subnetting based on the **number of required subnets**.

**Remember:**

> If the question asks for **subnets**, determine how many **host bits to borrow**.

---

# Formula

```
2^Borrowed Bits ≥ Required Subnets
```

After determining the borrowed bits:

```
New Prefix = Original Prefix + Borrowed Bits
```

Then:

- Determine Subnet Mask
- Calculate Block Size (Increment)
- List Subnets

---

# Problem 1

## Question

```
Take 28.0.0.0/8

Break it into 60 Subnets
```

---

## Step 1 — Required Subnets

```
60
```

Find borrowed bits.

```
2^5 = 32 ❌

2^6 = 64 ✅
```

Borrow:

```
6 Bits
```

---

## Step 2 — New Prefix

Original

```
/8
```

Borrow

```
6
```

New Prefix

```
/14
```

---

## Step 3 — Subnet Mask

```
11111111.11111100.00000000.00000000
```

Decimal

```
255.252.0.0
```

---

## Step 4 — Increment

Interesting Octet

```
252
```

Increment

```
256 - 252

= 4
```

Increment occurs in the **Second Octet**.

---

## First Few Subnets

| Subnet | Network Address | Broadcast Address |
|---------|-----------------|-------------------|
| 1 | 28.0.0.0/14 | 28.3.255.255 |
| 2 | 28.4.0.0/14 | 28.7.255.255 |
| 3 | 28.8.0.0/14 | 28.11.255.255 |
| 4 | 28.12.0.0/14 | 28.15.255.255 |
| 5 | 28.16.0.0/14 | 28.19.255.255 |

---

## Summary

| Item | Value |
|------|------|
| Original Prefix | /8 |
| Required Subnets | 60 |
| Borrowed Bits | 6 |
| New Prefix | /14 |
| Subnet Mask | 255.252.0.0 |
| Increment | 4 (2nd Octet) |
| Total Subnets | 64 |
| Hosts/Subnet | 262,142 |

---

# Problem 2

## Question

```
Take 172.16.0.0/16

Break it into 45 Subnets
```

---

## Step 1 — Required Subnets

```
45
```

```
2^5 = 32 ❌

2^6 = 64 ✅
```

Borrow

```
6 Bits
```

---

## Step 2 — New Prefix

Original

```
/16
```

Borrow

```
6
```

New Prefix

```
/22
```

---

## Step 3 — Subnet Mask

Binary

```
11111111.11111111.11111100.00000000
```

Decimal

```
255.255.252.0
```

---

## Step 4 — Increment

Interesting Octet

```
252
```

Increment

```
256 - 252

= 4
```

Increment occurs in the **Third Octet**.

---

## First Few Subnets

| Subnet | Network Address | Broadcast Address |
|---------|-----------------|-------------------|
| 1 | 172.16.0.0/22 | 172.16.3.255 |
| 2 | 172.16.4.0/22 | 172.16.7.255 |
| 3 | 172.16.8.0/22 | 172.16.11.255 |
| 4 | 172.16.12.0/22 | 172.16.15.255 |
| 5 | 172.16.16.0/22 | 172.16.19.255 |

---

## Summary

| Item | Value |
|------|------|
| Original Prefix | /16 |
| Required Subnets | 45 |
| Borrowed Bits | 6 |
| New Prefix | /22 |
| Subnet Mask | 255.255.252.0 |
| Increment | 4 (3rd Octet) |
| Total Subnets | 64 |
| Hosts/Subnet | 1022 |

---

# Problem 3

## Question

```
Take 210.40.20.0/24

Break it into 12 Subnets
```

---

## Step 1 — Required Subnets

```
12
```

```
2^3 = 8 ❌

2^4 = 16 ✅
```

Borrow

```
4 Bits
```

---

## Step 2 — New Prefix

Original

```
/24
```

Borrow

```
4
```

New Prefix

```
/28
```

---

## Step 3 — Subnet Mask

```
255.255.255.240
```

---

## Step 4 — Increment

Interesting Octet

```
240
```

Increment

```
256 - 240

= 16
```

Increment occurs in the **Fourth Octet**.

---

## All Subnets

| Subnet | Network | Broadcast |
|---------|---------|-----------|
| 1 | 210.40.20.0 | 210.40.20.15 |
| 2 | 210.40.20.16 | 210.40.20.31 |
| 3 | 210.40.20.32 | 210.40.20.47 |
| 4 | 210.40.20.48 | 210.40.20.63 |
| 5 | 210.40.20.64 | 210.40.20.79 |
| 6 | 210.40.20.80 | 210.40.20.95 |
| 7 | 210.40.20.96 | 210.40.20.111 |
| 8 | 210.40.20.112 | 210.40.20.127 |
| 9 | 210.40.20.128 | 210.40.20.143 |
| 10 | 210.40.20.144 | 210.40.20.159 |
| 11 | 210.40.20.160 | 210.40.20.175 |
| 12 | 210.40.20.176 | 210.40.20.191 |
| 13 | 210.40.20.192 | 210.40.20.207 |
| 14 | 210.40.20.208 | 210.40.20.223 |
| 15 | 210.40.20.224 | 210.40.20.239 |
| 16 | 210.40.20.240 | 210.40.20.255 |

---

## Summary

| Item | Value |
|------|------|
| Original Prefix | /24 |
| Required Subnets | 12 |
| Borrowed Bits | 4 |
| New Prefix | /28 |
| Subnet Mask | 255.255.255.240 |
| Increment | 16 (4th Octet) |
| Total Subnets | 16 |
| Hosts/Subnet | 14 |

---

# Quick Reference Table

| Network | Original Prefix | Required Subnets | Borrowed Bits | New Prefix | Subnet Mask | Increment | Total Subnets |
|---------|-----------------|-----------------:|--------------:|-----------:|----------------|-----------|---------------:|
| 28.0.0.0 | /8 | 60 | 6 | /14 | 255.252.0.0 | 4 (2nd Octet) | 64 |
| 172.16.0.0 | /16 | 45 | 6 | /22 | 255.255.252.0 | 4 (3rd Octet) | 64 |
| 210.40.20.0 | /24 | 12 | 4 | /28 | 255.255.255.240 | 16 (4th Octet) | 16 |

---

# Network-Based Subnetting Cheat Sheet

## Step 1

Read the question.

```
Need Subnets?
```

↓

Use

```
2^n ≥ Required Subnets
```

---

## Step 2

Borrow host bits.

```
New Prefix

=

Original Prefix

+

Borrowed Bits
```

---

## Step 3

Determine the subnet mask.

---

## Step 4

Find the **Interesting Octet**.

The interesting octet is the **first octet in the subnet mask that is neither 255 nor 0**.

Examples:

| Mask | Interesting Octet |
|------|--------------------|
| 255.252.0.0 | Second |
| 255.255.252.0 | Third |
| 255.255.255.240 | Fourth |

---

## Step 5

Calculate the Increment.

```
Increment

=

256

-

Interesting Octet
```

Examples:

```
256 - 252 = 4

256 - 248 = 8

256 - 240 = 16

256 - 224 = 32

256 - 192 = 64

256 - 128 = 128
```

---

## Step 6

Generate subnet addresses using the increment until the address space is exhausted.

---

# Common Mistakes

❌ Using the host-subnetting formula (`2ⁿ - 2`) instead of the network-subnetting formula (`2ⁿ`).

❌ Forgetting to add borrowed bits to the original prefix.

❌ Calculating the increment from the wrong octet.

❌ Stopping after generating only the required number of subnets instead of understanding that borrowing bits creates all possible subnets.

❌ Confusing the **number of subnets** with the **number of hosts per subnet**.

---

# Exam Tips

- If the question says **"Need X Subnets"**, think **Borrow Bits**.
- Use **2ⁿ ≥ Required Subnets**.
- Always identify the **interesting octet** before calculating the increment.
- Practice with Class A, Class B, and Class C networks—the process is identical.
- Memorize common subnet masks and increments to solve problems quickly during the CCNA exam.