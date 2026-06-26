```
! ============================================================
! Toronto West Edge R1 — Edge Router Left
! ============================================================

enable
configure terminal

! --- Hostname ---
hostname Toronto-West-Edge-R1

! --- Loopback (stable management IP for SolarWinds) ---
interface Loopback0
 description MGMT-LOOPBACK
 ip address 2.2.2.2 255.255.255.255
 no shutdown

! --- fa0/0 — Uplink to Toronto-West-SW1 e0/1 ---
interface FastEthernet0/0
 description UPLINK-TO-Toronto-West-SW1
 ip address 192.168.30.2 255.255.255.0
 no shutdown

! --- Default route — send all traffic to SW1 → Toronto-CR1 ---
ip route 0.0.0.0 0.0.0.0 192.168.30.1

! --- SNMP Configuration ---
access-list 10 permit 10.1.20.20
access-list 10 permit 10.1.50.10
snmp-server community exdxxssdd RO
snmp-server location "Toronto West"
snmp-server contact leo@homelab.local
snmp-server enable traps
snmp-server host 10.1.20.20 version 2c exdxxssdd

! --- Save configuration ---
end
write memory
```