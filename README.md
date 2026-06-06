# Home Lab Architecture Document

> **Project:** VMware ESXi + pfSense + VLANs Home Lab  
> **Version:** 1.0  
> **Date:** 2026-06-04  
> **Author:** LEONARD NOUMBA  
> **Status:** Active  

---

## Table of Contents

1. [Project Overview](#1-project-overview)
2. [Physical Topology](#2-physical-topology)
3. [Logical Topology](#3-logical-topology)
4. [Current Status](#4-current-status)
5. [Projects](#5-projects)
6. [Documentation](#6-documentation)
---

## 1. Project Overview

### Purpose
This home lab is designed to simulate an enterprise-grade network environment using VMware ESXi as the hypervisor platform and pfSense as the software firewall/router. The architecture implements network segmentation via VLANs to isolate traffic between management, infrastructure, DMZ, and lab/testing zones.

### Goals
- Practice and demonstrate Layer 2/Layer 3 network segmentation using industry-standard VLAN design
- Validate firewall zone policies using pfSense inter-VLAN routing and NAT
- Provide isolated environments for infrastructure services, DMZ-hosted workloads, and lab experimentation
- Serve as a reproducible platform for certification lab work (CCNA, CompTIA Network+)

### Scope
This document covers:
- Physical device inventory
- Logical VLAN and IP addressing design
- pfSense firewall architecture and VLAN interface configuration
- VMware ESXi virtual switching (vSwitch, Port Groups)
- Virtual machine placement and network attachment
- Traffic flow between network zones

Out of scope: cloud integration, off-site VPN tunnels, IDS/IPS configuration (future phases).

---

## 2. Architecture Diagram
![Home Lab Architecture Diagram](diagrams-images/architecture/homelab%20architecture-1.png)

### Physical Device Inventory

| Hostname | Role | Make / Model | Management IP | Interface | Uplink To | Notes |
|---|---|---|---|---|---|---|
| ISP-ROUTER | ISP Gateway | [REQUIRED: ISP modem/router model] | 192.168.1.1/24 | WAN port | Internet | Provides DHCP to pfSense WAN |
| PFSENSE-FW | Firewall / Router | pfSense VM on ESXi | 10.1.1.1/24 | vmnic0 (WAN), Esxi Port Group Trunk (LAN) | ISP-ROUTER / ESXi vSwitch | VLAN-aware; LAN is trunk VLAN 4095 |
| ESXI-HOST-01 | Hypervisor | [REQUIRED: HP EliteDesk 800 G4 or other] | [REQUIRED: ESXi MGMT IP] | vmnic0 (uplink) | pfSense LAN (trunk) | Hosts all VMs; single Port Group trunk |

### Physical Connections

```
INTERNET
    │
    │ (Coax / DSL / Fibre)
    ▼
ISP ROUTER (192.168.1.1/24)
    │
    │ Ethernet — 192.168.1.0/24 (WAN segment)
    ▼
pfSense — vmnic0 (WAN: 192.168.1.50/24)
pfSense — ESXi VLAN PG (LAN TRUNK: VLAN 4095 → all VLANs)
    │
    │ 802.1Q Trunk (VLAN 4095 carries all VLANs)
    ▼
ESXi HOST — ESXi VLAN PG
    │
    └── vSwitch-LAN (VMware Standard vSwitch)
            ├── PG-TRUNK   (VLAN 4095)
            ├── PG-MGMT    (VLAN 10)
            ├── PG-INFRA   (VLAN 20)
            ├── PG-DMZ     (VLAN 30)
            └── PG-LAB     (VLAN 50)
```

---

## 3. Logical Topology

### Architecture Summary

The logical design follows a **router-on-a-stick** model. pfSense terminates all VLAN subinterfaces on a single physical LAN trunk (`ESXi PG Trunk`). Each VLAN maps to a dedicated `/24` subnet, and pfSense acts as the default gateway for every segment.

The ESXi host carries the trunk upstream via a single vmnic. VMware Standard vSwitch (`vSwitch-LAN`) distributes traffic to Port Groups, each tagged with the appropriate VLAN ID. VMs connect to their respective Port Group and receive DHCP from pfSense.

### Layer 2 Boundary
All switching is handled by the VMware vSwitch inside the ESXi host. There is no physical managed switch in this design. VLAN tagging is enforced at the Port Group level.

### Layer 3 Boundary
pfSense is the sole Layer 3 device. Inter-VLAN routing, NAT, and firewall policy enforcement all occur on pfSense.


## 4. Current Status

✅ ESXi Deployment
✅ pfSense Deployment
✅ VLAN Segmentation


## 5. Projects
⏳


## 6. Documentation
 - [Home Lab Design Documentation](docs/architecture/Full_Enterprise_HomeLab_Network_Design%20document.pdf) 


## Appendix A — Skills Demonstrated

| Lab Component | Certification Domain |
|---|---|
| VLAN design and 802.1Q trunking | CCNA — Network Access (20%) |
| pfSense inter-VLAN routing (router-on-a-stick) | CCNA — IP Connectivity (25%) |
| NAT / PAT configuration | CCNA — IP Services (10%) |
| Firewall zone policy design | CompTIA Network+ — Network Security (17%) |
| VMware ESXi vSwitch and Port Group configuration | VMware VCP / CCNA — Virtualisation |
| IP addressing and subnetting (/24 per VLAN) | CCNA — IP Connectivity |
| Network segmentation and DMZ design | CompTIA Security+ — Architecture and Design |

