# S22-L02 Lab Notes — Setting the Clock with NTP

Your lab demonstrates the complete NTP workflow: **manually establish the router's time → make the router an NTP master → configure the switch as an NTP client → verify synchronization.**

---

## 1. Check the router's initial clock

```cisco
cafe01-RT01#show clock
*14:1:11.160 UTC Mon Mar 22 1993
```

Detailed:

```cisco
cafe01-RT01#show clock detail
*14:1:18.171 UTC Mon Mar 22 1993
Time source is hardware calendar
```

### What this tells us

The router initially has an incorrect/default date.

```text
Time source is hardware calendar
```

means the current clock is coming from the device's hardware calendar rather than NTP or manual user configuration.

---

# 2. Manually set the router's clock

The command syntax is:

```cisco
clock set HH:MM:SS DAY MONTH YEAR
```

You configured:

```cisco
clock set 11:22:33 1 sept 2026
```

Verification:

```cisco
cafe01-RT01#show clock
11:22:37.23 UTC Tue Sep 1 2026
```

Then:

```cisco
cafe01-RT01#show clock detail
11:22:41.41 UTC Tue Sep 1 2026
Time source is user configuration
```

### Important change

Before:

```text
Time source is hardware calendar
```

After `clock set`:

```text
Time source is user configuration
```

So `show clock detail` is useful for identifying **where the device's time is coming from**.

---

# 3. Configure the timezone

You configured India Standard Time:

```cisco
conf t
clock timezone IST 5 30
```

Before timezone configuration:

```text
11:22 UTC
```

After:

```text
16:53 IST
```

Because IST is **UTC+5:30**.

Later verification showed:

```cisco
cafe01-RT01#show clock
17:1:5.849 IST Tue Sep 1 2026
```

### Important concept

The timezone configuration changes the **displayed local time**.

It does not mean you changed the underlying NTP synchronization mechanism.

Keep these concepts separate:

```text
NTP
 └── Synchronizes time

Timezone
 └── Determines local-time display
```

---

# 4. Configure the router as an NTP master

You entered:

```cisco
conf t
ntp master 1
```

This tells the router to act as an **NTP master clock**.

The router can now provide time to other NTP clients.

Your topology is essentially:

```text
              cafe01-RT01
             NTP Master
             10.0.18.1
                  │
                  │ NTP
                  ▼
              cafe01-sw1
              NTP Client
              10.0.18.2
```

---

# 5. Understanding `ntp master 1`

Your lab produced an interesting result.

After configuring:

```cisco
ntp master 1
```

you initially saw:

```text
Clock is synchronized, stratum 1
reference is 127.127.1.1
```

Later, after changing the master configuration, you saw:

```text
Clock is synchronized, stratum 8
reference is 127.127.1.1
```

and then eventually:

```text
Clock is synchronized, stratum 1
reference is 127.127.1.1
```

The key lab observation is that the NTP master configuration affects the router's reported stratum.

Your final state was:

```text
Clock is synchronized, stratum 1
reference is 127.127.1.1
```

### `127.127.1.1`

This is the local NTP reference used by the Cisco NTP master in your lab.

You can see it in:

```cisco
show ntp associations
```

Output:

```text
*~127.127.1.1   .LOCL.   0   51   64   377
```

---

# 6. Configure the switch's management SVI

The switch needed IP connectivity to the router before NTP could work.

You configured:

```cisco
cafe01-sw1(config)#interface vlan 10
cafe01-sw1(config-if)#ip address 10.0.18.2 255.255.255.192
```

So:

```text
Switch VLAN 10
IP:      10.0.18.2
Mask:    255.255.255.192
Prefix:  /26
```

The router/NTP server is:

```text
10.0.18.1
```

Both addresses belong to:

```text
10.0.18.0/26
```

---

# 7. Verify connectivity before NTP

You tested:

```cisco
cafe01-sw1#ping 10.0.18.1
```

Result:

```text
Success rate is 80 percent (4/5)
```

So the switch was able to reach the router.

This is an important troubleshooting principle:

> **NTP cannot work if the client cannot reach the NTP server.**

---

# 8. Configure the switch timezone

The switch initially showed:

```cisco
cafe01-sw1#show clock
*9:56:34.625 UTC Mon May 10 1993
```

Then you configured:

```cisco
clock timezone IST 5 30
```

The displayed time changed to:

```text
15:28:15.800 IST Mon May 10 1993
```

Notice something important:

**The timezone changed, but the date/time itself was still wrong.**

That's because timezone configuration does **not synchronize the clock**.

---

# 9. Configure the switch as an NTP client

You configured:

```cisco
ntp server 10.0.18.1
```

This tells the switch:

> Use `10.0.18.1` as my NTP server.

Therefore:

```text
cafe01-RT01
10.0.18.1
     │
     │ NTP
     ▼
cafe01-sw1
10.0.18.2
```

