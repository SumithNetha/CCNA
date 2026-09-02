# Week 13 — July 30

## Skill 19 — Lesson 03: Understanding Identity and Password Attacks

---

# 1. Identity Attacks

A major portion of network security problems begins with **compromised identities and credentials**, rather than sophisticated attacks against network infrastructure.

An attacker who obtains a valid identity may not need to exploit a vulnerability at all.

### Core idea

> **Attackers don't always break in — sometimes they log in.**

Network environments contain many authentication points:

* Network devices
* Administrative portals
* Cloud dashboards
* Wi-Fi controllers
* POS systems
* SaaS applications
* VPNs

Every authentication point represents a potential target.

---

# 2. Authentication Factors

Passwords are an example of **something you know**.

MFA strengthens authentication by combining at least **two of three authentication factor categories**.

| Factor                 | Meaning                          | Examples                                                        |
| ---------------------- | -------------------------------- | --------------------------------------------------------------- |
| **Something you know** | Knowledge possessed by the user  | Password, PIN                                                   |
| **Something you have** | Physical/device-based possession | Authenticator app, hardware token, USB security key/certificate |
| **Something you are**  | Biological characteristic        | Fingerprint, facial recognition, retinal scan                   |

### MFA example

```text
Password
Something you know
       +
Authenticator app
Something you have
       ↓
      MFA
```

The important principle is:

> **A password by itself provides only one authentication factor.**

Adding another independent factor makes credential compromise significantly harder to exploit.

---

# 3. Why Passwords Matter

Passwords remain one of the most common authentication mechanisms.

The lesson recommends passwords that are:

* **Long**
* **Complex**
* **Unique**

It specifically discusses pushing password length toward **15 characters**.

### Why length matters

Attackers can use significant computing resources to attempt password recovery.

Conceptually:

```text
Short password
      ↓
Smaller search space
      ↓
Easier to crack

Long password
      ↓
Larger search space
      ↓
More difficult to crack
```

### Password rules

Avoid:

```text
Same password everywhere
        ↓
One compromise
        ↓
Multiple accounts compromised
```

Prefer:

```text
System A → Unique password
System B → Unique password
System C → Unique password
System D → Unique password
```

---

# 4. Password Hashes

Passwords are commonly stored as **hashes** rather than plaintext passwords.

A hash is a one-way representation of data.

Conceptually:

```text
Password
   ↓
Hashing function
   ↓
Password hash
```

An attacker who obtains a database containing password hashes can attempt to recover the original passwords **offline**.

This is important because the attacker no longer necessarily has to interact with the original authentication system for every guess.

### Offline password cracking

```text
Stolen password database
          ↓
Password hashes
          ↓
Attacker's computing resources
          ↓
Password guessing/cracking
          ↓
Potential recovered password
```

This is one reason password strength remains important even when passwords are stored as hashes.

---

# 5. Reconnaissance for Password Attacks

Before attacking an identity, an attacker may gather information about the target.

Possible information includes:

* Names
* Birthdays
* Family members
* Pets
* Schools
* Addresses
* Other publicly available information

The attacker can use this information to create more intelligent password guesses.

### Example

Suppose an attacker learns publicly that someone has:

```text
Pet → Rocky
School → ABC College
Birth year → 2002
```

The attacker may use combinations of such information when constructing guesses.

### Key principle

> **Information that appears harmless individually can become useful when combined.**

---

# 6. Dictionary Attacks

A **dictionary attack** uses automated tools to attempt passwords based on words and combinations of words.

Instead of blindly trying every possible character combination, the attacker uses likely words and patterns.

Conceptually:

```text
Dictionary
   ↓
Common words
   ↓
Word combinations
   ↓
Automated password attempts
```

Dictionary attacks are particularly effective against predictable passwords.

---

# 7. Brute-Force Attacks

A **brute-force attack** systematically attempts possible combinations until the correct password is found.

Conceptually:

```text
0000
0001
0002
0003
...
9999
```

For more complex passwords:

```text
a
b
c
...
aa
ab
ac
...
aaa
aab
...
```

The fundamental strategy is:

> **Try combinations until one works.**

The available computing power determines how quickly possibilities can be tested, especially in offline attacks.

---

# 8. Dictionary vs. Brute Force

| Characteristic                   | Dictionary Attack                          | Brute Force                          |
| -------------------------------- | ------------------------------------------ | ------------------------------------ |
| Strategy                         | Uses words/patterns                        | Tries combinations systematically    |
| Search space                     | More targeted                              | Potentially enormous                 |
| Depends on predictable passwords | Yes                                        | No                                   |
| Example                          | `coffee123`, `password`, word combinations | Every possible character combination |
| Main weakness exploited          | Human predictability                       | Limited password search space        |

