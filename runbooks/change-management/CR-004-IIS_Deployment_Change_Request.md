# Change Request — IIS Web Server Deployment on MON01

---

## Document Control

| Field | Value |
|---|---|
| CR ID | CR-004 |
| Title | Deployment of Internet Information Services (IIS) on MON01 |
| Requested By | Leonard Noumba |
| Date Submitted | 21 June 2026 |
| Environment | Home Lab (VMware ESXi 8.0 / pfSense Multi-VLAN) |
| Status | Approved |
| Version | 1.0 |

---

## 1. Change Summary

| Field | Detail |
|---|---|
| Change ID | CR-004 |
| Change Title | Install and configure IIS Web Server role on MON01 |
| Change Type | Standard / Normal |
| Priority | Medium |
| Category | Infrastructure — Server Role Installation |
| Affected Service(s) | Web hosting capability within VLAN20 (INFRA) |
| Affected System(s) | MON01 (10.1.20.20) |

---

## 2. Change Classification

| Type | Applicable? | Justification |
|---|---|---|
| Standard Change | ✅ | Well-understood, low-risk, repeatable procedure (IIS role install via Server Manager/PowerShell) with documented rollback. Could be pre-approved in future under a Standard Change model once this CR establishes precedent. |

---

## 3. Description of Change

### 3.1 Current State

MON01 is a Windows Server 2022 virtual machine provisioned on VLAN20 (INFRA) with no server roles currently installed beyond the base operating system. The server has no web hosting capability.

### 3.2 Proposed Change

Install the **Web Server (IIS)** role on MON01, verify the default website is functional, configure Windows Firewall and pfSense rules to permit controlled HTTP access from other VLANs, and apply baseline post-deployment configuration (custom default document, directory browsing disabled).

### 3.3 Reason for Change

| Driver | Detail |
|---|---|
| Skills demonstration | Establishes IIS web server administration experience for portfolio and technical interviews |
| Lab capability expansion | Provides a foundation for future internal web-based tooling (dashboards, status pages, internal documentation portals) |
| Infrastructure completeness | Complements existing AD DS, network segmentation, and remote access work already documented in the lab series |

### 3.4 Scope

**In scope:**
- IIS role installation on MON01
- Default website verification
- Windows Firewall rule verification (auto-created by IIS)
- pfSense inter-VLAN firewall rule to permit HTTP (TCP/80) from approved source VLAN(s) to MON01
- Basic post-deployment hardening (directory browsing disabled, custom default document)

**Out of scope:**
- HTTPS/TLS certificate configuration *(planned as a follow-on change)*
- Domain-joining MON01 to homelab.local *(not required for this change)*
- Application-level deployment (no custom web application is being deployed at this stage)
- SolarWinds monitoring integration *(planned as a follow-on change)*

---

## 4. Risk Assessment

### 4.1 Risk Rating

| Factor | Rating | Notes |
|---|---|---|
| Likelihood of failure | Low | Standard Microsoft role installation, well-documented procedure |
| Impact if failed | Low | Single non-production lab server; no dependent services currently rely on MON01 |
| Overall Risk Rating | **Low** | No critical impact on Lab environment |

### 4.3 Security Considerations

- New inbound firewall surface is being created (TCP/80 to 10.1.20.20) — scoped to a specific source VLAN, not "Any"
- No authentication is configured on the default IIS site at this stage — site will serve only a static, non-sensitive landing page
- This change does **not** expose any service to the external internet or WAN — access remains internal to the home lab network only

---

## 5. Impact Assessment

| Area | Impact |
|---|---|
| Other servers/services | None — MON01 currently hosts no other dependent services |
| Network | One new pfSense firewall rule; no changes to VLAN structure, routing, or existing rules |
| Users | None — no active users depend on MON01 currently |
| Downtime required | Minimal — a single reboot may be required if Windows Updates are pending; IIS role installation itself does not require an outage window since no existing service is being interrupted |
| Data | No data migration or modification involved |

---

## 6. Implementation Plan

| Step | Action |
|---|---|
| 1 | Apply pending Windows Updates and reboot MON01 |
| 2 | Confirm static IP, hostname, and connectivity |
| 3 | Install Web Server (IIS) role |
| 4 | Verify default website starts and responds locally |
| 5 | Confirm Windows Firewall HTTP rule is enabled |
| 6 | Add scoped pfSense firewall rule (source VLAN → 10.1.20.20:80) |
| 7 | Verify cross-VLAN access |
| 8 | Apply post-deployment hardening (custom page, disable directory browsing) |
| 9 | Run full verification checklist |

**Estimated implementation time:** 30–50 minutes

**Implementation window:** No formal maintenance window required (non-production lab); to be performed during a planned lab session.

---

## 7. Rollback Plan

| Step | Action |
|---|---|
| 1 | Remove the pfSense firewall rule added in Step 6 of the Implementation Plan |
| 2 | Uninstall the IIS role: `Uninstall-WindowsFeature -Name Web-Server -Restart` |
| 3 | Confirm `Get-WindowsFeature -Name Web-Server` reports `Removed` |
| 4 | Confirm `Get-Service W3SVC` no longer returns a result |
| 5 | Verify MON01 returns to its pre-change state — no listening service on TCP/80 (`Test-NetConnection -ComputerName 10.1.20.20 -Port 80` should fail) |
| 6 | Document rollback execution and reason in this CR's Post-Implementation Review section |

**Rollback trigger conditions:**
- IIS role installation fails and cannot be resolved within the implementation window
- pfSense rule causes unintended access to other VLAN resources beyond MON01
- Any conflict identified with planned future roles on MON01

**Estimated rollback time:** 15–20 minutes

---

## 8. Testing & Validation Plan

Full verification matrix defined in **HL-SRV-002 §10 — Verification & Testing**, summarized below:

| Test | Success Criteria |
|---|---|
| IIS role installed | `Get-WindowsFeature Web-Server` shows `Installed` |
| Default site running | `Get-Website` shows state `Started` |
| Local HTTP test | `Invoke-WebRequest http://localhost` returns `200 OK` |
| Windows Firewall rule active | HTTP inbound rule shows `Enabled: True` |
| pfSense rule applied | Rule visible under source VLAN tab in Firewall → Rules |
| Cross-VLAN access | `Test-NetConnection 10.1.20.20 -Port 80` returns `TcpTestSucceeded: True` from another VLAN |
| Directory browsing disabled | `Get-WebConfigurationProperty directoryBrowse` returns `enabled: False` |

---

## 9. Communication Plan

| Stakeholder | Notification Required | Method |
|---|---|---|
| Self (lab owner/operator) | N/A — single-operator lab environment | N/A |
| GitHub documentation audience | Post-implementation — published change record | GitHub repository update |

---

## 10. Approval

| Role | Name | Decision | Date |
|---|---|---|---|
| Change Requester | Leonard Noumba | Submitted | 21 June 2026 |
| Change Approver | Leonard Noumba | ✅ Approved ☐ Rejected ☐ Approved with Conditions | |

**Approval conditions / comments:**
```
IIS deployment is would be required for long-term implementations. As result, I approve this deployment with immediate effect.
```

---

## 11. Post-Implementation Review

*To be completed after the change is implemented.*

| Field | Detail |
|---|---|
| Implementation Date | |
| Implemented By | |
| Outcome | ☐ Successful ☐ Successful with issues ☐ Rolled back |
| Actual Implementation Time | |
| Issues Encountered | |
| Deviations from Plan | |
| Follow-up Actions Required | |

---