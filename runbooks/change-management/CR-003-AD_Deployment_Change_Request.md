# Change Request — Active Directory Domain Services Deployment
## CR-2026-003

---

## 1. Change Request Summary

| Field | Detail |
|---|---|
| CR ID | CR-003 |
| Title | Deployment of Active Directory Domain Services on DC01 |
| Requested By | Leonard Noumba |
| Change Owner / Implementer | Leonard Noumba |
| Date Raised | 2026-06-15 |
| Date Implemented | 2026-06-18 |
| Environment | Enterprise Home Lab |
| Change Type | Standard |
| Change Category | Server / Identity Infrastructure |
| Priority | High |
| Status | ✅ Completed |

---

## 2. Description of Change

Deployment of Active Directory Domain Services (AD DS) on a Windows Server 2022 virtual machine (**DC01**), establishing a new Active Directory forest and domain (`homelab.local`) to serve as the centralised identity and authentication platform for the home lab environment. The change includes server preparation, AD DS role installation, domain controller promotion, DNS configuration, Organisational Unit (OU) structure creation, user and group provisioning, Group Policy Object (GPO) baseline configuration, and verification via a domain-joined workstation.

---

## 3. Reason for Change

| Driver | Detail |
|---|---|
| Business / Lab Justification | The lab previously had no centralised identity management — all VMs used local accounts only, with no consistent password policy, audit trail, or policy enforcement across machines. |
| Technical Justification | Active Directory is required to demonstrate enterprise-relevant skills (user/group management, GPO, DNS integration, domain join) for portfolio and interview purposes, and to support future lab services (file shares, RDS, monitoring) that depend on domain authentication. |
| Risk of Not Implementing | Continued reliance on local accounts limits the realism of the lab environment and prevents demonstration of core sysadmin/IT operations competencies expected in NOC, sysadmin, and infrastructure roles. |

---

## 4. Scope

### 4.1 In Scope

- Installation of AD DS role on DC01
- Promotion of DC01 to Domain Controller (new forest/domain: `homelab.local`)
- DNS forward and reverse lookup zone configuration
- Default Domain Password Policy configuration
- Organisational Unit (OU) hierarchy creation
- Security group creation
- User account provisioning (3 accounts)
- Group Policy Object creation and linking (1+ GPOs)
- Domain join of one server (MON01) for verification

### 4.2 Out of Scope

- Deployment of a secondary Domain Controller (DC02) — planned for a future change request
- Fine-Grained Password Policies (PSOs)
- AD Sites and Services configuration
- Integration with SolarWinds monitoring
- Certificate Services (AD CS)
- Group Policy software deployment

---

## 5. Affected Systems

| System | VLAN | IP Address | Impact |
|---|---|---|---|
| DC01 (Windows Server 2022) | VLAN10 MGMT | 10.1.20.10 | New role installed; server promoted to Domain Controller |
| MON01 | VLAN20 MGMT | 10.1.20.x | Joined to domain for verification |
| pfSense | All VLANs | 10.1.x.1 | Firewall rules reviewed for AD ports (DNS, Kerberos, LDAP, SMB, GC) |

---

## 6. Pre-Change Checklist

| # | Item | Status |
|---|---|---|
| 1 | DC01 VM deployed and Windows Server 2022 installed | ✅ Complete |
| 2 | Static IP assigned to DC01 (10.1.20.10) | ✅ Complete |
| 3 | Hostname set to DC01 prior to promotion | ✅ Complete |
| 4 | Time zone configured correctly | ✅ Complete |
| 5 | Windows Updates applied | ✅ Complete |
| 6 | DSRM password generated and stored securely | ✅ Complete |
| 7 | Rollback plan reviewed | ✅ Complete |
| 8 | Maintenance window confirmed (lab — no external impact) | ✅ Complete |

---

## 7. Implementation Plan

| Step | Action | Reference |
|---|---|---|
| 1 | Configure static IP, hostname, time zone on DC01 | PROJ-01 |
| 2 | Install AD DS role | PROJ-04 |
| 3 | Promote DC01 to Domain Controller (new forest: homelab.local) | PROJ-04 |
| 4 | Configure reverse DNS lookup zone and NTP | PROJ-004 |
| 5 | Apply Default Domain Password Policy | PROJ-05 |
| 6 | Create OU hierarchy | PROJ-05 |
| 7 | Create security groups and user accounts | PROJ-05 |
| 8 | Create and link GPOs (Baseline Security, Workstation Config, Server Config) | PROJ-05 |
| 9 | Verify DNS (SRV records, dcdiag) | PROJ-05 |
| 10 | Join MON01 to domain and verify | PROJ-05 |

**Implementation Method:** Combination of GUI (Server Manager, Active Directory Users and Computers, Group Policy Management Console) and PowerShell, per PROJ-04 and PROJ-05 documentation.

---

## 8. Rollback Plan

| Scenario | Rollback Action |
|---|---|
| AD DS promotion fails or produces a non-functional domain | Demote DC01 via `Uninstall-ADDSDomainController -DemoteOperationMasterRole -RemoveApplicationPartitions`, then revert DC01 to standalone workgroup configuration |
| GPO causes unintended lockout or access issue | Unlink the offending GPO via Group Policy Management Console (`Remove-GPLink`), then run `gpupdate /force` on affected machines |
| Domain join breaks workstation connectivity | Remove workstation from domain (`Remove-Computer`), rejoin to workgroup, restore local administrator access |
| DC01 becomes unresponsive post-promotion | Boot into Directory Services Restore Mode (DSRM) using the stored DSRM password and restore from VM snapshot if available |
| Critical failure with no recovery path | Restore DC01 from pre-change ESXi snapshot |