---

# 9. Password Managers

When every system requires a unique, long password, remembering all of them becomes impractical.

A **password manager** provides a secure mechanism for storing credentials.

Instead of:

```text
Remember 30+ passwords
```

you use:

```text
Strong master password
        ↓
Password manager
        ↓
Stores unique credentials
```

### Important

The password manager itself becomes highly valuable.

Therefore, protect it with:

* Strong master password
* MFA

---

# 10. Social Engineering

Not every identity attack involves technically breaking a system.

**Social engineering** manipulates **people** into providing access or information.

Instead of attacking:

```text
Firewall
Switch
Server
Application
```

the attacker attacks:

```text
Human trust
```

### Core concept

> **Social engineering exploits people rather than primarily exploiting technology.**

---

# 11. Phishing

**Phishing** is a social-engineering technique where an attacker attempts to trick a victim into interacting with a malicious or fraudulent communication.

The message may attempt to make the victim:

* Reveal credentials
* Click a malicious link
* Open an attachment
* Transfer money
* Provide sensitive information

---

# 12. Spear Phishing

**Spear phishing** is a **targeted phishing attack** aimed at a specific person.

The attacker researches the target to make the message appear legitimate.

### Generic phishing

```text
Attacker
   ↓
Thousands of random users
```

### Spear phishing

```text
Attacker
   ↓
Research specific target
   ↓
Craft personalized message
   ↓
Specific victim
```

For example, a message might appear to come from:

* A manager
* An accountant
* A coworker
* Another trusted person

The more convincing the context, the greater the likelihood that the victim will trust it.

---

# 13. Whaling

**Whaling** is essentially spear phishing directed at **high-value targets**.

Targets may include:

* Executives
* Senior administrators
* People with financial authority
* Network administrators
* Other highly privileged users

### Why attackers target privileged users

Consider:

```text
Normal user account
       ↓
Limited access
```

versus:

```text
Administrator account
       ↓
Network
Cloud
Servers
Financial systems
Security controls
```

A compromised privileged identity can therefore produce significantly greater damage.

---

# 14. Smishing

**Smishing** is phishing conducted through **SMS/text messaging**.

Think:

```text
Phishing → Email
Smishing → SMS/text
```

Example concept:

```text
Fake message
     ↓
"Your account has a problem"
     ↓
Malicious link
     ↓
Victim interacts
```

---

# 15. Vishing

**Vishing** is phishing conducted through **voice communication**, such as a phone call.

Think:

```text
Phishing  → Email
Smishing  → SMS
Vishing   → Voice
```

The attacker may impersonate a trusted organization or individual.

---

# 16. Tailgating

**Tailgating** is a physical security attack where an unauthorized person follows an authorized person into a restricted area.

Conceptually:

```text
Authorized employee
        ↓
Opens secured door
        ↓
Attacker follows
        ↓
Unauthorized physical access
```

This demonstrates that identity security isn't purely digital.

---

# 17. Digital + Physical Identity Security

Identity attacks can occur across multiple layers:

```text
                 IDENTITY SECURITY
                       │
          ┌────────────┴────────────┐
          │                         │
       Digital                   Physical
          │                         │
    Password attacks            Tailgating
    Phishing                     Unauthorized entry
    MFA bypass                   Physical access
    Credential theft
```

If an attacker gets physical access to a:

* Wiring closet
* Server room
* Network equipment
* Office

the security problem becomes much larger.

---

# 18. Defending Against Identity Attacks

The lesson recommends a combination of technical controls, user awareness, and physical controls. 

## Password security

1. Use **long, unique passwords**
2. Use a **password manager**
3. Enable **MFA**
4. Never reuse passwords across systems
5. Protect administrative accounts more strongly

---

## Social-engineering defenses

Organizations should use:

* Security awareness training
* Simulated phishing campaigns
* User education
* Verification procedures
* A habit of slowing down before clicking

The goal isn't simply:

> "Don't click bad links."

It is to develop the habit of **questioning unexpected requests**.

---

# 19. Physical Security Controls

Physical security can help prevent attacks such as tailgating and unauthorized access.

Examples from the lesson include:

* Badge systems
* Mantraps
* Camera coverage
* Entry logging

### Example

```text
Employee badge
      ↓
Access control
      ↓
Secured door
      ↓
Entry logged
```

A **mantrap** can provide additional control by ensuring that a person passes through controlled entry points rather than simply following someone through an open door.

---

# 20. Administrative Accounts Need Extra Protection

Administrative accounts are especially valuable targets.

Compare:

```text
Compromised standard account
        ↓
Limited permissions
```

with:

