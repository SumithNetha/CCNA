# Packet Tracer Lab – Dynamic NAT

## Topology

PC1 -----SW ----- R1 ----- ISP
          |        |
          |        |
          |        R2
         PC2 

LAN:
192.168.1.0/24

Public Pool:
216.0.5.50–216.0.5.100

---

## Tasks

1. Configure router interfaces
2. Configure ACL for LAN
3. Create NAT Pool
4. Enable Dynamic NAT
5. Test connectivity
6. Verify translation table

---

## Expected Output

show ip nat translations

Inside local        Inside global

192.168.1.50       216.0.5.50
192.168.1.51       216.0.5.51

---

## Challenge

Add another subnet:

192.168.2.0/24

Update ACL and verify both networks use NAT.
