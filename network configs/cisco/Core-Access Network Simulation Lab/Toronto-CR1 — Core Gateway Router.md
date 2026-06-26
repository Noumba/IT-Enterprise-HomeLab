```
! ============================================================
! Toronto-CR1 — Core Gateway Router
! ============================================================

enable
configure terminal

! --- Hostname ---
hostname Toronto-CR1

! --- Loopback (stable management IP for SolarWinds) ---
interface Loopback0
 description MGMT-LOOPBACK
 ip address 1.1.1.1 255.255.255.255
 no shutdown

! --- fa0/0 — Uplink to EVE-NG Cloud (real lab VLAN50) ---
interface FastEthernet0/0
 description UPLINK-TO-REAL-LAB-VLAN50
 ip address 10.1.50.100 255.255.255.0
 no shutdown

! --- fa1/0 — Link to SW1 ---
interface FastEthernet1/0
 description LINK-TO-Toronto-West-SW1
 ip address 192.168.10.1 255.255.255.0
 no shutdown

! --- fa2/0 — Link to SW2 ---
interface FastEthernet2/0
 description LINK-TO-Toronto-East-SW1
 ip address 192.168.20.1 255.255.255.0
 no shutdown

! --- Default route — send all unknown traffic to pfSense ---
ip route 0.0.0.0 0.0.0.0 10.1.50.1

! --- Static routes to reach 37257 behind SW1 ---
ip route 192.168.30.0 255.255.255.0 192.168.10.2
ip route 2.2.2.2 255.255.255.255 192.168.10.2

! --- Static routes to reach 37258 behind SW2 ---
ip route 192.168.40.0 255.255.255.0 192.168.20.2
ip route 3.3.3.3 255.255.255.255 192.168.20.2

! --- SNMP Configuration ---
access-list 10 permit any
snmp-server community exdxxssdd RO 10
snmp-server location DaVince-Technologies-HomeLab-EVE-NG
snmp-server contact leo@homelab.local
snmp-server enable traps
snmp-server host 10.1.20.20 version 2c exdxxssdd

! --- Save configuration ---
end
write memory
```