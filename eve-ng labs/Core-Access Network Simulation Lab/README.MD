# EVE-NG Enterprise Topology — Full Device Configuration

## Document Control

| Field | Value |
|---|---|
| Document ID | EVE-NG01 |
| Title | Core-Access Network Simulation Lab |
| Version | 1.0 |
| Author | Leonard Noumba |
| Environment | Home Lab — EVE-NG 10.1.50.10 — VLAN50 LAB |

---

Problem Statement
## Objective: Build and Monitor a Hierarchical Campus Network Topology in EVE-NG

- You are tasked with designing and configuring a multi-device campus network simulation using EVE-NG Community Edition, integrated into a real multi-VLAN home lab environment running on VMware ESXi with pfSense as the core firewall and router.
- The topology consists of one core gateway router (Toronto-CR1), two access layer switches (Toronto-West-SW1, Toronto-East-SW2), and two edge routers, interconnected in a hierarchical hub-and-spoke design.
- Toronto-CR1 must be reachable from the real lab network via fa0/0 (10.1.50.100/24) connected to the EVE-NG Cloud0 bridge on VLAN50, with a default route pointing to pfSense (10.1.50.1).
- Toronto-West-SW1 and Toronto-East-SW2 must be configured with IP routing enabled, VLAN1 SVIs (192.168.10.2/24 and 192.168.20.2/24), and static default routes pointing to Toronto-CR1.
- Edge R1 and Edge R2 must be reachable via fa0/0 (192.168.10.3/24 and 192.168.20.3/24) and stable loopback addresses (2.2.2.2/32 and 3.3.3.3/32) used as primary monitoring endpoints.
- All five devices must be configured with SNMPv2c (community string `SNMP-RO`), restricted to the SolarWinds polling engine at MON01 (10.1.20.20), with SNMP traps enabled toward MON01.
- The EVE-NG host (10.1.50.10) must have persistent static routes for all internal topology subnets via Toronto-CR1, and pfSense must have corresponding static routes to ensure full bidirectional reachability from all lab VLANs.
- Upon completion, all five devices must be successfully onboarded into SolarWinds Observability Self-Hosted with interface traffic, CPU, and memory polling confirmed active.


## 1. Topology Overview

![Network Topology](../Core-Access%20Network%20Simulation%20Lab/Images/network%20topology.JPG)

---

## 2. Device Configurations

---

### 2.1 — TorontoCR1 (7206VXR Core Gateway Router)

> This router is the most critical device — it connects the simulated topology to the real lab network via fa0/0 and routes between all internal segments.

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

---

### 2.2 — Toronto-West-SW1  (Layer 2 Switch — Left)

> SW1 is a Layer 2 IOL switch. It does not route — it switches frames between the 7206VXR and 37257. The SVI (VLAN1) gives it a management IP so SolarWinds can poll it.

```
! ============================================================
! Toronto-West-SW1 — Layer 2 Access Switch (Left)
! ============================================================

enable
configure terminal

! --- Hostname ---
hostname Toronto-West-SW1

! --- VLAN1 SVI for management ---
interface Vlan1
 description MGMT-SVI
 ip address 192.168.10.2 255.255.255.0
 no shutdown

! --- Default gateway pointing to Toronto-CR1 fa1/0 ---
ip default-gateway 192.168.10.1

! --- e0/0 — Uplink to Toronto-CR1 fa1/0 ---
interface Ethernet0/0
 description UPLINK-TO-7206VXR-FA1/0
 switchport mode access
 switchport access vlan 1
 no shutdown

! --- e0/1 — Downlink to 37257 fa0/0 ---
interface Ethernet0/1
 description DOWNLINK-TO-37257
 switchport mode access
 switchport access vlan 1
 no shutdown

! --- Enable ip routing ---

ip routing
ip route 0.0.0.0 0.0.0.0 192.168.10.1


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

---

### 2.3 — Toronto-East-SW1 (Layer 2 Switch — Right)

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

---

### 2.4 — Toronto West Edge R1 (Edge Router — Left)

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

---

### 2.5 — Toronto-East-Edge-R1 (Edge Router — Right)

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

---

## 3. Missing Gateway Configuration

> **Important:** 37257 and 37258 use `192.168.30.1` and `192.168.40.1` as their default gateways respectively — but these IPs currently have no device assigned to them. They represent the SW1/SW2 side of the link toward the edge routers.

Since SW1 and SW2 are **Layer 2 switches** (no routing), we have two options:

### Option A — Add SVIs on SW1/SW2 for the 37257/37258 segments (Implemented)

Add these to SW1:
```
interface Vlan30
 ip address 192.168.30.1 255.255.255.0
 no shutdown
