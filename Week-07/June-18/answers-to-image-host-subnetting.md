# Subnetting Practice

## Objective

Strengthen subnetting skills by solving problems independently. The goal is to become comfortable designing subnets based on **host requirements** without relying on worked examples.

---

# Why Practice is Important

Subnetting is a practical networking skill.

Watching examples helps you understand the concept, but solving problems yourself develops real problem-solving ability.

> **Recognition is not mastery. Practice builds confidence.**

---

# Standard Subnetting Process

Use the same process for every host-based subnetting problem.

```
Read Requirement
        ↓
Determine Required Hosts
        ↓
Find Host Bits (2ⁿ - 2 ≥ Hosts)
        ↓
Calculate Prefix Length
        ↓
Determine Subnet Mask
        ↓
Find Block Size (Increment)
        ↓
List Network Addresses
        ↓
Find Host Range & Broadcast Address
```

---

# Formula

```
2^HostBits - 2 ≥ Required Hosts
```

Subtract **2** because every subnet reserves:

- Network Address
- Broadcast Address

---

# Practice Problem 1

### Network

```
200.1.1.0/24
```

### Requirement

```
20 Hosts per Subnet
```

### Solution

| Item | Value |
|------|------|
| Host Bits | 5 |
| Prefix | /27 |
| Subnet Mask | 255.255.255.224 |
| Increment | 32 |
| Usable Hosts | 30 |

### Subnets

| Network | Host Range | Broadcast |
|---------|------------|-----------|
| 200.1.1.0 | .1 - .30 | .31 |
| 200.1.1.32 | .33 - .62 | .63 |
| 200.1.1.64 | .65 - .94 | .95 |
| 200.1.1.96 | .97 - .126 | .127 |
| 200.1.1.128 | .129 - .158 | .159 |
| 200.1.1.160 | .161 - .190 | .191 |
| 200.1.1.192 | .193 - .222 | .223 |
| 200.1.1.224 | .225 - .254 | .255 |

---

# Practice Problem 2

### Network

```
199.9.10.0/24
```

### Requirement

```
12 Hosts per Subnet
```

### Solution

| Item | Value |
|------|------|
| Host Bits | 4 |
| Prefix | /28 |
| Subnet Mask | 255.255.255.240 |
| Increment | 16 |
| Usable Hosts | 14 |

### First Few Subnets

| Network | Broadcast |
|---------|-----------|
| 199.9.10.0 | .15 |
| 199.9.10.16 | .31 |
| 199.9.10.32 | .47 |
| 199.9.10.48 | .63 |
| ... | ... |
| 199.9.10.240 | .255 |

---

# Practice Problem 3

### Network

```
172.50.0.0/16
```

### Requirement

```
1000 Hosts per Subnet
```

### Solution

| Item | Value |
|------|------|
| Host Bits | 10 |
| Prefix | /22 |
| Subnet Mask | 255.255.252.0 |
| Increment | 4 (Third Octet) |
| Usable Hosts | 1022 |

### First Few Subnets

| Network | Broadcast |
|---------|-----------|
| 172.50.0.0 | 172.50.3.255 |
| 172.50.4.0 | 172.50.7.255 |
| 172.50.8.0 | 172.50.11.255 |
| 172.50.12.0 | 172.50.15.255 |

---

# Practice Problem 4

### Network

```
12.0.0.0/8
```

### Requirement

```
100 Hosts per Subnet
```

### Solution

| Item | Value |
|------|------|
| Host Bits | 7 |
| Prefix | /25 |
| Subnet Mask | 255.255.255.128 |
| Increment | 128 (Fourth Octet) |
| Usable Hosts | 126 |

### First Few Subnets

| Network | Broadcast |
|---------|-----------|
| 12.0.0.0 | 12.0.0.127 |
| 12.0.0.128 | 12.0.0.255 |
| 12.0.1.0 | 12.0.1.127 |
| 12.0.1.128 | 12.0.1.255 |

---

# Practice Workflow

For every subnetting question:

1. Read the requirement carefully.
2. Solve it completely on your own.
3. Verify your answer.
4. Identify mistakes.
5. Repeat until the process becomes automatic.

```
Attempt First
      ↓
Check Answer
      ↓
Correct Mistakes
      ↓
Repeat
```

---

# Common Mistakes

- Using subnet-count method when the question asks for hosts.
- Forgetting to subtract 2 for Network and Broadcast addresses.
- Choosing too few host bits.
- Calculating the wrong increment.
- Looking at the answer before solving the problem.

---

# Best Practices

- Solve problems on paper.
- Write every calculation.
- Follow the same sequence every time.
- Accuracy is more important than speed.
- Speed develops naturally through repetition.

---

# Real-World Usage

Host-based subnetting is commonly used when designing networks for:

- Employee PCs
- Servers
- Printers
- IP Phones
- Security Cameras
- Wireless Clients
- POS Terminals
- IoT Devices

Always allocate extra IP addresses for future expansion.

---

# Key Takeaways

- Practice is essential for mastering subnetting.
- Use the same step-by-step process for every problem.
- Host-based subnetting starts with the required number of devices.
- Accuracy comes before speed.
- Confidence comes from repetition, not memorization.

---

# Memory Tips

```
Recognition ≠ Mastery
```

```
Watch
↓

Practice
↓

Repeat
↓

Master
```

```
Attempt First

↓

Verify Second
```

```
Consistency

↓

Accuracy

↓

Speed
```