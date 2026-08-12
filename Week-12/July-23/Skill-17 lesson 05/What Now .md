# Week 12 — July 23

## Skill 17 Lesson 05 — What Now? IPv6

This lesson is essentially the **wrap-up of IPv6**. The main message is that you don't need to immediately deploy IPv6 everywhere, but you **do need to understand it well enough to recognize, configure, and work with it when it appears in an enterprise environment**.

---

## 1. What Should You Do After Learning IPv6?

The lesson's practical answer is:

> **Learn it, understand it, keep it fresh, and be ready when you need it.**

IPv6 does **not** mean that you should immediately replace IPv4 in an existing network.

Instead, you should be comfortable with:

* Why IPv6 exists
* How IPv6 addressing works
* How IPv6 differs from IPv4
* Global unicast addressing
* IPv6 address structure
* Enterprise IPv6 usage
* Dual-stack operation
* Basic IPv6 configuration and troubleshooting

The goal is to reach the point where seeing an IPv6 address doesn't feel unfamiliar or intimidating.

---

# 2. IPv6 Is Not About Replacing IPv4 Tomorrow

A major point of the lesson is that **IPv4 is not suddenly disappearing**.

Many networks still depend heavily on IPv4, and IPv6 adoption across enterprise environments is uneven.

Therefore:

```text
IPv4
  +
IPv6
  ↓
Real-world networking
```

You should be prepared to work with either protocol.

---

# 3. Why IPv6 Still Matters

Even though IPv6 may not be used heavily in every environment today, the lesson considers it an important technology for the future.

IPv6 provides:

* A massively larger address space
* A different approach to addressing
* Less dependence on IPv4 address conservation techniques
* Better scalability for large environments

The important mindset is:

> **Knowing IPv6 today prepares you for networks that will use it more heavily in the future.**

---

# 4. Don't Memorize Everything Forever

One of the more practical points in the lesson is that **forgetting syntax is normal**.

You may later forget:

* Exact commands
* Address formatting
* Specific configuration syntax
* Some IPv6 structures

That doesn't mean you failed to learn IPv6.

If you understand the underlying concepts, you can refresh the syntax when you need it.

### The important distinction

```text
Understanding
     ↓
Familiarity
     ↓
Practical experience
     ↓
Refresher when required
```

The objective isn't permanent memorization of every command.

---

# 5. NetworkChuck Coffee Example

The lesson uses **NetworkChuck Coffee** to demonstrate when IPv6 becomes more relevant.

Suppose the coffee shop has:

```text
One location
Few devices
Simple IPv4 network
```

There may be little reason to immediately redesign everything around IPv6.

But as the business grows:

```text
More locations
     ↓
More devices
     ↓
More providers
     ↓
Cloud integrations
     ↓
More complex network
     ↓
Greater IPv6 relevance
```

So the important question isn't:

> "Is IPv6 trendy?"

It's:

> **"Does the scale and environment make IPv6 relevant?"**

---

# 6. Exam Knowledge vs Job Knowledge

This is an important distinction from the lesson.

### For the CCNA/exam

You need to know:

* How IPv6 works
* IPv6 addressing
* Address types
* IPv6 configuration
* Routing concepts

### On the job

You also need to know:

> **When does IPv6 actually matter in this environment?**

Those aren't necessarily the same question.

Knowing the technology doesn't mean you should deploy it unnecessarily.

---

# 7. Dual Stack

One particularly important practical concept mentioned is **dual stack**.

Dual stack means operating:

```text
IPv4
 +
IPv6
```

at the same time.

For example:

```text
             Router
          /           \
       IPv4           IPv6
        |               |
   IPv4 networks    IPv6 networks
```

This allows an organization to support IPv6 while continuing to operate its existing IPv4 infrastructure.

### Why this matters

If your:

* ISP
* Cloud provider
* Enterprise
* Network team

starts introducing IPv6, you don't necessarily have to think in terms of:

```text
IPv4 OFF
IPv6 ON
```

A transition can involve **both protocols operating together**.

---

# 8. Practical IPv6 Knowledge Bank

The lesson gives four practical next steps.

### 1. Understand the purpose

Don't focus only on the formatting of an IPv6 address.

Understand **why IPv6 exists and what problem it solves**.

### 2. Recognize address types

Especially understand enterprise-relevant addressing such as:

```text
Global Unicast
```

You should also be familiar with the other address types you've encountered during Skill 17.

### 3. Be prepared to revisit it

You may not use IPv6 every day.

That's okay.

When your environment starts using it, revisit the details.

### 4. Stay curious

IPv6 adoption is increasing, even if it isn't happening at the same pace everywhere.

---

# 9. Where You Are After Skill 17

After completing this IPv6 section, the expected level isn't:

> "I can architect a massive global IPv6 deployment right now."

Instead, it is:

```text
See IPv6 address
       ↓
Recognize what it is
       ↓
Understand why it exists
       ↓
Understand how it is used
       ↓
Configure/work with it when required
```

That's the practical target.

---

# 10. Connection to Your Castle Rysen Lab

Your previous lab demonstrates exactly why this matters.

You went from understanding IPv6 to actually building:

```text
Cafe
2001:DB8:1:1::/64
2001:DB8:1:2::/64
        |
        |
2001:DB8:1:3::/64
     IPv6 WAN
        |
        |
2001:DB8:1:4::/64
2001:DB8:1:5::/64
2001:DB8:1:6::/64
2001:DB8:1:7::/64
Fallout Shelter
```

You configured:

* IPv6 unicast routing
* Global unicast addresses
* EUI-64
* Link-local addresses
* IPv6 WAN addressing
* Static IPv6 routes
* IPv6 connectivity testing

So this lesson isn't introducing another major configuration technique. It's telling you **how to retain and use what you just learned**.

---

# 11. Key Takeaways

```text
IPv6 is important
       ↓
But don't assume immediate IPv4 replacement
       ↓
Understand the fundamentals
       ↓
Recognize IPv6 addresses
       ↓
Know global unicast and other important address types
       ↓
Understand dual stack
       ↓
Refresh syntax when needed
       ↓
Be ready when IPv6 appears in your environment
```

### The core idea

**IPv6 is a "be ready when it matters" skill.**

You don't need to panic if your current environment is primarily IPv4. The valuable outcome is that when IPv6 eventually appears in a production network, you can look at it and think:

> **"I know what I'm looking at, I know why it's there, and I know how to work with it."**

That is the intended endpoint of **Skill 17**.
