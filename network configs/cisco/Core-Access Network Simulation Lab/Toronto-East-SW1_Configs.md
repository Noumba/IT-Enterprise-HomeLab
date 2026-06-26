```
! ============================================================
! Toronto-East-SW1 — Layer 2 Access Switch (Right)
! ============================================================

enable
configure terminal

! --- Hostname ---
hostname Toronto-East-SW1

! --- VLAN1 SVI for management ---
interface Vlan1
 description MGMT-SVI
 ip address 192.168.20.2 255.255.255.0
 no shutdown

! --- Default gateway pointing to Toronto-CR1 fa2/0 ---
ip default-gateway 192.168.20.1

! --- e0/0 — Uplink to Toronto-CR1 fa2/0 ---
interface Ethernet0/0
 description UPLINK-TO-Toronto-CR1-FA2/0
 switchport mode access
 switchport access vlan 1
 no shutdown

! --- e0/1 — Downlink to 37258 fa0/0 ---
interface Ethernet0/1
 description DOWNLINK-TO-37258
 switchport mode access
 switchport access vlan 1
 no shutdown

! --- Enable ip routing ---

ip routing
ip route 0.0.0.0 0.0.0.0 192.168.20.1

! --- SNMP Configuration ---
access-list 10 permit 10.1.20.20
access-list 10 permit 10.1.50.10
snmp-server community DaVince-SNMP-RO RO 10
snmp-server location DaVince-Technologies-HomeLab-EVE-NG
snmp-server contact leo@homelab.local
snmp-server enable traps
snmp-server host 10.1.20.20 version 2c DaVince-SNMP-RO

! --- Save configuration ---
end
write memory
```