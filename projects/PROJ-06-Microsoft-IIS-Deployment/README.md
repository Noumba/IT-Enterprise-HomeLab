# IIS Web Server — Installation & Configuration
## Project Documentation — PROJ-006

---

## Document Control

| Field | Value |
|---|---|
| Document ID | PROJ-006 |
| Title | Internet Information Services (IIS): Installation, Configuration & Verification |
| Version | 1.0 |
| Author | Leonard Noumba |
| Environment | VMware ESXi 8.0 / Windows Server 2022 / pfSense Multi-VLAN Home Lab |
| Server | MON01 — VLAN20 (10.1.20.20) |

---

## Table of Contents

1. [Executive Summary](#1-executive-summary)
2. [Objectives](#2-objectives)
3. [Environment Overview](#3-environment-overview)
4. [Prerequisites](#4-prerequisites)
5. [Phase 2 — Install IIS Role](#5-phase-2--install-iis-role)
6. [Phase 3 — Verify Default Website](#6-phase-3--verify-default-website)
7. [Phase 4 — pfSense Firewall Configuration](#7-phase-4--pfsense-firewall-configuration)
8. [Verification & Testing](#8-verification--testing)
9. [Lessons Learned](#9-lessons-learned)
10. [Troubleshooting Reference](#10-troubleshooting-reference)

---

## 1. Executive Summary

This document describes the installation and configuration of Internet Information Services (IIS) on a Windows Server 2022 virtual machine (MON01) running on VLAN20 within the home lab. The deployment installs the core Web Server (IIS) role, verifies the default website is serving content, configures the pfSense firewall to permit HTTP traffic to the server, and validates accessibility both locally and from other VLANs.

This deployment establishes the foundation for hosting web-based services within the home lab — a common requirement in enterprise environments for internal portals, monitoring dashboards, and application front-ends — and demonstrates core web server administration skills applicable to IIS-based infrastructure roles.

---

## 2. Objectives

| ID | Objective |
|---|---|
| OBJ-01 | Install the Web Server (IIS) role on MON01 |
| OBJ-02 | Verify the default IIS website is running and serving content |
| OBJ-03 | Configure pfSense firewall rules to permit HTTP access to MON01 |
| OBJ-04 | Verify access to the site locally and from other VLANs |
| OBJ-05 | Apply basic post-deployment hardening and configuration |
| OBJ-06 | Produce enterprise-grade documentation for portfolio and GitHub |

---

## 3. Environment Overview

### 3.1 Infrastructure

| Component | Detail |
|---|---|
| Hypervisor | VMware ESXi 8.0 (HP EliteDesk 800 G4) |
| Firewall / Router | pfSense |
| VLAN10 (MGMT) | 10.1.10.0/24 — Gateway 10.10.10.1 |
| VLAN20 (INFRA) | 10.1.20.0/24 — Gateway 10.1.20.1 *(see addressing note above)* |
| VLAN30 (DMZ) | 10.1.30.0/24 |
| VLAN50 (LAB) | 10.1.50.0/24 |

### 3.2 MON01 Server Specifications

| Parameter | Value |
|---|---|
| VM Name | MON01 |
| Operating System | Windows Server 2022 Standard (Desktop Experience) |
| vCPUs | 2 |
| RAM | 4 GB |
| Disk | 40 GB (OS) |
| NIC | Connected to PG-INFRA (VLAN20) |
| IP Address | 10.1.20.20 (Static) |
| Subnet Mask | 255.255.255.0 |
| Default Gateway | 10.1.20.1 |
| Preferred DNS | 10.1.20.10 (DC01) |
| Role | Web Server (IIS) |

---

## 4. Prerequisites

| # | Prerequisite | Status |
|---|---|---|
| 1 | MON01 VM is deployed on ESXi and powered on | ✅ |
| 2 | Windows Server 2022 is installed with Desktop Experience | ✅ |
| 3 | Local Administrator password is set | ✅ |
| 4 | MON01 NIC is connected to PG-INFRA (VLAN20) | ✅ |
| 5 | Static IP configured per Section 3.2 | ✅ |
| 6 | MON01 can ping the pfSense VLAN20 gateway | ✅ |
| 7 | Windows Server 2022 is activated or in evaluation mode | ✅ |
| 8 | Remote Desktop is enabled on MON01 for remote management | ✅ |
| 9 | Windows Updates are current | ✅ |

---

## 5. Phase 2 — Install IIS Role

### 5.1 — Install via Server Manager (GUI)

1. Open **Server Manager**
2. Click **Manage → Add Roles and Features**
3. **Before You Begin** → click **Next**
4. **Installation Type** → select **Role-based or feature-based installation** → **Next**
5. **Server Selection** → confirm `MON01` is selected → **Next**
6. **Server Roles** → check **Web Server (IIS)**
7. A pop-up appears listing required features (e.g. Management Tools) — click **Add Features**
8. Click **Next** through the Features page (no additional features required for a basic deployment)
9. **Web Server Role (IIS)** introduction page → click **Next**
10. **Role Services** — review the default selections:

| Role Service | Included by Default |
|---|---|
| Common HTTP Features | ✅ Static Content, Default Document, Directory Browsing, HTTP Errors |
| Health and Diagnostics | ✅ HTTP Logging |
| Performance | ✅ Static Content Compression |
| Security | ✅ Request Filtering |
| Management Tools | ✅ IIS Management Console |

Leave defaults selected for a basic deployment → click **Next**

11. **Confirmation** → check **Restart the destination server automatically if required**
12. Click **Install**
13. Wait for installation to complete — do **not** close the window
14. Click **Close** once installation finishes

---

## 6. Phase 3 — Verify Default Website

### 6.1 Confirm IIS Service is Running

1. Open **Server Manager → Tools → Internet Information Services (IIS) Manager**
2. Expand `MON01 → Sites → Default Web Site`
3. In the right-hand **Actions** pane, confirm the site shows as **Started**
4. If stopped, click **Start** in the Actions pane

---

### 6.2 Test Locally on MON01

1. Open a web browser on MON01
2. Navigate to `http://localhost`
3. Confirm the default **IIS Windows Server welcome page** loads (blue/teal IIS branded page)

---

### 6.3 Confirm Site Bindings

1. In **IIS Manager**, select **Default Web Site**
2. In the **Actions** pane, click **Bindings**
3. Confirm a binding exists:

| Type | IP Address | Port | Host Name |
|---|---|---|---|
| http | * (All Unassigned) | 80 | (blank) |

4. Click **Close**
---

## 7. Phase 4 — pfSense Firewall Configuration

By default, pfSense permits traffic *within* a VLAN but blocks traffic *between* VLANs unless explicitly allowed. To access MON01's website from another VLAN (e.g. MGMT), a firewall rule must be added.

### 7.1 Confirm Windows Firewall Permits HTTP

IIS installation automatically creates inbound firewall rules for HTTP. Verify these are enabled.

1. Open **Windows Defender Firewall with Advanced Security**
2. Click **Inbound Rules**
3. Locate **World Wide Web Services (HTTP Traffic-In)**
4. Confirm the rule is **Enabled** (green checkmark). If not, right-click → **Enable Rule**

---

### 7.2 Add a pfSense Firewall Rule (Allow Inter-VLAN Access)

#### GUI (pfSense)

1. Log into the pfSense GUI
2. Go to **Firewall → Rules**
3. Select VLAN
4. Click **+ Add**
5. Configure:

| Field | Value |
|---|---|
| Action | Pass |
| Protocol | TCP |
| Source | VLAN10 MGMT subnet (or appropriate source) |
| Destination | Single host — `10.1.20.20` |
| Destination Port Range | HTTP (80) |
| Description | Allow access to MON01 IIS site |

6. Click **Save** → **Apply Changes**

---

### 7.3 Verify Access from Another VLAN

From a device on a different VLAN (e.g. a machine on VLAN10 MGMT):

Then open a browser and navigate to:
```
http://10.1.20.20
```

Confirm the IIS welcome page loads.

---

## 8. Verification & Testing

| Test | Command / Method | Expected Result |
|---|---|---|
| Cross-VLAN browser test | Browse to `http://10.1.20.20` from another VLAN | Page loads |
| Custom page serving | Browse to `http://10.1.20.20` | Custom `index.html` content displays |

---

## 9. Lessons Learned

### 9.1 Technical

- **A Domain Controller is not the only server type that needs a static IP.** Any infrastructure service — web, monitoring, file sharing — should have a fixed address so firewall rules, DNS records, and bookmarks remain valid over time.
- **IIS installation auto-creates a Windows Firewall rule, but it does not configure anything at the network firewall layer.** In a segmented VLAN environment, the Windows Firewall rule only solves connectivity *on the local subnet* — pfSense rules are still required for any cross-VLAN access, and this is a step that is easy to forget when the site "works" locally but is unreachable from elsewhere.
- **Testing locally (`http://localhost`) before testing cross-VLAN isolates the failure domain.** If local access works but cross-VLAN access fails, the issue is almost always firewall-related (Windows or pfSense), not an IIS configuration problem — this two-step test saves significant troubleshooting time.
- **Default Document order matters.** IIS serves the first matching file in its Default Document list — if a custom `index.html` is added but not moved to the top of that list, IIS may continue to serve the original default page, which can look like a deployment failure when it's actually a configuration order issue.

---

## 10. Troubleshooting Reference

| Symptom | Likely Cause | Resolution |
|---|---|---|
| `http://localhost` fails on MON01 | W3SVC service stopped | `Start-Service W3SVC` |
| Site loads locally but not from other VLANs | pfSense rule missing | Add rule per Section 8.2 |
| Site loads locally but not from other VLANs (after pfSense rule added) | Windows Firewall rule disabled | `Enable-NetFirewallRule -DisplayName "World Wide Web Services (HTTP Traffic-In)"` |
| Browser shows default IIS page instead of custom page | Default Document order incorrect | Move `index.html` to top of Default Document list |
| `Test-NetConnection` succeeds but browser shows nothing | Site stopped or wrong binding | Check `Get-Website` state and `Get-WebBinding` |
| 403 Forbidden error | Directory Browsing disabled with no default document present | Ensure a valid default document (e.g. `index.html`) exists in `wwwroot` |
| Role install fails | Pending Windows Update reboot | Restart server, retry installation |
| Cross-VLAN test fails even with rules added | Rule on wrong VLAN tab in pfSense | Confirm rule is added under the **source** VLAN tab, not the destination |

---

## 11. References

- Microsoft Documentation — Internet Information Services (IIS): https://learn.microsoft.com/en-us/iis/
