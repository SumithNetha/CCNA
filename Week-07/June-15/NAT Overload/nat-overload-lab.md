

# NAT Overload Lab

Topology

PC1 ----
SW ---- Router ---- ISP
PC2 ----/

LAN:
192.168.1.0/24

WAN:
216.0.5.0/30

Tasks

[✓] Configure interfaces

[✓] Configure ACL

[✓] Configure PAT

[✓] Test connectivity

[✓] Verify NAT

Commands

show ip nat translations

show ip nat statistics

Expected Result

Multiple inside devices share one public IP using unique ports.

Screenshots

1_topology.png

2_show_ip_nat_translations.png

3_show_ip_nat_statistics.png

4_successful_ping.png
