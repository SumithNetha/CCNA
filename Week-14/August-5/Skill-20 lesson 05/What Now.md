# Skill 20 — Lesson 05: What Now?

This section is the **wrap-up of Layer 2 security**, but its real purpose is to change how you approach network security.

The lesson's central message is:

> **Stop treating Port Security, DHCP Snooping, and DAI as CCNA commands. Turn them into a repeatable security standard.**

---

## 1. From Commands → Policy

Knowing:

```text
switchport port-security
ip dhcp snooping
ip arp inspection vlan ...
```

is not enough.

The professional question is:

> **When I deploy a switch, what is my standard security baseline?**

That means defining:

```text
Port role
   ↓
Security policy
   ↓
Configuration
   ↓
Verification
   ↓
Documentation
```

Instead of configuring each switch differently depending on what you remember that day.

---

# 2. Build an Access-Switch Security Baseline

The lesson recommends creating a simple **one-page access-switch security baseline**.

For Castle Rysen, yours could contain:

### Port Security

```text
• Apply to appropriate end-device access ports
• Maximum 1 MAC address where appropriate
• Sticky MAC for appropriate fixed administrative devices
• Violation action: shutdown
• Do not blindly apply to uplinks/infrastructure ports
```

### DHCP Snooping

```text
• Enable on required VLANs
• Infrastructure/DHCP-facing ports → trusted
• Client-facing ports → untrusted
• Apply appropriate DHCP rate limiting
```

### DAI

```text
• Enable on required VLANs
• Infrastructure ports → trusted where appropriate
• Endpoint ports → untrusted
• Validate ARP traffic
• Apply appropriate ARP rate limiting
```

The exact policy should be based on the network design rather than blindly copied from one environment to another.

---

# 3. Your Castle Rysen Baseline

Your lab already gives you a concrete starting point.

From your deployment:

```text
                Castle Rysen Access Switch
                           |
          +----------------+----------------+
          |                |                |
          ↓                ↓                ↓
    Port Security    DHCP Snooping         DAI
          |                |                |
     1 MAC limit        VLAN 1,10,20     VLAN 1,10,20
     Shutdown           Trusted ports    Trusted ports
     Sticky where       5 pps clients    15 pps ARP
     appropriate
```

The Castle Rysen RFP explicitly requires these Layer 2 security controls as part of the network security implementation. 

---

# 4. Turn the Configuration Into a Template

The next step is to stop thinking:

> "What command do I type?"

and start thinking:

> **"What is my deployment sequence?"**

A template should have three sections.

### A. Configuration

```text
Global security configuration
        ↓
VLAN security
        ↓
Access-port security
        ↓
Trusted infrastructure ports
```

### B. Verification

```text
show port-security
show ip dhcp snooping
show ip arp inspection
show ip arp inspection interfaces
```

### C. Exceptions

Document cases where the standard **should not** be applied.

For example:

```text
Uplink
   ↓
Don't treat like ordinary endpoint port

Access Point
   ↓
May represent multiple client MAC addresses

Special infrastructure
   ↓
Review individually
```

This exception list is just as important as the commands.

---

# 5. Why Repeatability Matters

Imagine three switches:

```text
Switch A → Port Security + DHCP Snooping + DAI

Switch B → Port Security only

Switch C → Nothing
```

Even if all three switches are technically functioning, you've created an inconsistent security posture.

An attacker doesn't care that your network has excellent security documentation on Switch A.

They will look for:

```text
Switch C
   ↓
Security gap
   ↓
Potential attack path
```

Therefore:

> **Consistency itself becomes a security control.**

---

# 6. Security Should Be Built In

The lesson wants you to change your default workflow.

### Old mindset

```text
Build network
     ↓
Make it work
     ↓
Remember security later
```

### Better mindset

```text
Requirements
     ↓
Design
     ↓
Security policy
     ↓
Build
     ↓
Verify
     ↓
Document
```

Security becomes part of the network's architecture rather than an afterthought.

This fits the Castle Rysen RFP very well because security is explicitly part of the implementation requirements, alongside VLANs, routing, STP, NAT, DHCP, DNS, NTP, and other services. 

---

# 7. Don't Confuse Best Practice With Blind Configuration

One subtle point from this lesson is particularly important.

A security feature can be technically good but operationally wrong if you apply it to the wrong interface.

For example:

```text
Port Security
      ↓
Great security feature
      ↓
Apply blindly to uplink
      ↓
Multiple legitimate MACs
      ↓
Potential outage
```

The correct question is always:

> **What is connected here, and what behavior should I expect from it?**

That's why your earlier troubleshooting was valuable.

You discovered that Port Security couldn't be enabled while the interface was dynamically configured:

