# Packet Tracer Lab
## Skill 12 Lesson 02 – Creating and Naming VLANs

---

# Device 1

Hostname

```
cafe01-sw02
```

---

## Configure Management SVI

```cisco
configure terminal

interface vlan 1

no ip address

ip address 10.0.18.3 255.255.255.192

end

write memory
```

---

## Connectivity Test

```cisco
ping 10.0.18.1
```

Result

- First ping: 80%
- Second ping: 100%

Reason

First packet performs ARP resolution.

---

## Create VLANs

```cisco
configure terminal

vlan 10
name ADMIN-DEVICES

vlan 20
name PATRON-DEVICES

end
```

---

## Verification

```cisco
show vlan
```

Result

```
VLAN 10 ADMIN-DEVICES

VLAN 20 PATRON-DEVICES
```

---

# Device 2

Hostname

```
cafe01-sw1
```

---

## Create VLANs

```cisco
configure terminal

vlan 10
name ADMIN-DEVICES

vlan 20
name PATRON-DEVICES

end
```

---

## Configure Access Ports

```cisco
configure terminal

interface range fa0/10-15

switchport mode access

switchport access vlan 20

end

write memory
```

---

## Verification

```cisco
show vlan
```

Result

```
VLAN 20

Fa0/10
Fa0/11
Fa0/12
Fa0/13
Fa0/14
Fa0/15
```

Interfaces successfully moved from VLAN 1 to VLAN 20.