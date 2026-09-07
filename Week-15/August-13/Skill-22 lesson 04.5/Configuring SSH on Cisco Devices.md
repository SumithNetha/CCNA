# Week 15 — August 13

# Lesson 04.5: Configuring SSH on Cisco Devices

This lesson is important because it takes you from **"I can remotely access a Cisco device"** to **"I can remotely access it securely."**

The central idea is:

> **Telnet provides remote access without encrypting the session. SSH provides encrypted remote access and uses device/user identity mechanisms to establish that secure session.**

---

# 1. First understand the problem: remote management

Imagine you have this network:

```text
                    Network
                       |
              +--------+--------+
              |                 |
           Switch              Router
              |
          Admin PC
```

You're sitting at the **Admin PC** and want to configure the router.

Without remote management, you'd physically connect:

```text
Admin PC
   |
Console cable
   |
Router
```

That's fine when you're standing beside the router.

But imagine Castle Rysen has:

```text
Central Office
     |
     +---- Fallout Shelter 1
     |         |
     |         +---- District Shop 1
     |         +---- District Shop 2
     |         +---- District Shop 3
     |
     +---- Fallout Shelter 2
               |
               +---- District Shops
```

You don't want to physically visit every router and switch.

So you need **remote management**.

Two classic choices are:

```text
Telnet
SSH
```

---

# 2. Telnet — why it is considered insecure

Telnet uses:

```text
TCP port 23
```

The major problem is that the Telnet session is transmitted in **clear text**.

For example, suppose you connect:

```text
Admin PC
   |
   | Telnet
   |
   v
Router
```

You enter:

```text
Username: admin
Password: Cisco123
```

Your commands might include:

```text
show running-config
show ip interface brief
configure terminal
```

Telnet doesn't provide encryption for this session.

So conceptually an attacker capable of capturing the traffic could see something resembling:

```text
admin
Cisco123
show ip interface brief
configure terminal
...
```

That's extremely dangerous.

### Important nuance from the lesson

The lesson makes an important distinction:

**Clear text does not mean everyone on the Internet automatically sees your password.**

An attacker still needs a way to capture/intercept the traffic.

For example:

```text
                 Attacker
                    |
                    | packet capture
                    |
Admin PC -------- Network -------- Router
```

Possible ways include access to the local network, traffic capture, or a man-in-the-middle position.

So the accurate statement is:

> **Telnet is insecure because the session isn't encrypted, making captured Telnet traffic readable.**

Not:

> "Anyone anywhere can instantly see your password."

That's an important real-world security distinction.

---

# 3. SSH — the secure alternative

SSH means:

> **Secure Shell**

SSH normally uses:

```text
TCP port 22
```

Instead of:

```text
Admin PC ---- Telnet ---- Router
             plaintext
```

you have:

```text
Admin PC ---- SSH ---- Router
              |
          encrypted
           session
```

If someone captures the packets, they shouldn't be able to simply read:

```text
username
password
commands
```

as plain text.

That's the fundamental advantage.

---

# 4. SSH is more than "encrypted Telnet"

This is one of the most important points from the lesson.

Don't think:

```text
Telnet
   ↓
add encryption
   ↓
SSH
```

SSH is a separate secure remote-management protocol that incorporates **encryption, authentication, identity, and trust mechanisms**.

The Cisco device needs cryptographic material to participate in the SSH connection.

That's why configuring SSH requires several steps.

---

# 5. Public key and private key — basic idea

SSH uses cryptographic keys.

At a high level:

```text
             SSH
              |
       +------+------+
       |             |
   Public Key    Private Key
```

They are mathematically related.

You don't need to become a cryptography expert for CCNA.

For this lesson, remember:

> **The Cisco device needs cryptographic keys so it can establish the secure SSH relationship.**

That's why you'll run:

```cisco
crypto key generate rsa
```

---

# 6. The complete SSH configuration

The lesson gives you this sequence:

```text
1. Create local username and secret
2. Configure hostname
3. Configure IP domain name
4. Generate RSA keys
5. Enable SSH version 2
6. Configure VTY lines to use local login
7. Allow SSH only
```

Let's build an example.

Suppose we have:

```text
                192.168.10.0/24

Admin PC
192.168.10.10
     |
     |
     |
G0/0
192.168.10.1
  Router R1
```

We want:

```text
Admin PC
   |
   | SSH
   | TCP 22
   |
   v
R1
```

---

# 7. Step 1 — Configure hostname

```cisco
Router(config)# hostname R1
```

Now:

```text
Router(config)#
```

becomes:

```text
R1(config)#
```

Why?

The Cisco device needs a proper identity.

The lesson specifically points out that hostname and domain information are involved in the SSH/RSA setup.

---

# 8. Step 2 — Configure the domain name

```cisco
R1(config)# ip domain-name castlerysen.local
```

Now the device has:

```text
Hostname:
R1

Domain:
castlerysen.local
```

Together, the device identity can be represented conceptually as:

```text
R1.castlerysen.local
```

The domain name is also required as part of the RSA key-generation process.

---

# 9. Step 3 — Create a local user

```cisco
R1(config)# username admin privilege 15 secret Cisco123
```

This creates a local account:

```text
Username: admin
Password: Cisco123
Privilege: 15
```

Think of the router as having its own small user database:

```text
R1 local database

+----------------------------+
| admin                      |
| privilege 15               |
| secret/password            |
+----------------------------+
```

---

# 10. Why `login local` matters

This is where the lesson makes an important distinction.

You might previously have configured:

```cisco
line vty 0 4
 password Cisco123
 login
```

That essentially means:

> "Use the password configured on these VTY lines."

But with SSH, the lesson wants us to use the actual local user database:

```cisco
line vty 0 4
 login local
```

Now Cisco asks for:

```text
Username:
Password:
```

Instead of simply:

```text
Password:
```

The authentication becomes:

```text
User enters username/password
             |
             v
       Local user database
             |
        Authentication
             |
             v
          Access
```

---

# 11. Step 4 — Generate RSA keys

Now:

```cisco
R1(config)# crypto key generate rsa
```

Cisco will ask for the key size.

For example:

```text
How many bits in the modulus [512]: 2048
```

You can enter:

```text
2048
```

The router generates its RSA cryptographic keys.

Conceptually:

```text
R1
 |
 +---- RSA public key
 |
 +---- RSA private key
```

These provide part of the cryptographic foundation required for SSH.

---

# 12. Why RSA keys are important

Before:

```text
Router
  |
  | No SSH cryptographic keys
  |
  X
```

After:

```text
Router
  |
  +---- RSA keys
           |
           +---- SSH can establish secure sessions
```

So when you see:

```cisco
crypto key generate rsa
```

think:

> **"I'm creating the cryptographic identity SSH needs."**

---

# 13. Step 5 — Force SSH version 2

Configure:

```cisco
R1(config)# ip ssh version 2
```

SSH version 2 is the modern version you want to use.

Conceptually:

```text
SSH
 |
 +---- Version 1 ❌
 |
 +---- Version 2 ✅
```

For your CCNA configuration, remember:

```cisco
ip ssh version 2
```

---

# 14. Step 6 — Configure VTY lines

Now we configure the virtual terminal lines:

```cisco
R1(config)# line vty 0 4
R1(config-line)# login local
```

### What are VTY lines?

VTY stands for:

> **Virtual Teletype / Virtual Terminal lines**

They're logical lines used for remote sessions.

Think:

```text
                R1
                 |
       +---------+---------+
       |         |         |
     VTY 0     VTY 1     VTY 2 ...
       |
       |
   Remote session
```

They're not physical interfaces like:

```text
GigabitEthernet0/0
GigabitEthernet0/1
```

They're logical access lines for remote terminal connections.

---

# 15. Step 7 — Allow SSH only

This is arguably the most important command in this lesson:

```cisco
R1(config-line)# transport input ssh
```

This tells the VTY lines:

> **Only accept SSH connections.**

Without this restriction, you might configure SSH successfully but accidentally leave Telnet available.

That's the security mistake the lesson is warning about.

---

# 16. The difference becomes very clear

Suppose you do:

```cisco
line vty 0 4
login local
```

but don't restrict the transport protocol.

You may have:

```text
             Router
                |
        +-------+-------+
        |               |
      SSH             Telnet
       ✅                ⚠️
```

You've enabled the secure option.

But you haven't necessarily removed the insecure option.

With:

```cisco
transport input ssh
```

you get:

```text
             Router
                |
              VTY
                |
               SSH
                |
               ✅
```

Telnet is no longer accepted through those VTY lines.

---

# 17. Complete example

Here's the complete basic configuration:

```cisco
enable
configure terminal

hostname R1

ip domain-name castlerysen.local

username admin privilege 15 secret Cisco123

crypto key generate rsa
```

Choose an appropriate RSA modulus when prompted, for example:

```text
2048
```

Then:

```cisco
ip ssh version 2

line vty 0 4
 login local
 transport input ssh
exit
```

Conceptually:

```text
                  R1
        +----------------------+
        |                      |
        | hostname R1          |
        | domain name          |
        | local user           |
        | RSA keys             |
        | SSH v2               |
        |                      |
        | VTY                  |
        |   login local        |
        |   SSH only           |
        +----------+-----------+
                   |
                   |
                  SSH
                   |
                   |
              Admin PC
```

---

# 18. What happens when the administrator connects?

Suppose the administrator's PC is:

```text
192.168.10.10
```

Router:

```text
192.168.10.1
```

The administrator runs something conceptually like:

```bash
ssh -l admin 192.168.10.1
```

The process is roughly:

```text
Admin PC
   |
   | SSH connection
   | TCP 22
   v
Router
   |
   | SSH negotiation
   |
   | Authentication
   |
   v
Username: admin
Password: ********
   |
   v
Authenticated
   |
   v
Router CLI
```

The administrator can now execute commands remotely:

```cisco
R1# show ip interface brief
R1# show running-config
R1# show ip route
```

---

# 19. What happens if you try Telnet?

After:

```cisco
transport input ssh
```

you attempt:

```text
Admin PC ---- Telnet ----> R1
```

The router rejects the connection.

Conceptually:

```text
Telnet
  |
  v
VTY
  |
  X
SSH only
```

But:

```text
SSH
 |
 v
VTY
 |
 v
login local
 |
 v
Username + password
 |
 v
Access
```

That's exactly what the lesson demonstrates: **test both protocols**, not just SSH.

---

# 20. Why testing matters

A common mistake is:

```text
Configure SSH
      ↓
Assume SSH works
      ↓
Done
```

A better administrator does:

```text
Configure
   ↓
Test SSH
   ↓
Test Telnet
   ↓
Verify Telnet is rejected
   ↓
Verify SSH succeeds
   ↓
Done
```

You're proving two things:

### Test 1

```text
SSH → SUCCESS
```

### Test 2

```text
Telnet → REJECTED
```

This confirms that you've both:

* enabled the secure management method
* removed the insecure management method

---

# 21. Real-world Castle Rysen example

This connects directly to your RFP.

The Castle Rysen requirements say that SSH/Telnet access to network devices must be restricted to:

* **Admin VLAN** at the Cafe
* **Management VLAN** at the Fallout Shelter 

Imagine a district coffee shop:

```text
                  District Shop
                       |
                  Cisco Switch
                       |
       +---------------+---------------+
       |               |               |
   Admin VLAN       Guest VLAN      Cameras
       |               |
   Admin PC         Customer PC
```

Suppose:

```text
Admin VLAN = VLAN 10
Guest VLAN = VLAN 20
```

The desired security model is:

```text
Admin PC
VLAN 10
   |
   | SSH
   v
Switch/Router
```

But:

```text
Guest PC
VLAN 20
   |
   | SSH ❌
   v
Switch/Router
```

The RFP explicitly requires administrative access to be limited to the appropriate management segment. 

So **`transport input ssh` alone is not the entire security design**.

It tells the VTY lines:

> "Use SSH, not Telnet."

You still need network security controls—such as ACLs—to determine **which source devices/networks are allowed to reach the management service**.

That's an important enterprise distinction:

```text
SSH configuration
        +
Source access restriction
        =
Secure management design
```

---

# 22. SSH vs Telnet — memorize this

