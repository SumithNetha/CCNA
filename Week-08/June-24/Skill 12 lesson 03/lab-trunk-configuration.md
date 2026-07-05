# Packet Tracer Lab

## Objective

Configure trunk links between two Cisco switches.

Verify VLAN communication across switches.

Demonstrate VLAN isolation.

---

## Topology

PC1
|
SW1
||
|| Trunk
||
SW2
|
PC2

Router connected to SW1

---

## VLANs

VLAN 10

ADMIN-DEVICES

VLAN 20

PATRON-DEVICES

---

## Part 1

Verify VLANs

show vlan

Result

VLAN 10 created

VLAN 20 created

---

## Part 2

Configure Trunk Links

interface range fa0/1-2

switchport mode trunk

---

## Verification

show interfaces trunk

Result

Mode

trunk

Encapsulation

802.1Q

Status

trunking

Allowed VLANs

1-1005

---

## Part 3

Configure Access Ports

Switch 1

Fa0/3

switchport mode access

switchport access vlan 10

Switch 2

Fa0/3

switchport mode access

switchport access vlan 10

---

## Connectivity Test

PC1

10.0.18.2

PC2

10.0.18.3

Ping

Successful

Reason

Same VLAN

Same Subnet

Trunk carrying VLAN 10

---

## Allowed VLAN Test

Configured

switchport trunk allowed vlan 10

Observed

Only VLAN 10 remained on trunk.

Restored

no switchport trunk allowed vlan 10

Allowed VLANs returned to default.

---

## VLAN Isolation Test

Changed

Switch2

Fa0/3

switchport access vlan 20

Result

PC1

VLAN10

PC2

VLAN20

Ping initially succeeded due to cached ARP entries.

After ARP cache expired:

Request Timed Out

Reason

ARP broadcasts do not cross VLAN boundaries.

---

## STP Observation

Fa0/1

Forwarding

Fa0/2

Blocked (none)

Reason

Spanning Tree prevented Layer 2 loops.

---

## Conclusion

Successfully demonstrated:

- Trunk configuration
- VLAN extension
- VLAN isolation
- Trunk verification
- Allowed VLAN behavior
- STP operation