**Snapshot Taken Before Change:** ✅ Yes — ESXi snapshot of DC01 taken prior to role installation.

---

## 9. Risk Assessment

| Risk | Likelihood | Impact | Mitigation |
|---|---|---|---|
| Incorrect DNS configuration breaks domain resolution | Medium | High | DNS verified at each phase (forward, reverse, SRV records, `dcdiag`) before proceeding |
| GPO misconfiguration locks out admin access | Low | High | Logon banner and lockout policies tested on a non-admin account before being relied upon |
| Domain join fails due to inter-VLAN firewall rules | Medium | Medium | pfSense firewall rules for AD-required ports verified prior to join attempt |
| DSRM password lost | Low | High | Password recorded and stored securely outside the lab environment |
| Single point of failure (one DC only) | High | Medium | Documented as a known limitation; DC02 deployment planned as follow-up CR |

**Overall Risk Rating:** Low — lab environment, no production/business-critical dependency, full snapshot rollback available.

---

## 10. Testing & Validation

| Test | Result |
|---|---|
| `Get-ADDomain` / `Get-ADForest` return correct domain/forest info | ✅ Pass |
| `netdom query fsmo` confirms all 5 roles on DC01 | ✅ Pass |
| DNS forward lookup (`DC01.homelab.local` → 10.10.10.10) | ✅ Pass |
| DNS reverse lookup (10.10.10.10 → `DC01.homelab.local`) | ✅ Pass |
| SRV records resolve (`_ldap`, `_kerberos`, `_gc`) | ✅ Pass |
| `dcdiag /v` — all tests pass (Replications shows expected single-DC warning) | ✅ Pass (with known limitation) |
| OU hierarchy created and verified (`Get-ADOrganizationalUnit`) | ✅ Pass |
| All 4 security groups created | ✅ Pass |
| All 3 user accounts created and placed in correct OUs | ✅ Pass |
| Group memberships correct (`Get-ADGroupMember`) | ✅ Pass |
| All 3 GPOs created and linked (`Get-GPO -All`) | ✅ Pass |
| Password policy applied (`Get-ADDefaultDomainPasswordPolicy`) | ✅ Pass |
| MON01 successfully joined domain | ✅ Pass |
| MON01 placed in correct OU (Servers) | ✅ Pass |
| Domain login successful on MON01 (`HOMELAB\lnoumba`) | ✅ Pass |
| Logon banner displays correctly on MON01 | ✅ Pass |
| GPO applies on MON01 (`gpresult /r`) | ✅ Pass |

---

## 11. Issues Encountered During Implementation

| Issue | Root Cause | Resolution | Reference |
|---|---|---|---|
| Remote GPO update to MON01 failed — Error `8007071a` ("the remote procedure call was cancelled") | RPC/firewall interruption during remote policy push to MON01 | Verified MON01 connectivity, enabled required firewall rule groups (WMI, Remote Scheduled Tasks, Remote Event Log Management), retried `Invoke-GPUpdate` | Resolved |
| Group Policy processing failure — unable to read `gpt.ini` from SYSVOL | Investigated as DNS, SMB/RPC connectivity, or DFSR replication latency issue | Verified DNS resolution, SMB port 445 reachability, SYSVOL share availability, and DFSR service status on DC01 | Resolved |

> Both issues were transient/connectivity-related and did not require structural changes to the AD design. Full diagnostic steps are documented in PROJ-05 (Troubleshooting Reference).

---

## 12. Post-Implementation Review

| Item | Outcome |
|---|---|
| Change implemented within planned scope | ✅ Yes |
| All validation tests passed | ✅ Yes |
| Rollback required | ❌ No |
| Unplanned downtime | ❌ None (lab environment) |
| Documentation updated | ✅ PROJ-05 published |
| Follow-up changes identified | ✅ See Section 13 |

---

## 13. Follow-Up Actions / Related Future CRs

| # | Action | Target CR |
|---|---|---|
| 1 | Deploy DC02 on VLAN20 (INFRA) for redundancy and AD replication | CR-003 (planned) |
| 2 | Implement Fine-Grained Password Policies (PSOs) for `GRP-IT-Admins` | CR-004 (planned) |
| 3 | Integrate SolarWinds monitoring for DC health and replication alerts | CR-006 (planned) |
| 4 | Configure AD Sites and Services to align with VLAN topology | CR-007 (planned) |

---

## 14. Approval

| Role | Name | Decision | Date |
|---|---|---|---|
| Change Requester | Leonard Noumba | Approved | 2026-06-17 |
| Change Implementer | Leonard Noumba | Approved | 2026-06-17 |
| Change Reviewer | Leonard Noumba (self-reviewed — single-operator lab) | Approved | 2026-06-17 |

> **Note:** As this is a single-operator home lab environment, the requester, implementer, and reviewer roles are held by the same individual. In an enterprise setting, these roles would be segregated across distinct stakeholders (e.g. requester, technical lead, change advisory board) as part of standard change management governance.

---

## References

- PROJ-04 — Active Directory DS Installation & Promotion
- PROJ-05 — AD User & Group Management, GPO, DNS Verification & Domain Join