| Feature                               | Telnet                   | SSH                   |
| ------------------------------------- | ------------------------ | --------------------- |
| Remote CLI                            | ✅                        | ✅                     |
| TCP port                              | 23                       | 22                    |
| Session encryption                    | ❌                        | ✅                     |
| Credentials protected in transit      | ❌                        | ✅                     |
| Suitable for modern device management | ❌                        | ✅                     |
| Cisco production management           | Avoid                    | Preferred             |
| VTY configuration                     | `transport input telnet` | `transport input ssh` |

The key isn't simply:

> **"Telnet bad, SSH good."**

The deeper understanding is:

> **Telnet provides remote access without encryption; SSH provides secure encrypted remote management and requires cryptographic/device identity configuration.**

---

# 23. One subtle point from the lesson: changing the hostname

The lesson warns about changing the hostname after generating RSA keys.

The important practical idea is:

```text
Hostname
   +
Domain name
   ↓
Device identity
   ↓
RSA/SSH setup
```

Therefore, establish the device identity **before generating the RSA keys**.

A good sequence is:

```text
hostname
    ↓
ip domain-name
    ↓
username
    ↓
crypto key generate rsa
    ↓
ip ssh version 2
    ↓
line vty
    ↓
login local
    ↓
transport input ssh
```

This avoids unnecessary rework.

---

# 24. What AAA has to do with this

The lesson briefly introduces **AAA**:

> Authentication, Authorization, and Accounting.

Right now we're using:

```text
Cisco Router
     |
     └── Local user database
```

For example:

```cisco
username admin privilege 15 secret ...
```

That's fine for a small lab.

But imagine Castle Rysen has:

```text
200 routers
500 switches
1000 network devices
```

Would you want to manually maintain:

```text
username admin ...
```

on every device?

Not ideal.

That's where centralized AAA becomes important.

Eventually you might have:

```text
                 Network Devices
              /       |        \
           Router   Switch    Firewall
              \       |        /
               \      |       /
                +-----+------+
                      |
                    AAA
                  Server
```

But that's beyond the scope of this specific SSH lesson.

For now:

> **Local authentication is enough to understand and configure SSH.**

---

# 25. The entire lesson in one mental model

Remember this:

```text
                   REMOTE MANAGEMENT
                         |
              +----------+----------+
              |                     |
            Telnet                 SSH
              |                     |
          TCP 23                 TCP 22
              |                     |
          Clear text             Encrypted
              |                     |
          ❌ Avoid                 ✅ Use
                                    |
                            Requires setup
                                    |
          +-------------------------+----------------------+
          |                         |                      |
      Local user               Domain name            RSA keys
          |                         |                      |
          +-------------------------+----------------------+
                                    |
                              SSH version 2
                                    |
                              VTY configuration
                                    |
                              login local
                                    |
                           transport input ssh
                                    |
                                    ▼
                            Secure management
```

---

# 🔑 Commands you should know

### Create identity

```cisco
hostname R1
ip domain-name example.com
```

### Create local credentials

```cisco
username admin privilege 15 secret PASSWORD
```

### Generate SSH cryptographic keys

```cisco
crypto key generate rsa
```

### Use SSHv2

```cisco
ip ssh version 2
```

### Configure remote login

```cisco
line vty 0 4
login local
```

### Permit SSH only

```cisco
transport input ssh
```

---

# 🧠 Most important exam/job concepts

If I gave you only **7 things** to remember from this lesson:

1. **Telnet = TCP 23**
2. **SSH = TCP 22**
3. Telnet sends the session in **clear text**.
4. SSH provides an **encrypted remote-management session**.
5. Cisco SSH setup requires **hostname + domain name + RSA keys**.
6. `login local` tells the VTY lines to authenticate against the **local username database**.
7. `transport input ssh` means **SSH only**, preventing Telnet access through those VTY lines.

And in the Castle Rysen design, remember one level deeper:

> **SSH protects the management session; VLANs/ACLs and the management architecture determine who is allowed to reach the management service in the first place.** The RFP specifically requires management access to be restricted to the Admin VLAN at cafes and Management VLAN at fallout shelters. 

This is the real-world networking lesson behind the configuration: **secure the protocol, authenticate the administrator, and restrict where management access can originate.**