```

Add these to SW2:
```
interface Vlan40
 ip address 192.168.40.1 255.255.255.0
 no shutdown
```

Then put the SW1 e0/1 and SW2 e0/1 ports into the correct VLANs:

**SW1:**
```
vlan 30
 name EDGE-LEFT
interface Ethernet0/1
 switchport access vlan 30
```

**SW2:**
```
vlan 40
 name EDGE-RIGHT
interface Ethernet0/1
 switchport access vlan 40
```

### Option B — Keep all devices on VLAN1 (Simpler for lab)

Assign 37257 and 37258 IPs within the same subnet as their upstream switch and point default gateway directly to 7206VXR:

**37257 fa0/0:** `192.168.10.3/24` — gateway `192.168.10.1`
**37258 fa0/0:** `192.168.20.3/24` — gateway `192.168.20.1`

---

## 4. EVE-NG Host — Static Routes

The EVE-NG host needs return routes to reach devices inside the topology. Without these, pings from EVE-NG host to any internal subnet fail even when the devices can reach the host.

![EVE-NG static routes](../Core-Access%20Network%20Simulation%20Lab/Images/EVE-NG%20static%20routes.JPG)
---

## 5. pfSense — Static Routes Required

For MON01 (10.1.20.20) to reach the EVE-NG devices, pfSense needs static routes for the internal EVE-NG subnets:

**pfSense → System → Routing → Static Routes → Add:**

![EVE-NG static routes](../Core-Access%20Network%20Simulation%20Lab/Images/pfSense%20static%20routes.JPG)
---

## 6. Verification

### From EVE-NG Host (10.1.50.10)

```bash
# Ping 7206VXR uplink
ping 10.1.50.100

# Ping SW1 management
ping 192.168.10.2

# Ping SW2 management
ping 192.168.20.2
```
![EVE-NG ping test](../Core-Access%20Network%20Simulation%20Lab/Images/eve-ng%20ping%20test.JPG)

### From Toronto-CR1 Console

```
ping 10.1.50.1          ! pfSense gateway
ping 10.1.20.20         ! MON01 SolarWinds
```
![CR1 ping test](../Core-Access%20Network%20Simulation%20Lab/Images/CR1%20ping%20test.JPG)
---

## 6. SolarWinds Monitoring

Devices successfully onboarded into SolarWinds for monitoring.

![SolarWinds Monitoring](../Core-Access%20Network%20Simulation%20Lab/Images/monitoring.JPG)

## 7. Quick Reference

| Device | Management IP | Loopback | Gateway |
|---|---|---|---|
| Toronto-CR1 | 10.1.50.100 | 1.1.1.1 | 10.1.50.1 (pfSense) |
| Toronto-West-SW1 | 192.168.10.2 | — | 192.168.10.1 |
| Toronto-East-SW1 | 192.168.20.2 | — | 192.168.20.1 |
| Toronto-West-Edge-R1 | 192.168.30.2 | 2.2.2.2 | 192.168.30.1 |
| Toronto-East-Edge-R1 | 192.168.40.2 | 3.3.3.3 | 192.168.40.1 |
