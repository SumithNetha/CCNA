# Commands Used

## Verify VLANs

show vlan

---

## Verify Trunks

show interfaces trunk

---

## Configure Trunk

interface range fa0/1-2

switchport mode trunk

---

## Configure Access Port

interface fa0/3

switchport mode access

switchport access vlan 10

---

## Change VLAN

switchport access vlan 20

---

## Restrict Allowed VLANs

switchport trunk allowed vlan 10

---

## Restore Defaults

no switchport trunk allowed vlan 10

---

## Useful Verification

show running-config

show interfaces trunk

show interfaces status

show spanning-tree

show mac address-table

show vlan brief