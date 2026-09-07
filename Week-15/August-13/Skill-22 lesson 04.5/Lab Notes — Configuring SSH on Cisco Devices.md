# Lab Notes — Configuring SSH on Cisco Devices

### Skill 22 — Lesson 04.5 | S22-L04.5

## Objective

Configure a Cisco router for **secure remote management using SSH** instead of Telnet.

The lab configuration establishes:

* Local username authentication
* Hostname and domain identity
* RSA keys
* SSH version 2
* VTY authentication using the local database
* SSH-only remote access
* Source-IP restrictions using a VTY access-class

---

## Topology / Existing Device

```text
Device: cafe01-RT01
Hostname: cafe01-RT01
```

Relevant networks:

```text
ADMIN VLAN 10
Network: 10.0.18.0/27
Gateway: 10.0.18.1

PATRON VLAN 20
Network: 10.0.18.32/27
Gateway: 10.0.18.33
```

The router already had an `ADMIN_LIMIT` standard ACL applied to the VTY lines:

```cisco
ip access-list standard ADMIN_LIMIT
 permit 10.0.18.0 0.0.0.31
 permit 10.0.16.0 0.0.0.127
```

```cisco
line vty 0 4
 access-class ADMIN_LIMIT in
```

---

# 1. Create a Local User

Enter global configuration mode:

```cisco
enable
configure terminal
```

Create the local user:

```cisco
username luffy secret cisco
```

Verify:

```cisco
do show running-config | include username
```

Expected:

```text
username luffy secret 5 <hashed-secret>
```

### Purpose

Creates a local account that SSH will use for authentication.

```text
SSH login
    |
    v
username: luffy
password: cisco
    |
    v
Local Cisco user database
```

---

# 2. Configure the Domain Name

```cisco
ip domain-name castlerysen.local
```

The router already has the hostname:

```cisco
hostname cafe01-RT01
```

Therefore the device identity is conceptually:

```text
cafe01-RT01.castlerysen.local
```

### Why?

The hostname and domain name are prerequisites for generating the RSA keys used by SSH.

---

# 3. Generate RSA Keys

Run:

```cisco
crypto key generate rsa
```

When prompted:

```text
How many bits in the modulus [512]:
```

Enter:

```text
2048
```

Expected:

```text
% Generating 2048 bit RSA keys, keys will be non-exportable...[OK]
```

Cisco may display:

```text
%SSH-5-ENABLED: SSH 1.99 has been enabled
```

This indicates that SSH has become enabled after the RSA keys were generated.

---

# 4. Configure SSH Version 2

```cisco
ip ssh version 2
```

Verify:

```cisco
do show ip ssh
```

Look for:

```text
SSH Enabled - version 2.0
```

### Why?

SSHv2 is the version we want to use for secure remote management.

---

# 5. Configure VTY Lines

Enter VTY configuration mode:

```cisco
line vty 0 4
```

The VTY lines provide the logical terminal sessions used for remote access.

---

# 6. Use Local User Authentication

Configure:

```cisco
login local
```

This tells the VTY lines to authenticate users against the router's local username database.

Because we previously created:

```cisco
username luffy secret cisco
```

the SSH login will use:

```text
Username: luffy
Password: cisco
```

### Difference

Without `login local`:

```cisco
login
password cisco123
```

uses the VTY line password.

With:

```cisco
login local
```

Cisco checks the local user database.

---

# 7. Allow SSH Only

Configure:

```cisco
transport input ssh
```

This is critical.

It tells the VTY lines to accept **SSH connections only**.

```text
SSH       → ✅ Allowed
Telnet    → ❌ Not allowed
```

Without this command, simply configuring SSH does **not necessarily mean Telnet has been removed**.

---

# 8. Final VTY Configuration

Your VTY configuration should contain:

```cisco
line vty 0 4
 access-class ADMIN_LIMIT in
 login local
 transport input ssh
```

This gives you multiple layers of protection:

```text
Remote connection
       |
       v
   VTY lines
       |
       +---- access-class ADMIN_LIMIT
       |          |
       |          +-- Restrict source IPs
       |
       +---- transport input ssh
       |          |
       |          +-- SSH only
       |
       +---- login local
                  |
                  +-- Local username/password
```

