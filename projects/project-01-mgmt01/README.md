# Project 02 - MGMT01 Management Workstation Deployment

## Project Overview

MGMT01 is the dedicated management workstation for the Enterprise Hybrid Home Lab.

The purpose of this system is to provide a centralized administrative platform for managing infrastructure services, virtualization, networking, monitoring, automation, and cloud integrations.

This server follows enterprise administration best practices by providing a dedicated jump host for management activities instead of using personal devices.

---

# Project Information

| Item | Value |
|--------|--------|
| Project ID | PROJ-001 |
| Project Name | MGMT01 Deployment |
| Status | Completed |
| Priority | High |
| Environment | Home Lab |
| Project Owner | Leonard |
| Implementation Date | 2026-06-08 |

---

# Business Requirement

A dedicated management workstation is required to provide secure and centralized access to infrastructure resources.

The workstation serves as the primary administration platform for:

- VMware ESXi
- pfSense
- Active Directory
- DNS
- DHCP
- SolarWinds
- EVE-NG
- GitHub
- Automation Services

---

# Objectives

- Deploy a dedicated management workstation.
- Join workstation to Active Directory.
- Centralize infrastructure administration.
- Improve security through role separation.
- Provide a platform for remote administration tools.

---

# System Specifications

## Virtual Machine Details

| Parameter | Value |
|------------|------------|
| VM Name | MGMT01 |
| Hostname | MGMT01 |
| Guest OS | Windows Server 2022 |
| Hypervisor | VMware ESXi |
| Power State | Powered On |

---

## Resource Allocation

| Resource | Allocation |
|------------|------------|
| vCPU | 1 |
| Memory | 2 GB |
| Storage | 25 GB |
| Network Adapter | VMXNET3 |

---

# Network Configuration

| Parameter | Value |
|------------|------------|
| VLAN | VLAN10-MGMT |
| Port Group | PG-MGMT |
| IP Address | 10.1.10.10 |
| Subnet Mask | 255.255.255.0 |
| Default Gateway | 10.1.10.1 |
| Primary DNS | 10.1.20.10 |
| Domain | corp.homelab |

---

# Architecture Placement

```text
Internet
    |
ISP Router
    |
pfSense
    |
VLAN10 - MGMT
    |
MGMT01
```

---

# Installed Software

## Administrative Tools

- RSAT
- VMware Remote Console
- Windows Terminal
- PowerShell 7
- PuTTY
- WinSCP

## Development Tools

- Visual Studio Code
- Git
- Python

## Utilities

- Google Chrome
- Notepad++
- 7-Zip

---

# Active Directory Integration

## Domain Membership

```text
corp.homelab
```

## Organizational Unit

```text
Workstations
```

## Group Membership

- Domain Users
- Infrastructure Administrators

---

# Administrative Responsibilities

MGMT01 is used for:

## VMware Administration

- ESXi Management
- Virtual Machine Management
- Datastore Management

## Network Administration

- pfSense Administration
- Firewall Management
- VLAN Management
- Routing Verification

## Windows Administration

- Active Directory
- DNS
- DHCP
- Group Policy

## Monitoring Administration

- SolarWinds
- SNMP Configuration
- Syslog Analysis

## Automation Administration

- GitHub Management
- Ansible Administration
- PowerShell Development

---

# Security Controls

## Endpoint Security

- Windows Defender Enabled
- Automatic Updates Enabled
- Local Firewall Enabled

## Administrative Access

- Domain Authentication Required
- Local Administrator Disabled
- Strong Password Policy Enforced

## Network Security

- Management VLAN Isolation
- Restricted Firewall Rules
- Controlled Administrative Access

---

# Validation Results

| Test | Result |
|--------|--------|
| VM Power On | PASS |
| Static IP Configuration | PASS |
| Gateway Reachability | PASS |
| DNS Resolution | PASS |
| Internet Access | PASS |
| Domain Join | PASS |
| Active Directory Authentication | PASS |
| ESXi Access | PASS |
| pfSense Access | PASS |

---

# Operational Procedures

## Daily Tasks

- Monitor infrastructure health.
- Review SolarWinds alerts.
- Verify backup status.
- Review event logs.

## Weekly Tasks

- Install updates.
- Verify DNS health.
- Verify Active Directory health.
- Review firewall logs.

## Monthly Tasks

- Snapshot review.
- Security audit.
- Documentation review.
- Resource utilization review.

---

# Risks

| Risk | Impact | Mitigation |
|----------|----------|----------|
| Workstation failure | High | Backup VM regularly |
| DNS issues | Medium | Monitor DNS services |
| Domain issues | High | Verify AD health checks |
| Unauthorized access | High | Enforce strong authentication |

---

# Lessons Learned

- Dedicated management systems simplify administration.
- Domain integration improves operational efficiency.
- Centralized administration aligns with enterprise practices.
- Proper VLAN segmentation improves security.

---

# Future Enhancements

- Multi-Factor Authentication
- Remote Desktop Gateway
- Password Vault Integration
- Privileged Access Workstation Configuration
- Azure AD Integration

---

# Project Status

## Current Status

```text
COMPLETED
```

## Project Outcome

The MGMT01 workstation was successfully deployed and integrated into the Enterprise Hybrid Home Lab environment.

The platform now serves as the centralized administration workstation for all infrastructure, networking, monitoring, automation, and cloud management activities.

---

# References

- CR-002-MGMT01-Deployment.md
- VMware-Design.md
- VLAN-Design.md
- IP-Addressing-Plan.md
- Active-Directory-Design.md