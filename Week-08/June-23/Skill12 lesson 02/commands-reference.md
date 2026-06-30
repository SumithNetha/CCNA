# Cisco Commands Reference

## Verification

```cisco
show ip interface brief

show vlan

show vlan brief

show interfaces switchport

show interfaces trunk

show running-config

ping 10.0.18.1
```

---

## VLAN Configuration

```cisco
configure terminal

vlan 10
name ADMIN-DEVICES

vlan 20
name PATRON-DEVICES
```

---

## Interface Configuration

```cisco
interface range fa0/10-15

switchport mode access

switchport access vlan 20
```

---

## SVI Configuration

```cisco
interface vlan 1

ip address 10.0.18.3 255.255.255.192

no shutdown
```

---

## Save Configuration

```cisco
write memory
```