```text
Compromised admin account
        ↓
Elevated privileges
        ↓
Greater potential impact
```

Therefore:

> **The more privileged the account, the stronger the security controls should be.**

This becomes particularly important for network administrators because administrator credentials may provide access to routers, switches, firewalls, controllers, servers, and cloud infrastructure.

---

# 21. Identity Attack Chain

A realistic attack can combine several techniques:

```text
Reconnaissance
      ↓
Collect information about employee
      ↓
Spear phishing
      ↓
Victim reveals credentials
      ↓
Attacker obtains identity
      ↓
Login
      ↓
Privilege/access abuse
      ↓
Network compromise
```

Another possibility:

```text
Password reuse
      ↓
Credential stolen from one service
      ↓
Same password tried elsewhere
      ↓
Multiple accounts compromised
```

This is why **unique passwords + MFA** are so important.

---

# 22. AAA — The Next Cisco Concept

This lesson leads directly into **Cisco AAA**.

AAA stands for:

| AAA component      | Meaning                | Question answered               |
| ------------------ | ---------------------- | ------------------------------- |
| **Authentication** | Proves identity        | **Who are you?**                |
| **Authorization**  | Determines permissions | **What are you allowed to do?** |
| **Accounting**     | Tracks activity        | **What did you do?**            |

### Simple model

```text
User
 ↓
AUTHENTICATION
"Who are you?"
 ↓
AUTHORIZATION
"What can you access?"
 ↓
ACCOUNTING
"What did you do?"
```

This is the bridge between today's identity-security concepts and the next Cisco networking lesson.

---

# 23. Network Administrator Example

Suppose an administrator wants to access a Cisco router.

Without proper identity controls:

```text
Username/password
      ↓
Router access
```

With stronger identity architecture:

```text
Administrator
      ↓
Authentication
      ↓
MFA / credentials
      ↓
Authorization
      ↓
Determine privilege
      ↓
Router access
      ↓
Accounting
      ↓
Record activity
```

This is the basic security philosophy behind AAA.

---

# 24. Quick Comparison

| Attack             | What the attacker does                          | Key word   |
| ------------------ | ----------------------------------------------- | ---------- |
| **Dictionary**     | Tries likely words/passwords                    | Words      |
| **Brute force**    | Tries combinations systematically               | Everything |
| **Phishing**       | Tricks victims through fraudulent communication | Deception  |
| **Spear phishing** | Targets a specific individual                   | Targeted   |
| **Whaling**        | Targets high-value individuals                  | Executives |
| **Smishing**       | Phishing through SMS                            | Text       |
| **Vishing**        | Phishing through voice                          | Voice      |
| **Tailgating**     | Follows authorized person into secured area     | Physical   |
| **Reconnaissance** | Collects information about target               | Research   |

---

# 25. Exam Memory Tricks

### Authentication factors

```text
KNOW
     Password / PIN

HAVE
     Phone / Token / Security Key

ARE
     Fingerprint / Face / Retina
```

**MFA = at least 2 different factor categories.**

---

### Password attacks

```text
Dictionary → Words
Brute Force → All combinations
```

---

### Phishing family

```text
Phishing   → General fraudulent communication
Spear      → Specific target
Whaling    → High-value target
Smishing   → SMS
Vishing    → Voice
```

---

### AAA

```text
A — Authentication → WHO?
A — Authorization  → WHAT can you do?
A — Accounting     → WHAT did you do?
```

---

# 26. Final Takeaway

The central lesson is that **identity itself is a security boundary**.

An attacker doesn't necessarily need to exploit a router or firewall if they can obtain a legitimate identity.

The defensive model is:

```text
             IDENTITY SECURITY
                    │
        ┌───────────┼───────────┐
        ↓           ↓           ↓
     Strong       MFA       Awareness
    Passwords                 Training
        │           │           │
        └───────────┼───────────┘
                    ↓
             AAA / Access Control
                    ↓
             Reduced Exposure
```

And the progression of today's Skill 19 lessons is now:

```text
Lesson 01
Vulnerabilities
     ↓
Exploits
     ↓
Threats

Lesson 02
Threat Categories
     ↓
DoS / DDoS / Spoofing / MITM
Recon / Malware

Lesson 03
Identity & Password Attacks
     ↓
Password Attacks
     ↓
Social Engineering
     ↓
Physical Identity Attacks
     ↓
              AAA
```

**The key sentence to remember:**

> **Attackers don't always break into systems — if they can obtain a valid identity, they can simply log in.**

Your **next lesson (July 31)** is **Skill 19 Lesson 04 — Understanding the Place of Cisco AAA**, where these identity concepts become actual Cisco access-control architecture. 
