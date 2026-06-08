# Phase 02 - Core Infrastructure Deployment

## Overview

This document outlines the deployment of the core infrastructure services that form the foundation of the Enterprise Hybrid Home Lab environment.

The objective of this phase is to develop a server and service deployment structure (Order) and establish the essential services required on each deployed server.

All subsequent infrastructure, monitoring, automation, and networking components depend on the successful completion of this phase.

---

# Objectives

The goals of this phase are to:

- Deploy server infrastructure
- Deploy centralized identity services.
- Implement enterprise DNS and DHCP services.
- Establish management access.
- Prepare the environment for network simulations, monitoring and cloud integrations.

---

# Deployment Order

The deployment sequence follows enterprise infrastructure implementation practices.

```text
1. pfSense
      ↓
2. DC01
      ↓
3. DNS Configuration
      ↓
4. DHCP Configuration
      ↓
5. NTP Configuration
      ↓
6. MGMT01
      ↓
7. Domain Join
      ↓
8. MON01
      ↓
9. SolarWinds Deployment
      ↓
10. AUTO01
      ↓
11. EVE01
      ↓
12. Cloud Integration Services
```

---

# Infrastructure Services

## DC01 - Domain Controller

### Purpose

DC01 provides centralized authentication, DNS resolution, DHCP services, and time synchronization for the environment.

### Configuration

| Setting | Value |
|----------|----------|
| Hostname | DC01 |
| VLAN | VLAN20-INFRA |
| Port Group | PG-INFRA |
| IP Address | 10.1.20.10 |
| Subnet Mask | 255.255.255.0 |
| Gateway | 10.1.20.1 |
| Preferred DNS | 127.0.0.1 |
| Operating System | Windows Server 2022 |

### Resource Allocation

| Resource | Allocation |
|----------|----------|
| vCPU | 1 |
| RAM | 1 GB |
| Disk | 10 GB |

### Roles Installed

- Active Directory Domain Services
- DNS Server
- DHCP Server
- NTP Services

### Domain Name

```text
corp.homelab
```

---

## DNS Configuration

### Purpose

Provides centralized name resolution for all infrastructure services and client devices.

### DNS Forwarders

```text
1.1.1.1
8.8.8.8
```

### Internal DNS Zone

```text
corp.homelab
```

### Example Records

| Hostname | IP Address |
|----------|----------|
| dc01 | 10.1.20.10 |
| mon01 | 10.1.20.20 |
| auto01 | 10.1.20.30 |
| eve01 | 10.1.50.10 |
| pfsense | 10.1.10.1 |

---

## DHCP Configuration

### VLAN10 - Management

```text
Network: 10.1.10.0/24
Gateway: 10.1.10.1
Range: 10.1.10.100 - 10.1.10.199
```

### VLAN50 - Lab

```text
Network: 10.1.50.0/24
Gateway: 10.1.50.1
Range: 10.1.50.100 - 10.1.50.250
```

### Infrastructure VLAN

Static IP assignments only.

---

## Time Synchronization

### Time Source

```text
pool.ntp.org
```

### NTP Hierarchy

```text
Internet NTP
      ↓
DC01
      ↓
Domain Members
```

---

# MGMT01 - Management Workstation

## Purpose

Provides a dedicated workstation for infrastructure administration.

### Configuration

| Setting | Value |
|----------|----------|
| Hostname | MGMT01 |
| VLAN | VLAN10-MGMT |
| Port Group | PG-MGMT |
| IP Address | 10.1.10.10 |
| DNS | 10.1.20.10 |
| Operating System | Windows Server 2022 |

### Resource Allocation

| Resource | Allocation |
|----------|----------|
| vCPU | 1 |
| RAM | 1 GB |
| Disk | 25 GB |

### Responsibilities

- ESXi Administration
- pfSense Administration
- Active Directory Administration
- SolarWinds Administration
- Remote Management

---

# MON01 - Monitoring Server

## Purpose

Provides centralized monitoring and observability.

### Configuration

| Setting | Value |
|----------|----------|
| Hostname | MON01 |
| VLAN | VLAN20-INFRA |
| Port Group | PG-INFRA |
| IP Address | 10.1.20.20 |
| DNS | 10.1.20.10 |
| Operating System | Windows Server 2022 |

### Resource Allocation

| Resource | Allocation |
|----------|----------|
| vCPU | 4 |
| RAM | 8 GB |
| Disk | 50 GB |

### Services

- SolarWinds HCO
- Syslog Collection
- SNMP Monitoring
- NetFlow Monitoring

---

# AUTO01 - Automation Server

## Purpose

Provides automation and Infrastructure-as-Code services.

### Configuration

| Setting | Value |
|----------|----------|
| Hostname | AUTO01 |
| VLAN | VLAN20-INFRA |
| Port Group | PG-INFRA |
| IP Address | 10.1.20.30 |
| DNS | 10.1.20.10 |
| Operating System | Ubuntu Server 24.04 |

### Resource Allocation

| Resource | Allocation |
|----------|----------|
| vCPU | 2 |
| RAM | 2 GB |
| Disk | 20 GB |

### Services

- Git
- Ansible
- Python
- Docker
- Terraform

---

# EVE01 - Network Simulation Platform

## Purpose

Provides a platform for network engineering labs and certification preparation.

### Configuration

| Setting | Value |
|----------|----------|
| Hostname | EVE01 |
| VLAN | VLAN50-LAB |
| Port Group | PG-Networking |
| IP Address | 10.1.50.10 |
| DNS | 10.1.20.10 |
| Operating System | EVE-NG Community Edition |

### Resource Allocation

| Resource | Allocation |
|----------|----------|
| vCPU | 4 |
| RAM | 8 GB |
| Disk | 60 GB |

### Supported Platforms

- Cisco IOS
- Cisco IOS-XE
- Cisco ASA
- Juniper vSRX
- MikroTik CHR
- Linux Appliances

---

# Validation Checklist

- [ ] DC01 deployed successfully
- [ ] Active Directory operational
- [ ] DNS operational
- [ ] DHCP operational
- [ ] NTP operational
- [ ] MGMT01 deployed and domain joined
- [ ] MON01 deployed
- [ ] SolarWinds operational
- [ ] AUTO01 deployed
- [ ] Automation tools installed
- [ ] EVE01 operational
- [ ] Inter-VLAN routing verified
- [ ] Internet access verified
- [ ] DNS resolution verified

---

# Success Criteria

This phase is considered complete when:

- Core infrastructure services are operational.
- DNS and DHCP services are functioning correctly.
- Monitoring services are collecting infrastructure data.
- Automation services are available.
- Network simulation platforms are accessible.
- All deployed systems are documented and backed up.