```
! ============================================================
! Toronto-East-Edge-R1 — Edge Router Right
! ============================================================

enable
configure terminal

! --- Hostname ---
hostname Toronto-East-Edge-R1

! --- Loopback (stable management IP for SolarWinds) ---
interface Loopback0
 description MGMT-LOOPBACK
 ip address 3.3.3.3 255.255.255.255
 no shutdown

! --- fa0/0 — Uplink to Toronto-East-SW1 e0/1 ---
interface FastEthernet0/0
 description UPLINK-TO-Toronto-East-Edge-R1
 ip address 192.168.40.2 255.255.255.0
 no shutdown

! --- Default route — send all traffic to Toronto-East-SW1 → Toronto-CR1 ---
ip route 0.0.0.0 0.0.0.0 192.168.40.1

! --- SNMP Configuration ---
access-list 10 permit any
snmp-server community exdxxssdd RO
snmp-server location "Toronto East Edge"
snmp-server contact leo@homelab.local
snmp-server enable traps
snmp-server host 10.1.20.20 version 2c exdxxssdd

! --- Save configuration ---
end
write memory
```