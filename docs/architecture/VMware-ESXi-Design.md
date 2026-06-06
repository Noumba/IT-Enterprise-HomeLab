# VMware Virtualization Design Architecture

## 1. Introduction

This document outlines the design of a VMware-based virtualization environment intended for a home lab / enterprise simulation setup. The design focuses on scalability, high availability, resource optimization, and segregation of workloads for networking, security, and system administration practice.

The goal is to build a structured virtual infrastructure that mirrors real-world enterprise environments while remaining within limited hardware constraints.

---

## 2. Design Objectives

- Build a scalable and modular virtualization platform
- Simulate enterprise-grade network and server infrastructure
- Enable hands-on practice with VMware ESXi, vCenter, and virtual networking
- Optimize use of limited physical resources (CPU, RAM, storage)
- Provide isolation between management, production, and lab environments
- Support future expansion into cloud and DevOps environments

---

## 3. Physical Infrastructure Overview

### Host Machine (ESXi Hypervisor)

- **Hypervisor:** VMware ESXi 8.x
- **CPU:** 6 vCPU cores available
- **Memory:** 32 GB RAM
- **Storage:** 319 GB SSD
- **Network:** 1–2 physical NICs (bridged or VLAN-capable switch recommended)

---

## 4. Virtualization Architecture Design

### 4.1 ESXi Host Layer
The ESXi host acts as the bare-metal hypervisor managing all virtual machines and virtual networking components.

### ESXi Host

| Attribute | Value |
|---|---|
| Hypervisor | VMware ESXi 8.0.3 |
| Hardware | [REQUIRED: HP EliteDesk 800 G4 — confirm CPU, RAM, storage] |
| Management IP | [] |
| vSwitch | vSwitch-LAN (VMware Standard vSwitch) |
| Physical uplink | vmnic0 → pfSense vtnet1 (802.1Q trunk) |

### VMware Standard vSwitch — Port Group Configuration

| Port Group | VLAN ID | MTU | Promiscuous Mode | Notes |
|---|---|---|---|---|
| PG-TRUNK | 4095 | 1500 | Yes | Carries all VLANs; connected to pfSense LAN |
| PG-MGMT | 10 | 1500 | No | Management VMs |
| PG-INFRA | 20 | 1500 | No | Infrastructure VMs |
| PG-DMZ | 30 | 1500 | No | DMZ workloads |
| PG-LAB | 50 | 1500 | No | Lab / test VMs |

---

## 6. Storage Design

- **Datastore Type:** Local SSD datastore
- **Allocation Strategy:**
  - Thin provisioning enabled
  - Priority storage for DC and monitoring servers
  - ISO repository for OS installations
- **Backup Strategy:**
  - Manual snapshots for lab states
  - Optional export to external storage or NAS

---

## 8. High Availability & Resilience (Lab Simulation)

While full HA is not available in a single-host setup, the following simulations are included:

- VM snapshots for rollback testing
- Simulated failover scenarios (manual VM migration)
- Redundant service design (DNS, DHCP backup roles)
- Firewall rule redundancy testing

---

## 9. Security Design

- Network segmentation via VLANs and pfSense firewall
- Least privilege access model for services
- Isolated DMZ for external simulation
- Logging and monitoring via centralized server
- Regular snapshot and restore testing

---

## 10. Monitoring & Management

- VMware Host Client / vCenter (optional deployment)
- SolarWinds or similar monitoring platform
- Syslog collection for network devices
- Performance tracking (CPU, RAM, disk usage)

---

## 11. Future Expansion Plan

- Add second ESXi host for cluster simulation
- Introduce vCenter Server for centralized management
- Integrate cloud environment (AWS or Azure hybrid lab)
- Add Kubernetes / Docker environment for DevOps practice
- Implement automated provisioning (Ansible / Terraform)

---

## 12. Conclusion

This VMware design provides a structured and scalable foundation for building a realistic enterprise lab environment. It balances resource constraints with practical learning objectives, allowing progressive development into advanced networking, system administration, and cloud engineering skills.