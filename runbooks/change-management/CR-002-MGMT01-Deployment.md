# Change Request - CR-006

| Field | Value |
|---------|---------|
| Change ID | CR-006 |
| Title | Deploy MGMT01 Management Workstation |
| Change Type | Standard |
| Priority | Medium |
| Risk Level | Low |
| Status | Planned |
| Requested By | Leonard |
| Implemented By | Leonard |
| Date Raised | 2026-06-08 |
| Planned Implementation Date | 2026-06-08 |
| Actual Implementation Date | 2026-06-08 |

---

## 1. Executive Su06ary

This change introduces the MGMT01 Management Workstation into the Enterprise Hybrid Home Lab environment.

MGMT01 will serve as the centralized administrative workstation used to manage VMware ESXi, pfSense, Active Directory, monitoring systems, and other infrastructure components.

The server will function as a secure jump box and administration platform, reducing the need to manage infrastructure directly from personal devices.

---

## 2. Business Justification

The deployment of MGMT01 provides a dedicated administrative endpoint that aligns with enterprise operational and security best practices.

### Benefits

- Centralized administration
- Improved security
- Dedicated management platform
- Reduced dependency on personal workstations
- Simplified remote administration
- Enterprise-aligned architecture

---

## 3. Scope

### In Scope

- Deploy Windows 11 Pro virtual machine
- Configure static IP address
- Join workstation to Active Directory domain
- Install administration tools
- Configure remote management utilities

### Out of Scope

- Endpoint management solutions
- Azure AD Join
- Microsoft Intune integration
- Multi-factor authentication

---

## 4. Systems Affected

| System | Impact |
|----------|----------|
| VMware ESXi | New VM deployment |
| VLAN10-MGMT | New endpoint |
| Active Directory | New domain member |
| DNS | New host record |
| DHCP | No impact |

---

## 5. Implementation Plan

### Step 1 - Create Virtual Machine

Deploy a new virtual machine on VMware ESXi.

#### Configuration

```text
VM Name: MGMT01
Guest OS: Windows Server 2022
Port Group: PG-MGMT
vCPU: 1
Memory: 2 GB
Disk: 25 GB
```

### Step 2 - Configure Networking

Assign a static IP address.

```text
IP Address: 10.1.10.100
Subnet Mask: 255.255.255.0
Gateway: 10.1.10.1
Primary DNS: 10.1.20.10
```

### Step 3 - Validate Connectivity

Verify:

- Gateway reachability
- DNS resolution
- Internet access
- Active Directory connectivity

### Step 4 - Join Domain

Join workstation to:

```text
corp.homelab
```

### Step 5 - Install Administrative Tools

Install:

- RSAT Tools
- VMware Remote Console
- PuTTY
- Windows Terminal
- Visual Studio Code
- Git

---

## 6. Risk Assessment

| Risk | Probability | Impact | Mitigation |
|----------|----------|----------|----------|
| Incorrect network configuration | Low | Medium | Validate addressing before deployment |
| Domain join failure | Low | Medium | Verify DNS configuration |
| Resource overallocation | Low | Low | Follow approved resource plan |
| DNS resolution issues | Medium | Medium | Validate DC01 DNS services |

---

## 7. Rollback Plan

If deployment fails:

1. Shut down MGMT01.
2. Remove VM from inventory.
3. Delete VM files if required.
4. Review deployment logs.
5. Correct configuration issues.
6. Redeploy workstation.

---

## 8. Validation Plan

The following tests must pass:

- [ ] VM powers on successfully
- [ ] Static IP configuration verified
- [ ] Gateway reachable
- [ ] DNS resolution successful
- [ ] Internet access verified
- [ ] Domain join successful
- [ ] RSAT tools installed
- [ ] VMware management access verified
- [ ] pfSense management access verified

---

## 9. Success Criteria

This change will be considered successful when:

- MGMT01 is operational.
- The workstation is joined to the corp.homelab domain.
- DNS resolution is functioning correctly.
- Administrative tools are installed.
- Infrastructure management can be performed from MGMT01.

---

## 10. Post-Implementation Review

### Result

TBD

### Issues Encountered

TBD

### Lessons Learned

TBD

---

## 11. Change Approval

| Role | Name | Status |
|---------|---------|---------|
| Requestor | Leonard | Approved |
| Implementer | Leonard | Approved |
| Reviewer | Leonard | Approved |

---

## Change Status History

| Date | Status | Notes |
|---------|---------|---------|
| 2026-06-08 | Planned | Change request created |
| 2026-06-08 | In Progress | Deployment started |
| 2026-06-08 | Completed | Deployment completed successfully |