# Skill 12 Lesson 09 – Castle Rysen Fallout Shelter VLAN Implementation

## Summer of CCNA – Week 8

## Overview

The Castle Rysen Fallout Shelter network was redesigned from one large `/23` network into four separate VLANs and four `/25` subnets.

The complete implementation included:

- Analyzing the RFP requirements
- Subnetting the original `/23` network
- Creating four VLANs
- Configuring VTP
- Configuring DTP
- Establishing 802.1Q trunks
- Configuring access ports
- Configuring Router-on-a-Stick (ROAS)
- Configuring inter-VLAN routing
- Configuring DHCP
- Testing DHCP address assignment
- Configuring switch management IP addresses
- Configuring switch default gateways
- Verifying Spanning Tree Protocol
- Testing end-to-end connectivity
- Saving all device configurations

---

# 1. Original Fallout Shelter Network

The Castle Rysen Fallout Shelter was originally assigned the following network:

```text
10.0.16.0/23

Subnet mask:

255.255.254.0

The RFP required four separate logical network segments:

MGMT
INTERNAL
VIDEO
GUEST

Therefore, the original /23 network needed to be divided into four smaller networks.

2. Why VLAN Implementation Is a Network Redesign

Implementing VLANs is not simply creating VLAN numbers on switches.

Every VLAN creates a separate Layer 2 broadcast domain.

Each VLAN also requires:

A VLAN ID
A VLAN name
A separate IP subnet
A default gateway
A DHCP scope
Inter-VLAN routing
Correct access port assignments
Correct trunk configuration

The original network:

10.0.16.0/23

was divided into four /25 networks.

10.0.16.0/23
        |
        +---- VLAN 10 MGMT
        |     10.0.16.0/25
        |
        +---- VLAN 20 INTERNAL
        |     10.0.16.128/25
        |
        +---- VLAN 30 VIDEO
        |     10.0.17.0/25
        |
        +---- VLAN 40 GUEST
              10.0.17.128/25
3. Subnet Design
VLAN 10 – MGMT
Network Address:    10.0.16.0/25
Subnet Mask:        255.255.255.128
Default Gateway:    10.0.16.1
Usable Range:       10.0.16.1 - 10.0.16.126
Broadcast Address:  10.0.16.127
VLAN 20 – INTERNAL
Network Address:    10.0.16.128/25
Subnet Mask:        255.255.255.128
Default Gateway:    10.0.16.129
Usable Range:       10.0.16.129 - 10.0.16.254
Broadcast Address:  10.0.16.255
VLAN 30 – VIDEO
Network Address:    10.0.17.0/25
Subnet Mask:        255.255.255.128
Default Gateway:    10.0.17.1
Usable Range:       10.0.17.1 - 10.0.17.126
Broadcast Address:  10.0.17.127
VLAN 40 – GUEST
Network Address:    10.0.17.128/25
Subnet Mask:        255.255.255.128
Default Gateway:    10.0.17.129
Usable Range:       10.0.17.129 - 10.0.17.254
Broadcast Address:  10.0.17.255
4. Final VLAN and Subnet Table
VLAN	Name	Network	Default Gateway
10	MGMT	10.0.16.0/25	10.0.16.1
20	INTERNAL	10.0.16.128/25	10.0.16.129
30	VIDEO	10.0.17.0/25	10.0.17.1
40	GUEST	10.0.17.128/25	10.0.17.129
5. Verify the Physical Topology

Before configuring VLANs and trunks, verify how the switches are physically connected.

Use:

show cdp neighbors

Example from FO-SW01:

FO-SW02    Fa0/6    Fa0/6
FO-SW02    Fa0/7    Fa0/7
FO-SW03    Fa0/2    Fa0/1
FO-SW04    Fa0/3    Fa0/1
FO-SW05    Fa0/4    Fa0/1
FO-SW06    Fa0/5    Fa0/1

CDP helps identify:

Neighbor device
Local interface
Remote interface
Device platform
Device capability

Useful command:

show cdp neighbors
6. Configure VTP

VTP stands for:

VLAN Trunking Protocol

VTP distributes VLAN database information between Cisco switches.

Important:

VTP does not create trunks.

VTP advertisements travel across already operational trunk links.

The design used:

FO-SW01 → VTP Server

FO-SW02 → VTP Client
FO-SW03 → VTP Client
FO-SW04 → VTP Client
FO-SW05 → VTP Client
FO-SW06 → VTP Client
7. Configure FO-SW01 as VTP Server
configure terminal

vtp domain FALLOUT
vtp mode server
vtp version 1
vtp password cisco123

end

Verify:

show vtp status

Expected:

VTP Operating Mode: Server
VTP Domain Name: FALLOUT
VTP Version Running: 1
8. Configure FO-SW02 through FO-SW06 as VTP Clients
configure terminal

vtp domain FALLOUT
vtp mode client
vtp password cisco123

end

Repeat on:

FO-SW02
FO-SW03
FO-SW04
FO-SW05
FO-SW06

Verify:

show vtp status

Expected:

VTP Operating Mode: Client
VTP Domain Name: FALLOUT
VTP Version Running: 1
9. VTP Version Mismatch Troubleshooting

During the implementation, FO-SW01 was running:

VTP Version 2

while FO-SW02 was running:

VTP Version 1

Attempting the following on FO-SW02 failed:

vtp version 2

Error:

Cannot modify version in VTP client mode

The solution used in the lab was to change the VTP server to version 1:

configure terminal

vtp version 1

end

Verify:

show vtp status

Afterward, FO-SW01 displayed:

VTP Version Running: 1
10. Configure the VTP Password

The same VTP password must be configured on all participating switches.

configure terminal

vtp password cisco123

end

Repeat on every switch.

A mismatched VTP password can prevent VLAN information from successfully synchronizing.

11. Create VLANs on FO-SW01

Because FO-SW01 is the VTP server, create the VLANs there.

configure terminal

vlan 10
 name MGMT

vlan 20
 name INTERNAL

vlan 30
 name VIDEO

vlan 40
 name GUEST

end

Verify:

show vlan brief

Expected:

10    MGMT
20    INTERNAL
30    VIDEO
40    GUEST
12. Configure DTP and Trunks

DTP stands for:

Dynamic Trunking Protocol

DTP allows Cisco switches to negotiate whether an interface should operate as an access port or trunk port.

The switch-to-switch interfaces were configured using:

switchport mode dynamic desirable

Example:

configure terminal

interface range fa0/2-7
 switchport mode dynamic desirable

end
13. Dynamic Auto vs Dynamic Desirable

A port in:

dynamic auto

waits for the neighboring switch to initiate trunk negotiation.

A port in:

dynamic desirable

actively attempts to negotiate a trunk.

Example:

Dynamic Auto + Dynamic Auto
            =
        No Trunk
Dynamic Desirable + Dynamic Auto
                 =
               Trunk
Dynamic Desirable + Dynamic Desirable
                    =
                  Trunk
14. Why Dynamic Auto Did Not Appear in Running Config

Before configuring dynamic desirable, the switch interfaces did not display:

switchport mode dynamic auto

inside:

show running-config

This is because IOS commonly hides commands that are using default values.

Therefore:

Command absent from running-config

does not necessarily mean:

The feature is not configured

Use the following command to see the actual switchport behavior:

show interfaces fa0/3 switchport
15. Verify Trunks

Use:

show interfaces trunk

Example output:

Port        Mode         Encapsulation  Status
Fa0/2       desirable    n-802.1q       trunking
Fa0/3       desirable    n-802.1q       trunking
Fa0/4       desirable    n-802.1q       trunking

Important sections include:

Port
Mode
Encapsulation
Status
Native VLAN
VLANs allowed on trunk
VLANs allowed and active
VLANs in STP forwarding state
16. Detailed Switchport Verification

Use:

show interfaces fa0/3 switchport

Example:

Administrative Mode: dynamic desirable
Operational Mode: trunk
Administrative Trunking Encapsulation: dot1q
Operational Trunking Encapsulation: dot1q
Negotiation of Trunking: On
Access Mode VLAN: 1
Trunking Native Mode VLAN: 1
Trunking VLANs Enabled: All

The most important difference is:

Administrative Mode

means what was configured.

Operational Mode

means what the port is currently doing.

17. Spanning Tree PVID Error Troubleshooting

During trunk configuration, the following messages appeared:

%SPANTREE-2-RECV_PVID_ERR

and:

%SPANTREE-2-BLOCK_PVID_LOCAL

This occurred because one side of a connection was trunking while the other side was still operating as a non-trunk port.

Example:

FO-SW01                      FO-SW05

Non-Trunk                    Trunk
Fa0/4 ---------------------- Fa0/1

              Mismatch

STP detected inconsistent Layer 2 behavior and blocked the port to protect the network.

The solution was to configure matching trunk behavior on both sides.

18. Verify VTP VLAN Replication

On every VTP client switch:

show vlan brief

Expected:

10    MGMT
20    INTERNAL
30    VIDEO
40    GUEST

If VLANs are missing, verify:

show vtp status
show interfaces trunk
show cdp neighbors

Check:

VTP domain
VTP version
VTP password
VTP operating mode
Operational trunk links
19. Configure Access Ports

Four PCs were created to test the VLAN implementation:

PC-MGMT
PC-INTERNAL
PC-VIDEO
PC-GUEST

Each PC was placed into a different VLAN.

Example design:

FO-SW03 → VLAN 10 MGMT PC

FO-SW04 → VLAN 20 INTERNAL PC

FO-SW05 → VLAN 30 VIDEO PC

FO-SW06 → VLAN 40 GUEST PC
20. Configure VLAN 10 Access Port

Example:

configure terminal

interface fa0/3
 switchport mode access
 switchport access vlan 10

end
21. Configure VLAN 20 Access Port
configure terminal

interface fa0/3
 switchport mode access
 switchport access vlan 20

end
22. Configure VLAN 30 Access Port
configure terminal

interface fa0/3
 switchport mode access
 switchport access vlan 30

end
23. Configure VLAN 40 Access Port
configure terminal

interface fa0/3
 switchport mode access
 switchport access vlan 40

end

Verify:

show vlan brief

Detailed verification:

show interfaces fa0/3 switchport
24. Router Interface Was Administratively Down

Initially, FO-RT01 showed:

FastEthernet0/0    unassigned    administratively down    down

The router also showed no CDP neighbors.

This happened because router interfaces are shut down by default.

Enable the interface:

configure terminal

interface fa0/0
 no shutdown

end

Verify:

show ip interface brief

Result:

FastEthernet0/0    unassigned    up    up

After enabling the interface, CDP discovered:

FO-RT01 Fa0/0 ↔ FO-SW01 Fa0/1
25. Configure the Router-Facing Trunk

Router-on-a-Stick requires the switch port connected to the router to carry traffic from multiple VLANs.

On FO-SW01:

configure terminal

interface fa0/1
 switchport mode trunk

end

Verify:

show interfaces trunk

and:

show interfaces fa0/1 switchport

The router-facing interface should be configured as a static trunk because the router does not negotiate DTP like a Cisco switch.

26. Router-on-a-Stick

Router-on-a-Stick allows one physical router interface to route traffic between multiple VLANs.

Physical interface:

FastEthernet0/0

Logical subinterfaces:

Fa0/0.10 → VLAN 10

Fa0/0.20 → VLAN 20

Fa0/0.30 → VLAN 30

Fa0/0.40 → VLAN 40

Each subinterface:

Is associated with a VLAN
Uses 802.1Q encapsulation
Has an IP address
Acts as the default gateway for that VLAN
27. Configure Router-on-a-Stick
Physical Interface
configure terminal

interface fa0/0
 no ip address
 no shutdown

The physical interface does not require an IP address.

The IP addresses are configured on the subinterfaces.

VLAN 10 Subinterface
interface fa0/0.10
 encapsulation dot1Q 10
 ip address 10.0.16.1 255.255.255.128
VLAN 20 Subinterface
interface fa0/0.20
 encapsulation dot1Q 20
 ip address 10.0.16.129 255.255.255.128
VLAN 30 Subinterface
interface fa0/0.30
 encapsulation dot1Q 30
 ip address 10.0.17.1 255.255.255.128
VLAN 40 Subinterface
interface fa0/0.40
 encapsulation dot1Q 40
 ip address 10.0.17.129 255.255.255.128
28. Complete ROAS Configuration
configure terminal

interface fa0/0
 no ip address
 no shutdown

interface fa0/0.10
 encapsulation dot1Q 10
 ip address 10.0.16.1 255.255.255.128

interface fa0/0.20
 encapsulation dot1Q 20
 ip address 10.0.16.129 255.255.255.128

interface fa0/0.30
 encapsulation dot1Q 30
 ip address 10.0.17.1 255.255.255.128

interface fa0/0.40
 encapsulation dot1Q 40
 ip address 10.0.17.129 255.255.255.128

end

Verify:

show ip interface brief
29. Connected and Local Routes

After configuring ROAS:

show ip route

displayed:

C 10.0.16.0/25
L 10.0.16.1/32

C 10.0.16.128/25
L 10.0.16.129/32

C 10.0.17.0/25
L 10.0.17.1/32

C 10.0.17.128/25
L 10.0.17.129/32
30. C = Connected Route

Example:

C 10.0.16.0/25

This means the entire:

10.0.16.0/25

network is directly connected to the router.

The router uses this route to reach devices inside that subnet.

31. L = Local Route

Example:

L 10.0.16.1/32

This represents the router's own interface IP address.

A /32 route represents exactly one IP address.

Therefore:

C 10.0.16.0/25

means:

The complete VLAN 10 network.

While:

L 10.0.16.1/32

means:

The router's own VLAN 10 IP address.

For every configured router interface IP address, IOS automatically creates:

One Connected Route
+
One Local Route

Therefore:

4 VLANs × 2 Routes = 8 Routes
32. Configure DHCP Excluded Addresses

Infrastructure IP addresses should not be dynamically assigned to clients.

The first ten usable addresses from every subnet were excluded.

ip dhcp excluded-address 10.0.16.1 10.0.16.10

ip dhcp excluded-address 10.0.16.129 10.0.16.138

ip dhcp excluded-address 10.0.17.1 10.0.17.10

ip dhcp excluded-address 10.0.17.129 10.0.17.138

These addresses can be used for:

Routers
Switches
Servers
Wireless access points
Other infrastructure devices
33. DHCP Configuration

FO-RT01 was configured as the DHCP server.

MGMT DHCP Pool
ip dhcp pool MGMT
 network 10.0.16.0 255.255.255.128
 default-router 10.0.16.1
 dns-server 1.1.1.1
INTERNAL DHCP Pool
ip dhcp pool INTERNAL
 network 10.0.16.128 255.255.255.128
 default-router 10.0.16.129
 dns-server 1.1.1.1
VIDEO DHCP Pool
ip dhcp pool VIDEO
 network 10.0.17.0 255.255.255.128
 default-router 10.0.17.1
 dns-server 1.1.1.1
GUEST DHCP Pool
ip dhcp pool GUEST
 network 10.0.17.128 255.255.255.128
 default-router 10.0.17.129
 dns-server 1.1.1.1
34. Complete DHCP Configuration
configure terminal

ip dhcp excluded-address 10.0.16.1 10.0.16.10
ip dhcp excluded-address 10.0.16.129 10.0.16.138
ip dhcp excluded-address 10.0.17.1 10.0.17.10
ip dhcp excluded-address 10.0.17.129 10.0.17.138

ip dhcp pool MGMT
 network 10.0.16.0 255.255.255.128
 default-router 10.0.16.1
 dns-server 1.1.1.1

ip dhcp pool INTERNAL
 network 10.0.16.128 255.255.255.128
 default-router 10.0.16.129
 dns-server 1.1.1.1

ip dhcp pool VIDEO
 network 10.0.17.0 255.255.255.128
 default-router 10.0.17.1
 dns-server 1.1.1.1

ip dhcp pool GUEST
 network 10.0.17.128 255.255.255.128
 default-router 10.0.17.129
 dns-server 1.1.1.1

end
35. Duplicate DHCP Exclusion Cleanup

The router originally contained:

ip dhcp excluded-address 10.0.16.1
ip dhcp excluded-address 10.0.16.129
ip dhcp excluded-address 10.0.17.1
ip dhcp excluded-address 10.0.17.129

and also:

ip dhcp excluded-address 10.0.16.1 10.0.16.10
ip dhcp excluded-address 10.0.16.129 10.0.16.138
ip dhcp excluded-address 10.0.17.1 10.0.17.10
ip dhcp excluded-address 10.0.17.129 10.0.17.138

The individual exclusions are redundant because they are already included in the exclusion ranges.

They can be removed:

configure terminal

no ip dhcp excluded-address 10.0.16.1
no ip dhcp excluded-address 10.0.16.129
no ip dhcp excluded-address 10.0.17.1
no ip dhcp excluded-address 10.0.17.129

end
36. Verify DHCP

Verify DHCP pools:

show ip dhcp pool

Verify DHCP bindings:

show ip dhcp binding

Actual lab result:

10.0.16.11
10.0.16.139
10.0.17.11
10.0.17.139

Mapping:

10.0.16.11   → MGMT

10.0.16.139  → INTERNAL

10.0.17.11   → VIDEO

10.0.17.139  → GUEST

This confirms that all four VLANs successfully reached the router's DHCP service.

37. DNS Does Not Provide Internet Connectivity

The DHCP configuration contains:

dns-server 1.1.1.1

This only tells clients which DNS server to use.

It does not provide Internet connectivity.

Therefore:

ping 1.1.1.1

will fail if the network does not have:

ISP connectivity
A default route toward the Internet
NAT/PAT
A valid return path

Also:

Pinging an IP address does not use DNS.

DNS is used when converting names into IP addresses.

Example:

www.example.com
        ↓
       DNS
        ↓
IP Address
38. Configure Switch Management IP Addresses

Layer 2 switches do not require IP addresses to:

Forward Ethernet frames
Operate VLANs
Operate trunks
Run VTP
Run STP

However, switches need IP addresses for remote management.

The management IP address should be configured on:

interface vlan 10

because VLAN 10 is the MGMT VLAN.

39. Switch Management Addressing Plan
Device	Management IP
FO-RT01	10.0.16.1
FO-SW01	10.0.16.2
FO-SW02	10.0.16.3
FO-SW03	10.0.16.4
FO-SW04	10.0.16.5
FO-SW05	10.0.16.6
FO-SW06	10.0.16.7

Subnet mask:

255.255.255.128

Default gateway:

10.0.16.1
40. Configure FO-SW01 Management IP
configure terminal

interface vlan 10
 ip address 10.0.16.2 255.255.255.128
 no shutdown

ip default-gateway 10.0.16.1

end
41. Configure FO-SW02 Management IP
configure terminal

interface vlan 10
 ip address 10.0.16.3 255.255.255.128
 no shutdown

ip default-gateway 10.0.16.1

end
42. Configure FO-SW03 Management IP
configure terminal

interface vlan 10
 ip address 10.0.16.4 255.255.255.128
 no shutdown

ip default-gateway 10.0.16.1

end
43. Configure FO-SW04 Management IP
configure terminal

interface vlan 10
 ip address 10.0.16.5 255.255.255.128
 no shutdown

ip default-gateway 10.0.16.1

end
44. Configure FO-SW05 Management IP
configure terminal

interface vlan 10
 ip address 10.0.16.6 255.255.255.128
 no shutdown

ip default-gateway 10.0.16.1

end
45. Configure FO-SW06 Management IP
configure terminal

interface vlan 10
 ip address 10.0.16.7 255.255.255.128
 no shutdown

ip default-gateway 10.0.16.1

end
46. What Is an SVI?

SVI stands for:

Switched Virtual Interface

Example:

interface vlan 10
 ip address 10.0.16.2 255.255.255.128

This IP address belongs to the switch itself.

It is not assigned to a physical switch port.

Physical Switch Ports
          |
          |
          v
Forward Ethernet Frames


Interface VLAN 10
          |
          |
          v
Management Interface
for the Switch
47. Why Configure ip default-gateway?

The Layer 2 switch needs a default gateway when communicating with devices outside its local management subnet.

Configure:

ip default-gateway 10.0.16.1

The gateway is:

FO-RT01 Fa0/0.10
10.0.16.1/25

Conceptually:

FO-SW01
10.0.16.2
     |
     |
Destination Outside MGMT Network
     |
     v
Default Gateway
10.0.16.1
     |
     v
FO-RT01
48. Verify Switch Management Interfaces

Run:

show ip interface brief

Expected:

Vlan10    10.0.16.x    up    up

Test the gateway:

ping 10.0.16.1

Test switch-to-switch connectivity:

ping 10.0.16.3
ping 10.0.16.4
ping 10.0.16.5
ping 10.0.16.6
ping 10.0.16.7
49. SVI Up/Up Requirements

For:

interface vlan 10

to become:

up/up

the following conditions must be satisfied:

VLAN 10 must exist.
The SVI must not be administratively shut down.
At least one operational Layer 2 interface must be associated with VLAN 10.
An active trunk carrying VLAN 10 can satisfy the active VLAN requirement.

If the conditions are not met, the SVI may show:

down/down

or:

up/down
50. Spanning Tree Protocol

The Fallout Shelter topology contains redundant Layer 2 connections.

Redundant connections improve availability.

However, redundant Layer 2 paths can create switching loops.

STP stands for:

Spanning Tree Protocol

STP prevents Layer 2 loops.

Verify:

show spanning-tree

Some interfaces may be:

FWD

while other interfaces may be:

BLK

This is normal.

A blocked port is not necessarily broken.

STP may intentionally block the interface to prevent a Layer 2 loop.

51. Spanning Tree Was Not Yet Optimized

The lesson review states:

Spanning Tree is not setup yet

More precisely:

STP has not yet been intentionally configured or optimized.

Cisco switches run STP by default.

Therefore, STP is already protecting the topology.

Future lessons can optimize:

Root bridge placement
VLAN-specific STP paths
Redundant path selection
EtherChannel
Layer 2 traffic flow
52. Final PC Connectivity Testing
MGMT PC
ping 10.0.16.1
INTERNAL PC
ping 10.0.16.129
VIDEO PC
ping 10.0.17.1
GUEST PC
ping 10.0.17.129
53. Test Inter-VLAN Routing

Actual DHCP-assigned addresses:

MGMT       10.0.16.11

INTERNAL   10.0.16.139

VIDEO      10.0.17.11

GUEST      10.0.17.139

From the MGMT PC:

ping 10.0.16.139
ping 10.0.17.11
ping 10.0.17.139

The pings should succeed because ROAS provides inter-VLAN routing and no ACLs have been configured to restrict traffic.

54. Final Router Verification Commands
show ip interface brief
show ip route
show ip dhcp pool
show ip dhcp binding
show running-config
55. Final Switch Verification Commands
show cdp neighbors
show vtp status
show vlan brief
show interfaces trunk
show interfaces switchport
show ip interface brief
show spanning-tree
show running-config
56. Complete Implementation Flow
Analyze RFP
     |
     v
Identify Required Network Segments
     |
     v
Start with 10.0.16.0/23
     |
     v
Subnet into Four /25 Networks
     |
     v
Design VLANs
     |
     v
Configure VTP
     |
     v
Configure DTP
     |
     v
Establish Trunk Links
     |
     v
Create VLANs on VTP Server
     |
     v
Verify VLAN Replication
     |
     v
Configure Access Ports
     |
     v
Configure Router-Facing Trunk
     |
     v
Configure Router-on-a-Stick
     |
     v
Verify Connected and Local Routes
     |
     v
Configure DHCP Exclusions
     |
     v
Configure DHCP Pools
     |
     v
Configure PCs for DHCP
     |
     v
Verify DHCP Bindings
     |
     v
Test Inter-VLAN Routing
     |
     v
Configure Switch Management SVIs
     |
     v
Configure Switch Default Gateways
     |
     v
Verify STP
     |
     v
Perform Final Connectivity Testing
     |
     v
Save Configurations
57. Final Save

Save the configuration on every router and switch:

copy running-config startup-config

Alternative:

write memory

Devices to save:

FO-RT01
FO-SW01
FO-SW02
FO-SW03
FO-SW04
FO-SW05
FO-SW06
Key Takeaways
VLAN implementation is a network redesign, not simply VLAN creation.
Every VLAN creates a separate Layer 2 broadcast domain.
Every VLAN should have its own IP subnet.
Every VLAN requires a Layer 3 gateway to communicate outside its subnet.
The original 10.0.16.0/23 network was divided into four /25 networks.
VLAN 10 was assigned to MGMT traffic.
VLAN 20 was assigned to INTERNAL traffic.
VLAN 30 was assigned to VIDEO traffic.
VLAN 40 was assigned to GUEST traffic.
VTP distributes VLAN database information.
VTP does not create trunk links.
VTP requires compatible domain, version, and password configuration.
DTP negotiates trunk formation between Cisco switches.
Dynamic auto waits for trunk negotiation.
Dynamic desirable actively attempts to form a trunk.
Default commands may not appear in show running-config.
show interfaces switchport provides detailed administrative and operational switchport information.
802.1Q tags VLAN traffic across trunk links.
Access ports carry traffic for one VLAN.
Trunk ports carry traffic for multiple VLANs.
Router interfaces are administratively shut down by default.
Router-on-a-Stick provides inter-VLAN routing.
The physical ROAS router interface does not require an IP address.
Router subinterfaces use 802.1Q encapsulation.
Each router subinterface acts as the default gateway for one VLAN.
C represents a directly connected network route.
L represents the router's own local interface address.
IOS automatically creates connected and local routes.
DHCP automatically provides client IP configuration.
DHCP exclusions protect infrastructure addresses from dynamic assignment.
DNS configuration does not create Internet connectivity.
Pinging an IP address does not use DNS.
Layer 2 switches do not need IP addresses to forward Ethernet frames.
Switches need management IP addresses for remote administration.
Management IP addresses are configured on SVIs.
The Fallout Shelter switches use VLAN 10 as the management VLAN.
ip default-gateway allows Layer 2 switches to reach remote networks.
STP prevents Layer 2 switching loops.
STP can intentionally block redundant links.
A blocked STP port is not necessarily a failed port.
STP was operating by default but had not yet been intentionally optimized.
Verification commands should be used after every major implementation phase.
Troubleshooting is an important part of the VLAN implementation process.
Always save the running configuration after completing and verifying the implementation.