# Trunk Troubleshooting

## Problem

Devices in same VLAN cannot communicate.

Check

show interfaces trunk

---

## Problem

Trunk not operational.

Verify

switchport mode trunk

---

## Problem

Only one VLAN passing.

Cause

switchport trunk allowed vlan 10

Solution

no switchport trunk allowed vlan 10

or

switchport trunk allowed vlan add 20

---

## Problem

Ping works briefly then fails.

Cause

ARP cache expired.

Devices moved into different VLANs.

---

## Problem

Duplicate IP Warning

%IP-4-DUPADDR

Possible Causes

- Duplicate static IP
- Duplicate DHCP lease
- Duplicate SVI IP

---

## Problem

One trunk forwarding

Other trunk blocked

Cause

Spanning Tree Protocol

Normal behavior when redundant links exist.

---

## Verification Commands

show interfaces trunk

show vlan

show spanning-tree

show mac address-table

show ip interface brief