```text
Command rejected:
FastEthernet0/3 is a dynamic port.
```

You then changed the port to access mode before enabling Port Security.

That is much more useful than simply memorizing the command.

---

# 8. The Three Things You Should Do Next

The lesson gives you a very practical progression.

## Step 1 — Write the policy

Plain English:

```text
What am I protecting?
What ports are endpoints?
What ports are infrastructure?
What is trusted?
What is untrusted?
What happens during a violation?
```

You've already started doing this with your **Castle Rysen Café: Layer 2 Security Policy**.

---

## Step 2 — Build the CLI template

Translate the policy into Cisco IOS configuration.

Structure it logically:

```text
1. VLAN/security prerequisites
2. Port Security
3. DHCP Snooping
4. Trusted DHCP interfaces
5. DHCP rate limits
6. DAI
7. Trusted DAI interfaces
8. ARP rate limits
```

Don't just keep a random collection of commands.

---

## Step 3 — Lab it repeatedly

The goal is:

```text
Configure
   ↓
Verify
   ↓
Break it intentionally
   ↓
Troubleshoot
   ↓
Fix it
   ↓
Repeat
```

Eventually you should be able to look at a switch and reason:

> "This is an endpoint port, so it gets X. This is an uplink, so it gets Y. This interface is trusted because Z."

That's the actual skill being developed.

---

# 9. Your Layer 2 Security Troubleshooting Mindset

When something breaks, don't immediately remove all security features.

Work backwards.

### Client can't get an IP

Check:

```text
DHCP Snooping
      ↓
Is snooping enabled?
      ↓
Correct VLAN?
      ↓
Correct trusted interface?
      ↓
Rate limit?
      ↓
DHCP binding?
```

### ARP/connectivity problem

Check:

```text
DAI
 ↓
Correct VLAN?
 ↓
Interface trusted/untrusted correctly?
 ↓
DHCP Snooping binding?
 ↓
ARP validation?
 ↓
DAI statistics?
```

### Port suddenly shuts down

Check:

```text
Port Security
      ↓
show port-security
      ↓
Security violation?
      ↓
Which MAC appeared?
      ↓
Was the device actually authorized?
```

This is the beginning of a proper troubleshooting methodology.

---

# 10. The Consultant Mindset

The lesson makes an important distinction:

### Command-oriented mindset

> "I know Cisco commands."

### Network-engineering mindset

> "I know how to design, implement, verify, troubleshoot, and standardize network configurations."

### Consultant mindset

> "I can take a business requirement and turn it into a repeatable technical solution."

Castle Rysen's RFP is deliberately giving you that kind of exercise.

The RFP doesn't say:

> "Put this exact command on Fa0/5."

It says the network needs to be secure and requires Layer 2 security features. 

**You** have to turn that requirement into:

```text
Business requirement
       ↓
Technical requirement
       ↓
Security policy
       ↓
Interface design
       ↓
Cisco configuration
       ↓
Verification
       ↓
Documentation
```

That's a much more realistic networking workflow.

---

# 11. The Bigger Picture

This lesson is really connecting everything you've learned so far.

You've already worked through:

```text
VLANs
 ↓
Segmentation

STP
 ↓
Loop prevention

EtherChannel
 ↓
Redundancy + bandwidth

OSPF
 ↓
Dynamic routing

HSRP
 ↓
Gateway redundancy

IPv6
 ↓
Addressing

ACLs
 ↓
Traffic control

Layer 2 Security
 ↓
Endpoint/infrastructure protection
```

The individual technologies aren't the final objective.

The objective is being able to combine them into a **coherent network design**.

---

# 12. Final Mental Model

Keep this model:

```text
                  SECURE NETWORK
                       │
          ┌────────────┼────────────┐
          │            │            │
          ↓            ↓            ↓
      WHO CAN       WHO CAN      WHO CAN
      CONNECT?      PROVIDE      CLAIM AN
                    DHCP?        ARP IDENTITY?
          │            │            │
          ↓            ↓            ↓
   Port Security  DHCP Snooping     DAI
          │            │            │
          └────────────┼────────────┘
                       ↓
                 Defense in Depth
                       ↓
              Standardized Policy
                       ↓
              Repeatable Deployment
```

## The real takeaway

**Don't let Layer 2 security remain something you studied for the CCNA.**

Make it part of your default network-building process:

> **Define the policy → create the template → deploy it → verify it → document it → troubleshoot it when it breaks.**

That is what turns today's commands into an actual networking skill.

And for your Castle Rysen project, **S20-L04 is effectively the point where the Layer 2 security features you've learned become a standardized deployment rather than isolated labs.**
