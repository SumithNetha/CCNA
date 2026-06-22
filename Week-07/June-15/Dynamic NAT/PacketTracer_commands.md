# Dynamic NAT – Full Configuration Commands

## Step 1 — Enter Privileged Mode

enable

---

## Step 2 — Enter Global Configuration

configure terminal

---

## Step 3 — Configure INSIDE Interface (LAN)

interface gigabitEthernet0/0

ip address 192.168.10.1 255.255.255.0

no shutdown

ip nat inside

exit

---

## Step 4 — Configure OUTSIDE Interface (ISP)

interface gigabitEthernet0/1

ip address 216.0.5.1 255.255.255.0

no shutdown

ip nat outside

exit

---

## Step 5 — Configure ACL

(Identify private addresses allowed to translate)

access-list 1 permit 192.168.10.0 0.0.0.255

exit

---

## Step 6 — Create NAT Pool

ip nat pool cafepublic 216.0.5.50 216.0.5.100 netmask 255.255.255.0

---

## Step 7 — Enable Dynamic NAT

ip nat inside source list 1 pool cafepublic

---

## Step 8 — Configure Default Route

ip route 0.0.0.0 0.0.0.0 216.0.5.254

---

## Step 9 — Save Configuration

end

copy running-config startup-config


---
## If adding second network (Fallout Shelter)
configure terminal

access-list 1 permit 192.168.20.0 0.0.0.255

end