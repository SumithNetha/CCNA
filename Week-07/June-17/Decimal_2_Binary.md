# notes.md — Converting Decimal to Binary

## Skill 11 — Lesson 02

## Converting Decimal to Binary

---

# Why Learn Binary?

Binary is not random math.

Binary exists because computers store information using only:

```text
0 = OFF
1 = ON
```

Subnetting depends on understanding binary.

If binary is skipped, subnet masks and CIDR become difficult later.

Think of binary as:

> Selecting fixed-value buckets to build a number.

---

# Binary Value Table (Memorize)

Each bit position has a fixed value.

| Bit Position  |   7 |  6 |  5 |  4 |  3 |  2 |  1 |  0 |
| ------------- | --: | -: | -: | -: | -: | -: | -: | -: |
| Decimal Value | 128 | 64 | 32 | 16 |  8 |  4 |  2 |  1 |

Rules:

```text
1 → Use the value
0 → Skip the value
```



# Decimal → Binary Conversion Process

Steps:

1. Start at 128
2. Ask:
   "Does this value fit?"
3. If YES:

   * Write 1
   * Subtract value
4. If NO:

   * Write 0
5. Move right until reaching 1



# Example 1 — Convert 153

Values:

```text
128 64 32 16 8 4 2 1
```

Work:

```text
153
−128 = 25
64 → No
32 → No
25−16 = 9
9−8 = 1
4 → No
2 → No
1−1 = 0
```

Binary:

```text
10011001
```

Check:

```text
128+16+8+1=153
```



# Example 2 — Convert 210

Work:

```text
210
−128 = 82
−64 = 18
32 → No
−16 = 2
8 → No
4 → No
−2 = 0
```

Binary:

```text
11010010
```

Check:

```text
128+64+16+2=210
```


# Example 3 — Convert 20

Work:

```text
20
128 → No
64 → No
32 → No
−16 = 4
8 → No
−4 = 0
```

Binary:

```text
00010100
```

Check:

```text
16+4=20
```



# Why 255 Matters

Maximum value of one byte:

```text
11111111
```

Calculation:

```text
128+64+32+16+8+4+2+1
=255
```

Therefore:

```text
IPv4 Octet Range:
0–255
```

Examples:

```text
192.168.1.10
```

Binary:

```text
11000000.10101000.00000001.00001010
```

IPv4 contains:

```text
4 octets
8 bits each
32 bits total
```



# Binary Patterns (Important for Subnetting)

```text
128 = 10000000
192 = 11000000
224 = 11100000
240 = 11110000
248 = 11111000
252 = 11111100
254 = 11111110
255 = 11111111
```

Memorize these.

These become subnet masks later.



# Practice Questions

Convert:

```text
45
78
100
172
192
224
255
```

Check answers after solving.

---

# Key Takeaways

* Binary uses only 0 and 1
* One byte = 8 bits
* Max byte value = 255
* IPv4 = 32 bits
* Binary conversion = selecting values that fit
* Binary is the foundation of subnetting

---

Next Lesson:
Three Steps to Class C Subnetting
