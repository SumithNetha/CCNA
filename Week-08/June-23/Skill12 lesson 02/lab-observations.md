# Lab Observations

## Observation 1

The first ping showed:

```
.!!!!
```

Explanation

- First ICMP packet failed.
- Switch first learned the destination MAC address using ARP.
- Remaining packets succeeded.

---

## Observation 2

Creating VLANs does not automatically assign interfaces.

Example

```
vlan 20
```

Only creates VLAN 20.

Interfaces remain in VLAN 1 until manually assigned.

---

## Observation 3

Incorrect command used

```cisco
switchport mode access vlan 20
```

Cisco returned

```
% Invalid input detected
```

Reason

This combines two separate commands.

Correct

```cisco
switchport mode access

switchport access vlan 20
```

---

## Observation 4

Using

```cisco
interface range fa0/10-15
```

allows configuring multiple interfaces simultaneously.

This is faster and reduces configuration errors.

---

## Observation 5

Verification using

```cisco
show vlan
```

confirmed:

- VLAN creation
- VLAN names
- Port membership

Always verify configuration after making changes.

---

## Observation 6

Management IP addresses

```
cafe01-sw1

10.0.18.2/26
```

```
cafe01-sw02

10.0.18.3/26
```

Used for remote switch management through VLAN 1.