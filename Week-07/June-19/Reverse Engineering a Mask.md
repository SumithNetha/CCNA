# Reverse Engineering a Mask

## Overview

Reverse Engineering a Mask is the process of analyzing an existing IP address and subnet mask to determine the characteristics of the subnet it belongs to. Instead of designing a subnet from host or network requirements, you work backward from an already configured network to understand its structure.

This is one of the most common subnetting tasks performed by network administrators because enterprise networks are usually already deployed. Troubleshooting requires understanding existing IP configurations rather than creating new ones.

---

# Why Reverse Engineering is Important

In real-world networking, engineers rarely build a network from scratch. Instead, they inherit existing networks and troubleshoot connectivity issues.

Common troubleshooting tasks include:

- Determining the subnet of a device
- Verifying the default gateway
- Finding the network address
- Finding the broadcast address
- Checking valid host ranges
- Identifying incorrect IP configurations
- Verifying whether multiple devices belong to the same subnet

Reverse engineering enables administrators to quickly identify addressing problems that prevent communication.

---

# Designing vs Reverse Engineering

## Designing a Subnet

Start with:

- Number of hosts
- Number of networks

Then calculate:

1. Borrow bits
2. Subnet mask
3. Prefix length
4. Increment
5. Network ranges
6. Host addresses

---

## Reverse Engineering

Start with:

- Existing IP address
- Existing subnet mask

Then determine:

1. Prefix length
2. Interesting octet
3. Increment
4. Network range
5. Network address
6. Broadcast address
7. Valid host range
8. Default gateway validity

---

# Reverse Engineering Process

## Step 1: Identify the Subnet Mask

Example:

```
IP Address:
192.168.5.22

Subnet Mask:
255.255.255.240
```

The subnet mask determines which bits represent the network and which represent the hosts.

---

## Step 2: Find the Interesting Octet

The interesting octet is the first octet that is not 255.

```
255.255.255.240
              ↑
```

Only the last octet is used for subnet calculations.

---

## Step 3: Determine the Prefix Length

```
255.255.255.240

↓

11111111.11111111.11111111.11110000
```

There are:

- 28 network bits
- 4 host bits

Therefore:

```
Prefix Length = /28
```

---

## Step 4: Find the Increment

### Method 1

```
256 - 240 = 16
```

Increment = **16**

### Method 2

Binary:

```
11110000
```

The first zero bit has a decimal value of **16**.

---

# Step 5: Build the Network Ranges

Using an increment of 16:

```
0
16
32
48
64
80
96
112
128
144
160
176
192
208
224
240
256
```

Each subnet contains **16 addresses**.

---

# Step 6: Find the Correct Range

Host:

```
192.168.5.22
```

Last octet:

```
22
```

Subnet ranges:

```
0-15

16-31  ← Host belongs here

32-47

48-63
```

---

# Step 7: Determine the Network Address

The first address of the subnet is always the network address.

```
Network Address

192.168.5.16
```

The network address identifies the subnet and cannot be assigned to a host.

---

# Step 8: Determine the Broadcast Address

The last address of the subnet is the broadcast address.

```
Broadcast Address

192.168.5.31
```

Broadcast packets are delivered to every host within the subnet.

---

# Step 9: Determine the Valid Host Range

```
Network Address

192.168.5.16

↓

First Host

192.168.5.17

↓

Last Host

192.168.5.30

↓

Broadcast

192.168.5.31
```

Valid hosts:

```
192.168.5.17
through
192.168.5.30
```

---

# Complete Example

Given:

```
IP Address:
192.168.5.22

Subnet Mask:
255.255.255.240
```

Results:

| Property | Value |
|----------|-------|
| Prefix Length | /28 |
| Increment | 16 |
| Network Address | 192.168.5.16 |
| Broadcast Address | 192.168.5.31 |
| First Host | 192.168.5.17 |
| Last Host | 192.168.5.30 |
| Total Addresses | 16 |
| Usable Hosts | 14 |

---

# Comparing Multiple Devices

Example:

| Device | IP Address | Mask |
|---------|------------|------|
| PC | 192.168.1.10 | /28 |
| Server 1 | 192.168.1.17 | /28 |
| Server 2 | 192.168.1.19 | /28 |
| Router | 192.168.1.33 | /28 |

Subnet analysis:

```
PC

10

↓

0-15
```

```
Server 1

17

↓

16-31
```

```
Server 2

19

↓

16-31
```

```
Router

33

↓

32-47
```

Although every IP address is valid, the devices belong to three different logical networks.

Communication will fail because they are not members of the same subnet.

---

# Default Gateway Validation

Example:

```
PC

192.168.1.10/28

Gateway

192.168.1.33
```

PC subnet:

```
0-15
```

Gateway subnet:

```
32-47
```

The gateway is outside the PC's subnet.

Result:

- The PC cannot reach the gateway.
- Communication with remote networks fails.

**Rule:**

The default gateway must always be a valid host address within the same subnet as the device.

---

# Increment Reference Table

| Subnet Mask | Prefix | Increment |
|-------------|---------|-----------|
|255.255.255.0|/24|1|
|255.255.255.128|/25|128|
|255.255.255.192|/26|64|
|255.255.255.224|/27|32|
|255.255.255.240|/28|16|
|255.255.255.248|/29|8|
|255.255.255.252|/30|4|
|255.255.255.254|/31|2|

---

# Troubleshooting Workflow

1. Identify the subnet mask.
2. Find the interesting octet.
3. Calculate the increment.
4. Build subnet ranges.
5. Locate the host's subnet.
6. Determine the network address.
7. Determine the broadcast address.
8. Find the valid host range.
9. Verify the host IP is valid.
10. Verify the default gateway belongs to the same subnet.
11. Compare devices to ensure they are in the same logical network.

---

# Key Takeaways

- Reverse engineering analyzes an existing subnet rather than designing one.
- The subnet mask is the starting point because it determines the subnet increment.
- The increment defines subnet boundaries.
- The network address is the first address of a subnet.
- The broadcast address is the last address of a subnet.
- Valid hosts exist between the network and broadcast addresses.
- Two devices can have valid IP addresses but still fail to communicate if they belong to different subnets.
- The default gateway must always be within the same subnet as the host.
- Reverse engineering subnet masks is a critical CCNA troubleshooting skill used extensively in real-world networking.