---

# 9. Verify the Configuration

### Check SSH

```cisco
do show ip ssh
```

Expected:

```text
SSH Enabled - version 2.0
```

---

### Check RSA keys

```cisco
do show crypto key mypubkey rsa
```

This verifies that RSA keys exist.

---

### Check local user

```cisco
do show running-config | include username
```

Expected:

```text
username luffy secret 5 ...
```

---

### Check VTY configuration

```cisco
do show running-config | section line vty
```

Expected:

```cisco
line vty 0 4
 access-class ADMIN_LIMIT in
 login local
 transport input ssh
```

---

### Check the ACL

```cisco
do show access-lists ADMIN_LIMIT
```

Expected permitted networks:

```text
10.0.18.0 0.0.0.31
10.0.16.0 0.0.0.127
```

---

# 10. Test SSH

From an allowed Admin VLAN device, connect to the router using SSH.

Conceptually:

```text
Admin PC
10.0.18.x
    |
    | SSH / TCP 22
    v
cafe01-RT01
10.0.18.1
```

Login:

```text
Username: luffy
Password: cisco
```

Successful authentication should give you the router CLI.

---

# 11. Test Telnet

Now test Telnet from an appropriate device.

The connection should **fail** because of:

```cisco
transport input ssh
```

Expected behavior:

```text
SSH       → ✅ Successful
Telnet    → ❌ Rejected
```

This is an important validation step.

You're not simply checking that SSH works—you are checking that **the insecure remote-access method has been removed**.

---

# 12. Configuration Summary

The SSH-related configuration for `cafe01-RT01` is:

```cisco
hostname cafe01-RT01

ip domain-name castlerysen.local

username luffy secret cisco

crypto key generate rsa
! Choose 2048 bits

ip ssh version 2

line vty 0 4
 access-class ADMIN_LIMIT in
 login local
 transport input ssh
```

---

# 13. Command → Purpose

| Command                            | Purpose                                  |
| ---------------------------------- | ---------------------------------------- |
| `hostname cafe01-RT01`             | Establish device hostname                |
| `ip domain-name castlerysen.local` | Configure domain identity                |
| `username luffy secret cisco`      | Create local authentication account      |
| `crypto key generate rsa`          | Generate RSA keys for SSH                |
| `ip ssh version 2`                 | Use SSHv2                                |
| `line vty 0 4`                     | Configure remote terminal lines          |
| `login local`                      | Authenticate against local user database |
| `transport input ssh`              | Permit SSH and reject Telnet             |
| `access-class ADMIN_LIMIT in`      | Restrict which source IPs can access VTY |

---

# 14. Important Troubleshooting

If SSH doesn't work, check in this order:

```text
1. Does the router have an IP address?
        ↓
2. Can the PC ping the router?
        ↓
3. Is the domain name configured?
        ↓
4. Do RSA keys exist?
        ↓
5. Is SSH version 2 configured?
        ↓
6. Is a local username configured?
        ↓
7. Is "login local" configured?
        ↓
8. Is "transport input ssh" configured?
        ↓
9. Does ADMIN_LIMIT permit the client's source IP?
```

Useful commands:

```cisco
show ip interface brief
show ip ssh
show crypto key mypubkey rsa
show running-config | include username
show running-config | section line vty
show access-lists ADMIN_LIMIT
```

---

# 🔑 Lab Takeaway

The complete security chain is:

```text
                 SSH
                  |
             TCP port 22
                  |
          +-------+-------+
          |               |
    Source restriction  Authentication
     access-class         login local
          |               |
          |          luffy / password
          |               |
          +-------+-------+
                  |
             RSA keys
                  |
              SSHv2
                  |
             Secure CLI
```

**Most important commands from this lab:**

```cisco
username luffy secret cisco
ip domain-name castlerysen.local
crypto key generate rsa
ip ssh version 2
line vty 0 4
login local
transport input ssh
```

And remember the key distinction:

> **`transport input ssh` controls *how* you can connect; `access-class` controls *which source addresses* can connect; `login local` controls *who* can authenticate.**
