# Lab 04 – Router-on-a-Stick (Inter-VLAN Routing)

## Lab Objective

Configure Router-on-a-Stick (ROAS) to enable communication between VLAN 10 and VLAN 20 using a single physical router interface.

---

# Lab Topology

```
             ISP Router
                 |
           216.0.5.0/24
                 |
          Gi0/1  Router  Gi0/0
                     |
                 802.1Q Trunk
                     |
              +--------------+
              |   Switch 1   |
              +--------------+
              |              |
           Fa0/3          Fa0/24
             |               |
            PC1         Trunk to Router
             |
         VLAN 10

                 ||
             Trunk Links
                 ||

              +--------------+
              |   Switch 2   |
              +--------------+
              |              |
           Fa0/3          Server
             |
            PC2
         VLAN 20
```

---

# IP Addressing

| Device | Interface | IP Address |
|----------|-----------|------------|
| Router VLAN10 | Gi0/0.10 | 10.0.18.1/27 |
| Router VLAN20 | Gi0/0.20 | 10.0.18.33/27 |
| WAN | Gi0/1 | 216.0.5.2/30 |
| PC1 | DHCP | 10.0.18.2 |
| PC2 | DHCP | 10.0.18.34 |

---

# VLAN Information

| VLAN | Name | Network |
|------|------|-----------|
| 10 | ADMIN-DEVICES | 10.0.18.0/27 |
| 20 | PATRON-DEVICES | 10.0.18.32/27 |

---

# Router Configuration

## Configure Physical Interface

```cisco
enable

configure terminal

interface GigabitEthernet0/0
 no ip address
 ip nat inside
 no shutdown
```

---

## Configure VLAN 10 Subinterface

```cisco
interface GigabitEthernet0/0.10
 encapsulation dot1Q 10
 ip address 10.0.18.1 255.255.255.224
```

---

## Configure VLAN 20 Subinterface

```cisco
interface GigabitEthernet0/0.20
 encapsulation dot1Q 20
 ip address 10.0.18.33 255.255.255.224
```

---

# DHCP Configuration

## Exclude Gateway Addresses

```cisco
ip dhcp excluded-address 10.0.18.1
ip dhcp excluded-address 10.0.18.33
```

---

## VLAN 10 DHCP Pool

```cisco
ip dhcp pool ADMIN-10
 network 10.0.18.0 255.255.255.224
 default-router 10.0.18.1
 dns-server 1.1.1.1
```

---

## VLAN 20 DHCP Pool

```cisco
ip dhcp pool PATRON-20
 network 10.0.18.32 255.255.255.224
 default-router 10.0.18.33
 dns-server 1.1.1.1
```

---

# Switch 1 Configuration

## Configure Router Link

```cisco
interface FastEthernet0/24
 switchport mode trunk
```

---

## Configure PC1

```cisco
interface FastEthernet0/3
 switchport mode access
 switchport access vlan 10
```

---

# Switch 2 Configuration

## Configure Trunk

```cisco
interface range FastEthernet0/1-2
 switchport mode trunk
```

---

## Configure PC2

```cisco
interface FastEthernet0/3
 switchport mode access
 switchport access vlan 20
```

---

# Verification Commands

## Router

```cisco
show ip interface brief

show ip dhcp binding

show running-config
```

---

## Switch

```cisco
show interfaces trunk

show vlan brief
```

---

# PC Verification

```text
ipconfig

ping 10.0.18.34

tracert 10.0.18.34

ping 1.1.1.1
```

---

# Expected Results

## DHCP

PC1

```
IP Address : 10.0.18.2

Gateway : 10.0.18.1
```

PC2

```
IP Address : 10.0.18.34

Gateway : 10.0.18.33
```

---

## Trunk Verification

```
show interfaces trunk
```

Expected

```
802.1Q

Status : trunking

Allowed VLANs : 1-1005

Active VLANs : 1,10,20
```

---

## Inter-VLAN Routing

PC1

```
ping 10.0.18.34
```

Expected

```
Reply from 10.0.18.34
```

---

## Traceroute

```
tracert 10.0.18.34
```

Expected

```
Hop 1

10.0.18.1

Hop 2

10.0.18.34
```

---

## Internet Connectivity

```
ping 1.1.1.1
```

Expected

```
Replies received after initial ARP resolution.
```

---

# Troubleshooting

| Problem | Possible Cause |
|----------|----------------|
| 169.254.x.x Address | DHCP failed |
| Cannot Ping Between VLANs | Missing router subinterface |
| No DHCP Lease | Switch-to-router link is not a trunk |
| Router Doesn't Receive Traffic | Missing `encapsulation dot1Q` |
| Hosts Receive Wrong Network | Incorrect DHCP pool |
| Trunk Down | Incorrect switchport configuration |

---

# Lab Summary

Successfully completed:

- Created Router-on-a-Stick configuration
- Configured router subinterfaces
- Configured 802.1Q encapsulation
- Configured DHCP for multiple VLANs
- Configured switch trunk to router
- Assigned access ports to VLANs
- Verified DHCP address allocation
- Verified Inter-VLAN Routing
- Verified routing using `tracert`
- Verified connectivity to external network