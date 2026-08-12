# IPv6 Address Shortcuts

IPv6 addresses are 128-bit addresses written as **eight 16-bit hexadecimal groups**, called **hextets**. At first they look extremely long, but IPv6 provides rules for shortening them without changing the actual address.

The lesson focuses on **two main shortcuts**, plus the formatting recommendations from **RFC 5952**.

---

## 1. Shortcut #1 — Drop Leading Zeros

A hextet can contain up to four hexadecimal digits.

For example:

```text
0001
00AB
0ABC
```

You can remove **leading zeros**—zeros at the beginning of the hextet.

```text
0001 → 1
00AB → AB
0ABC → ABC
```

### What you cannot do

You cannot remove zeros from the middle or end of a hextet.

For example:

```text
AB10
```

cannot become:

```text
AB1
```

because the `0` is not a leading zero.

### Important rule

> **Only leading zeros can be removed.**

Think:

```text
000A → A       ✅
00AB → AB     ✅
0ABC → ABC    ✅

AB10 → AB1    ❌
```

You are not changing the value—you are only removing unnecessary padding.

---

# 2. Shortcut #2 — Replace Consecutive Zero Hextets with `::`

The second shortcut is much more powerful.

If an IPv6 address contains **one or more consecutive hextets containing all zeros**, they can be replaced with:

```text
::
```

For example:

```text
2001:0DB8:0000:0000:0000:0000:0000:0001
```

First remove leading zeros:

```text
2001:DB8:0:0:0:0:0:1
```

The consecutive zero hextets can then be compressed:

```text
2001:DB8::1
```

The `::` represents the omitted consecutive zero groups.

---

# 3. `::` Can Only Be Used Once

This is one of the most important IPv6 rules.

You can use:

```text
::
```

**only once in an IPv6 address.**

Why?

Because `::` represents an unknown number of zero hextets.

Consider:

```text
2001::ABCD::1
```

There is no way to determine exactly how many zero groups belong to the first `::` and how many belong to the second.

Therefore:

```text
2001::ABCD::1
```

is **invalid/ambiguous**.

### If there are two separate zero runs

You have to choose **one** of them to compress.

For example:

```text
2001:0:0:ABCD:0:0:0:1
```

You could write:

```text
2001::ABCD:0:0:0:1
```

or:

```text
2001:0:0:ABCD::1
```

But not:

```text
2001::ABCD::1
```

---

# 4. IPv6 Loopback Address — `::1`

A very important example is:

```text
::1
```

This is the **IPv6 loopback address**.

Its IPv4 equivalent is:

```text
127.0.0.1
```

The loopback address allows a machine to communicate with itself and test its local TCP/IP stack without sending traffic onto the physical network.

Full IPv6 representation:

```text
0000:0000:0000:0000:0000:0000:0000:0001
```

Remove leading zeros:

```text
0:0:0:0:0:0:0:1
```

Compress the consecutive zero groups:

```text
::1
```

So:

```text
IPv4 loopback → 127.0.0.1
IPv6 loopback → ::1
```

---

# 5. RFC 5952 — Standard IPv6 Text Representation

IPv6 addresses can technically be represented in multiple equivalent ways.

For example, these represent the same address:

```text
2001:0DB8:0000:0000:0000:0000:0000:0001
```

```text
2001:DB8:0:0:0:0:0:1
```

```text
2001:DB8::1
```

That can become confusing when different vendors or administrators display the same address differently.

**RFC 5952** provides recommendations for a consistent textual representation of IPv6 addresses.

The lesson highlights these important conventions.

### Leading zeros should be removed

Instead of:

```text
2001:0DB8:0001:0002
```

use:

```text
2001:DB8:1:2
```

---

## 6. Prefer `::` for the Longest Zero Run

When compression is possible, RFC 5952 recommends using `::` for the **longest sequence of consecutive zero hextets**.

Example:

```text
2001:0:0:0:ABCD:0:0:1
```

There are two zero runs:

```text
2001:0:0:0:ABCD:0:0:1
     └──────┘       └──┘
```

The first run is longer, so compress it:

```text
2001::ABCD:0:0:1
```

---

# 7. Single Zero Hextet — Don't Necessarily Use `::`

