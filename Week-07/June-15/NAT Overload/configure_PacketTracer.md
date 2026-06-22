# Router Configuration – NAT Overload (PAT)

## Router: cafe01-RT01

```bash
enable

configure terminal

interface GigabitEthernet0/0
 ip address 192.168.1.1 255.255.255.0
 ip nat inside
 no shutdown

interface GigabitEthernet0/0/0
 ip address 192.168.2.1 255.255.255.0
 ip nat inside
 no shutdown

interface GigabitEthernet0/1
 ip address 216.0.5.2 255.255.255.252
 ip nat outside
 no shutdown

access-list 1 permit 192.168.1.0 0.0.0.255
access-list 1 permit 192.168.2.0 0.0.0.255

ip nat inside source list 1 interface GigabitEthernet0/1 overload

ip route 0.0.0.0 0.0.0.0 216.0.5.1

end

copy running-config startup-config
```

---

## Verify

```bash
show ip nat translations

show ip nat statistics

show access-lists

show ip interface brief
```



## Expected Result

Multiple inside private IP addresses share:

216.0.5.2

using unique source ports.