---

# 10. Initial NTP status — not synchronized

Immediately after configuration:

```cisco
cafe01-sw1#show ntp status
```

You got:

```text
Clock is unsynchronized, stratum 16, no reference clock
```

This is an important state.

### Stratum 16

In this context:

```text
Stratum 16
    ↓
Not synchronized / no usable reference
```

The switch had an NTP server configured, but synchronization had **not yet completed**.

---

# 11. Check NTP association

You used:

```cisco
show ntp associations
```

Output:

```text
address         ref clock       st   when     poll    reach
~10.0.18.1      127.127.1.1     1    11       16      37
```

The symbols at the bottom explain the output:

```text
* sys.peer
# selected
+ candidate
- outlyer
x falseticker
~ configured
```

Your server had:

```text
~10.0.18.1
```

The `~` means the server was **configured**.

But at that moment the switch still wasn't synchronized.

---

# 12. Wait for synchronization

After waiting, you checked again:

```cisco
show ntp status
```

And finally received:

```text
Clock is synchronized, stratum 2, reference is 10.0.18.1
```

🎯 **NTP synchronization succeeded.**

This is the exact hierarchy you wanted:

```text
              Router
          NTP Master
           Stratum 1
          10.0.18.1
               │
               │
               ▼
             Switch
          NTP Client
           Stratum 2
          10.0.18.2
```

The switch is one synchronization level below the router.

---

# 13. Verify the NTP association

Final switch output:

```cisco
cafe01-sw1#show ntp associations

address         ref clock       st   when     poll    reach
*~10.0.18.1     127.127.1.1     1    10       16      177
```

The important part is:

```text
*~10.0.18.1
```

### `~`

The server is configured.

### `*`

The server is currently the **system peer** — the NTP source selected for synchronization.

So:

```text
~ = configured
* = selected/current synchronization peer
```

That's a very useful verification distinction.

---

# 14. Final router verification

On the router:

```cisco
show ntp status
```

```text
Clock is synchronized, stratum 1, reference is 127.127.1.1
```

And:

```cisco
show ntp associations
```

```text
*~127.127.1.1   .LOCL.   0
```

The router is using its local NTP master reference.

---

# 15. Final switch verification

On the switch:

```cisco
show ntp status
```

```text
Clock is synchronized, stratum 2, reference is 10.0.18.1
```

Then:

```cisco
show ntp associations
```

```text
*~10.0.18.1
```

And finally:

```cisco
show clock detail
```

```text
17:31:13.678 IST Tue Sep 1 2026
Time source is NTP
```

This is probably the **most important final verification**.

Initially:

```text
Time source is hardware calendar
```

Then after `clock set`:

```text
Time source is user configuration
```

Finally:

```text
Time source is NTP
```

That demonstrates the entire progression.

---

# 🔥 The Complete Lab Flow

```text
                    START
                      │
                      ▼
        show clock / show clock detail
                      │
                      ▼
         Hardware calendar detected
                      │
                      ▼
              clock set ...
                      │
                      ▼
         Time source = user config
                      │
                      ▼
       clock timezone IST 5 30
                      │
                      ▼
             ntp master 1
                      │
                      ▼
           Router = NTP Master
             Stratum 1
                      │
                      │ 10.0.18.1
                      ▼
       Configure switch SVI
          10.0.18.2/26
                      │
                      ▼
          ping 10.0.18.1
                      │
                      ▼
             Connectivity OK
                      │
                      ▼
         clock timezone IST 5 30
                      │
                      ▼
       ntp server 10.0.18.1
                      │
                      ▼
          Initially unsynchronized
             Stratum 16
                      │
                      ▼
                WAIT ⏳
                      │
                      ▼
          NTP synchronization
                      │
                      ▼
             Switch = Stratum 2
                      │
                      ▼
         show clock detail
                      │
                      ▼
          Time source is NTP
```

---

# 🧠 Commands to Remember

### Router — NTP master

```cisco
clock set HH:MM:SS DAY MONTH YEAR
clock timezone IST 5 30
ntp master 1
show ntp status
show ntp associations
show clock detail
```

### Switch — NTP client

```cisco
interface vlan 10
ip address 10.0.18.2 255.255.255.192

clock timezone IST 5 30
ntp server 10.0.18.1

show ntp status
show ntp associations
show clock detail
```

---

## ⭐ What this lab actually proves

The most important thing isn't memorizing `ntp master` or `ntp server`.

You demonstrated the **entire NTP relationship**:

**RT01 → authoritative NTP master → Stratum 1**

**SW1 → NTP client → synchronizes from RT01 → Stratum 2**

And the final proof was:

```text
Router:
Clock is synchronized, stratum 1

Switch:
Clock is synchronized, stratum 2

Switch:
Time source is NTP
```

That's the state you should be able to recognize immediately in a CCNA troubleshooting scenario.
