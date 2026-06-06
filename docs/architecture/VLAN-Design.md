# VLAN Design

## Overview

Virtual Local Area Networks (VLANs) are used within the Enterprise Hybrid Home Lab to logically segment network traffic into distinct security and operational zones. This design follows enterprise networking best practices by separating management, infrastructure, cloud, DMZ, and lab resources into dedicated broadcast domains.

The primary objectives of VLAN segmentation are to improve security, simplify administration, reduce broadcast traffic, and provide clear separation between workloads with different operational and security requirements.

By implementing VLAN-based segmentation, the environment more closely reflects modern enterprise network architectures while providing a scalable foundation for monitoring, automation, virtualization, and hybrid cloud integration.

---

## Purpose

The purpose of this document is to define the VLAN architecture used throughout the home lab environment and provide a centralized reference for network segmentation, subnet allocation, gateway assignments, and VLAN usage.

This document supports:

- Network deployment and administration
- Firewall policy implementation
- Routing configuration
- Monitoring and observability
- Security enforcement
- Future infrastructure expansion

---

## Design Objectives

The VLAN architecture has been designed to achieve the following objectives:

- Isolate critical infrastructure from user and testing environments.
- Support secure administrative access to systems and services.
- Separate production-like workloads from experimental lab environments.
- Enable granular firewall policies and traffic inspection.
- Simplify troubleshooting and operational management.
- Provide a scalable structure for future services and integrations.

---

## VLAN Architecture

The environment uses a firewall-on-a-stick design where pfSense performs inter-VLAN routing for all internal networks.

```text
Internet
    |
ISP Router
    |
pfSense
    |
PG-TRUNK (VLAN 4095)
    |
vSwitch-LAN
    |
├── VLAN 10 - Management
├── VLAN 20 - Infrastructure
├── VLAN 30 - DMZ
├── VLAN 40 - Cloud
└── VLAN 50 - Lab
```

The trunk connection between pfSense and ESXi carries all VLAN traffic and allows pfSense to enforce routing and security policies between network segments.

---

### VLAN Table

| VLAN ID | Name | Subnet | Gateway | Purpose | Port Group | pfSense Interface |
|---|---|---|---|---|---|---|
| 10 | MGMT | 10.1.10.0/24 | 10.1.10.1 | Management network — pfSense GUI, ESXi DCUI, admin access | PG-MGMT | LAN1.10 |
| 20 | INFRA | 10.1.20.0/24 | 10.1.20.1 | Infrastructure services — DNS, NTP, monitoring, file servers | PG-INFRA | LAN1.20 |
| 30 | DMZ | 10.1.30.0/24 | 10.1.30.1 | Demilitarised zone — publicly reachable or internet-facing workloads | PG-DMZ | LAN1.30 |
| 50 | LAB | 10.1.50.0/24 | 10.1.50.1 | Lab and testing — experimental VMs, routing labs, CCNA practice | PG-LAB | LAN1.50 |
| 4095 | TRUNK | N/A | N/A | Trunk port group carrying all VLANs to pfSense | PG-TRUNK | LAN1 (parent) |

### Segmentation Rationale

| Zone | Trust Level | Inter-zone Access Policy |
|---|---|---|
| MGMT (VLAN10) | Highest | Initiates to all zones; no inbound from INFRA/DMZ/LAB |
| INFRA (VLAN20) | High | Can reach MGMT for management services; cannot reach DMZ/LAB uninitiated |
| DMZ (VLAN30) | Low | Isolated; inbound from WAN permitted on specific ports only; no access to MGMT/INFRA |
| LAB (VLAN50) | Lowest | Fully isolated for experimentation; no access to production VLANs; outbound internet via NAT |

---

## VLAN Descriptions

### VLAN 10 – Management

The Management VLAN provides secure administrative access to network devices, servers, virtualization platforms, and monitoring systems.

**Typical systems include:**

- ESXi Management
- pfSense Administration
- Administrative Workstations
- Jump Hosts
- Monitoring Administration Interfaces

This VLAN should have the most restrictive access controls and should only be accessible by authorized administrators.

---

### VLAN 20 – Infrastructure

The Infrastructure VLAN hosts core services required for the operation of the environment.

**Typical systems include:**

- Active Directory
- DNS
- DHCP
- NTP
- Certificate Services
- File Services
- Network Monitoring Solutions

This VLAN forms the operational backbone of the lab environment.

---

### VLAN 30 – DMZ

The DMZ VLAN hosts services that may require access from external networks while remaining isolated from critical infrastructure systems.

**Typical systems include:**

- Web Servers
- Reverse Proxies
- Public Applications
- Testing Port Publishing

Access from the DMZ to internal networks should be explicitly controlled through firewall policies.

---

### VLAN 40 – Cloud

The Cloud VLAN is dedicated to hybrid cloud services and integrations.

**Typical systems include:**

- Azure Connectors
- AWS Integrations
- Cloud Monitoring Agents
- Hybrid Networking Components

This segment allows testing of cloud connectivity and identity synchronization scenarios.

---

### VLAN 50 – Lab

The Lab VLAN provides an isolated environment for experimentation, testing, and network simulations.

**Typical systems include:**

- EVE-NG
- Cisco Virtual Routers
- Juniper Virtual Routers
- MikroTik CHR
- Security Testing Appliances

This VLAN enables practical hands-on experience without impacting infrastructure services.

---

## Inter-VLAN Routing

All inter-VLAN routing is performed by pfSense.

Each VLAN is assigned a dedicated Layer 3 interface on pfSense with its own gateway address. Traffic between VLANs is inspected and controlled through firewall policies before being forwarded to its destination.

This design provides:

- Centralized traffic control
- Improved visibility
- Granular security enforcement
- Simplified troubleshooting

---

## Security Considerations

The VLAN design supports a defense-in-depth security strategy through logical network segmentation.

### Key Principles

- Administrative traffic remains isolated from user workloads.
- Infrastructure services are protected from experimental environments.
- Public-facing services are separated from internal resources.
- Cloud integrations are isolated from production-like infrastructure.
- Firewall policies enforce least-privilege communication between segments.

---

## Future Expansion

The VLAN architecture has been designed to accommodate future growth.

### Planned VLANs

| VLAN ID | Purpose |
|----------|----------|
| 60 | Monitoring |
| 70 | Security Operations (SOC) |
| 80 | Storage |
| 90 | Guest Network |
| 100 | Kubernetes |

Additional VLANs can be integrated into the existing trunk architecture without requiring significant redesign.

---

## Summary

The VLAN design establishes a secure, scalable, and enterprise-aligned network segmentation strategy for the home lab environment. By separating management, infrastructure, cloud, DMZ, and lab resources into dedicated VLANs, the environment provides realistic operational experience while supporting future learning objectives in networking, automation, monitoring, security, and cloud engineering.