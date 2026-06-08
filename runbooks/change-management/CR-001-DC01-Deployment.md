# Change Request

| Field | Value |
|---------|---------|
| Change ID | CR-001 |
| Title | Deploy DC01 Domain Controller |
| Change Type | Standard |
| Priority | Medium |
| Risk Level | Medium |
| Status | Planned |
| Requested By | Leonard |
| Implemented By | Leonard |
| Date Raised | 2026-06-08 |
| Planned Implementation Date | 2026-06-09 |
| Actual Implementation Date | TBD |

---

# 1. Executive Summary

This change introduces the first Active Directory Domain Controller (DC01) into the Enterprise Hybrid Home Lab environment.

The server will provide centralized authentication, DNS services, DHCP services, and time synchronization for all infrastructure components.

---

# 2. Business Justification

The deployment of DC01 is required to establish enterprise-grade identity and infrastructure services.

Benefits include:

- Centralized authentication
- DNS management
- DHCP management
- Group Policy support
- Simplified administration
- Enterprise architecture alignment

---

# 3. Scope

## In Scope

- Deploy Windows Server 2022
- Configure static IP address
- Install Active Directory Domain Services
- Install DNS
- Install DHCP
- Create corp.homelab domain

## Out of Scope

- Additional Domain Controllers
- Azure AD Integration
- Certificate Services

---

# 4. Systems Affected

| System | Impact |
|----------|----------|
| ESXi | New VM Deployment |
| pfSense | No Impact |
| Infrastructure VLAN | New Service Introduction |

---

# 5. Implementation Plan

## Step 1

Create virtual machine.

### Configuration

```text
Hostname: DC01
OS: Windows Server 2022
CPU: 1 vCPU
RAM: 1 GB
Disk: 10 GB
Port Group: PG-INFRA
```

---

## Step 2

Configure networking.

```text
IP Address: 10.1.20.10
Subnet Mask: 255.255.255.0
Gateway: 10.1.20.1
DNS: 127.0.0.1
```

---

## Step 3

Install required roles.

- Active Directory Domain Services
- DNS
- DHCP

---

## Step 4

Promote server to Domain Controller.

```text
Domain Name:
corp.homelab
```

---

# 6. Risk Assessment

| Risk | Probability | Impact | Mitigation |
|--------|--------|--------|--------|
| Incorrect IP configuration | Medium | Medium | Verify addressing before deployment |
| DNS failure | Medium | High | Validate DNS after installation |
| Domain promotion failure | Low | High | Snapshot VM before promotion |

---

# 7. Rollback Plan

If deployment fails:

1. Shut down VM.
2. Restore snapshot.
3. Verify network configuration.
4. Reattempt deployment.

If severe issues occur:

1. Delete VM.
2. Recreate from template.

---

# 8. Validation Plan

The following tests must pass:

- [ ] Ping gateway
- [ ] Ping internet
- [ ] DNS resolution works
- [ ] Domain controller operational
- [ ] DHCP service operational
- [ ] Active Directory Users and Computers opens
- [ ] SYSVOL shared successfully

---

# 9. Post-Implementation Review

## Result

TBD

## Issues Encountered

TBD

## Lessons Learned

TBD

---

# 10. Change Approval

| Role | Name | Status |
|--------|--------|--------|
| Requestor | Leonard | Approved |
| Implementer | Leonard | Approved |
| Reviewer | Leonard | Approved |

---

# Change Status History

| Date | Status | Notes |
|---------|---------|---------|
| 2026-06-08 | Planned | Change created |