This is an important detail from the lesson.

Suppose the address contains only **one zero hextet**:

```text
2001:DB8:0:ABCD:1234:5678:9ABC:DEF0
```

You don't need to turn that single `0` into `::`.

The preferred representation is:

```text
2001:db8:0:abcd:1234:5678:9abc:def0
```

rather than trying to use `::`.

The reason is **clarity and consistency**, not simply making the address as short as possible.

---

# 8. Lowercase Hexadecimal

RFC 5952 recommends using **lowercase hexadecimal characters**.

Instead of:

```text
2001:DB8:ABCD::1
```

the preferred representation is:

```text
2001:db8:abcd::1
```

Both represent the same IPv6 address.

For standardized documentation, diagrams, tickets, and configurations, the lesson recommends following the consistent RFC 5952 style.

---

# 9. Complete Example

Start with:

```text
2001:0DB8:0000:0000:0000:0042:0000:0001
```

### Step 1 — Remove leading zeros

```text
2001:DB8:0:0:0:42:0:1
```

### Step 2 — Identify consecutive zero groups

```text
2001:DB8:0:0:0:42:0:1
         └─────┘
```

### Step 3 — Replace that run with `::`

```text
2001:DB8::42:0:1
```

### Step 4 — Apply lowercase formatting

RFC 5952-style representation:

```text
2001:db8::42:0:1
```

---

# 10. How to Expand `::`

You also need to be able to go **backward**.

Given:

```text
2001:db8::1
```

You know an IPv6 address must contain **8 hextets**.

Currently you have:

```text
2001
db8
1
```

That's 3 hextets.

Therefore:

```text
8 - 3 = 5
```

zero hextets must be represented by `::`.

So:

```text
2001:db8::1
```

expands to:

```text
2001:0db8:0000:0000:0000:0000:0000:0001
```

This is why `::` can only occur once: the receiver needs to know exactly where the missing zero groups belong.

---

# 11. Two Rules You Must Memorize

### Rule 1

**Remove leading zeros from each hextet.**

```text
0001 → 1
00AB → AB
0ABC → ABC
```

But:

```text
AB10 → AB1 ❌
```

---

### Rule 2

**Replace one consecutive sequence of all-zero hextets with `::`.**

```text
0:0:0 → ::
```

But:

```text
0:0:ABCD:0:0
```

cannot become:

```text
::ABCD::
```

because `::` can only be used **once**.

---

# 12. Quick Reference

| Rule                              | Example                       |
| --------------------------------- | ----------------------------- |
| IPv6 has 8 hextets                | `2001:db8:0:0:0:0:0:1`        |
| Remove leading zeros              | `000A → A`                    |
| Don't remove middle zeros         | `AB10 → AB10`                 |
| Don't remove trailing zeros       | `AB10 → AB1` ❌                |
| Compress consecutive zero hextets | `0:0:0 → ::`                  |
| Use `::` only once                | `2001::ABCD::1` ❌             |
| Prefer longest zero run           | Compress the longest sequence |
| Single zero hextet                | Usually leave as `0`          |
| Preferred hex case                | lowercase                     |
| IPv6 loopback                     | `::1`                         |
| IPv4 loopback                     | `127.0.0.1`                   |
| IPv6 standardization              | RFC 5952                      |

---

## 🧠 The Mental Process

Whenever you receive a full IPv6 address, use this sequence:

```text
FULL IPv6 ADDRESS
       ↓
Remove leading zeros
       ↓
Find consecutive zero hextets
       ↓
Compress ONE zero run with ::
       ↓
Use the longest zero run
       ↓
Use lowercase hexadecimal
       ↓
RFC 5952-style address
```

### Example

```text
2001:0DB8:0000:0000:0000:0042:0000:0001
```

↓

```text
2001:DB8:0:0:0:42:0:1
```

↓

```text
2001:DB8::42:0:1
```

↓

```text
2001:db8::42:0:1
```

**Core idea:** IPv6 addresses are not actually being changed when you shorten them. You're only changing their **textual representation**. The two shortcuts—**removing leading zeros** and **compressing consecutive zero hextets with `::`**—make IPv6 manageable while preserving the exact same 128-bit address.
