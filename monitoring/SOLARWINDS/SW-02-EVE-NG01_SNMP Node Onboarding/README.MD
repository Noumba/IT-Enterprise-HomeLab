# SolarWinds Observability Self-Hosted — SNMP Node Onboarding

---

## Document Control

| Field | Value |
|---|---|
| Document ID | SW-02 |
| Title | SolarWinds SNMP Node Onboarding |
| Version | 1.0 |
| Author | Leonard Noumba |
| Environment | SolarWinds Observability Self-Hosted on MON01 (10.1.20.20) |

---

## Overview

This document demonstrates how to onboard a Cisco router running inside EVE-NG into SolarWinds Observability Self-Hosted (SWOSH) for active monitoring. 

The device is monitored via **SNMPv2c**, which provides interface traffic statistics, CPU load, memory utilisation, and node availability. The device was pre-configured with SNMP before onboarding — refer to [EVE-NG01 Lab](../../../eve-ng%20labs/EVE-NG01-Core-Access%20Network%20Simulation%20Lab/README.MD) for the full SNMP configuration applied on the device.

---

## Prerequisites

| Item | Detail |
|---|---|
| Toronto-CR1 IP | 10.1.50.100 |
| SNMP Version | v2c |
| Community String | `DaVince-SNMP-RO` |
| MON01 reachable from CR1 | Confirmed — ping 10.1.20.20 succeeds |
| pfSense static route added | 192.168.10.0/24 → 10.1.50.100 |
| SNMP port open | UDP 161 confirmed via `Test-NetConnection` |

---

## Step 1 — Log into Orion Web Console

1. Open a browser from any device on the lab network
2. Navigate to `https://10.1.20.20`
3. Log in with administrator credentials

> 📸 *Screenshot: Orion login page*

---

## Step 2 — Navigate to Manage Nodes

1. Click **Settings** in the top navigation bar
2. Select **All Settings** from the dropdown
3. Under the **Node & Group Management** section, click **Manage Nodes**

> 📸 *Screenshot: Settings → All Settings → Manage Nodes*

---

## Step 3 — Add New Node

1. Click the **Add Node** button (top left of the Manage Nodes page)
2. The **Add Node** wizard opens

> 📸 *Screenshot: Manage Nodes page with Add Node button highlighted*

---

## Step 4 — Enter Node Details

1. In the **Hostname or IP Address** field enter:
```
10.1.50.100
```
2. Under **Polling Method** select **SNMP**
3. Set **SNMP Version** to **SNMPv2c**
4. Enter the community string:
```
DaVince-SNMP-RO
```
5. Leave all other fields at default
6. Click **Test** to verify SNMP connectivity

> 📸 *Screenshot: Add Node form filled with IP and SNMP details*

**Expected Test Result:**
```
System name:     Toronto-CR1
System location: DaVince-Technologies-HomeLab-EVE-NG
System contact:  leo@homelab.local
SNMP status:     ✅ Successful
```

> 📸 *Screenshot: Successful SNMP test result*

7. Click **Next**

---

## Step 5 — Select Resources to Monitor

SolarWinds will display all available resources discovered via SNMP. Select the following:

| Resource | Why |
|---|---|
| ✅ ICMP Node Status | Tracks device up/down availability |
| ✅ CPU Load | Monitors router CPU utilisation |
| ✅ Memory Used | Monitors router memory utilisation |
| ✅ All Network Interfaces | Tracks traffic on fa0/0, fa1/0, fa2/0, Loopback0 |

> 📸 *Screenshot: Resource selection screen with selected items highlighted*

1. Check all items listed above
2. Click **Next**

---

## Step 6 — Review and Add Node

1. Review the node summary:

| Field | Value |
|---|---|
| Node Name | Toronto-CR1 |
| IP Address | 10.1.50.100 |
| Polling Method | SNMPv2c |
| Community String | DaVince-SNMP-RO |
| Resources | ICMP, CPU, Memory, Interfaces |

2. Click **Add Node**
3. SolarWinds begins polling immediately

> 📸 *Screenshot: Node summary confirmation screen*

---

## Step 7 — Verify Node is Being Monitored

1. Click **My Dashboards → Home** in the top navigation
2. Locate **Toronto-CR1** in the **All Nodes** widget
3. Confirm the node shows a **green** Up status

> 📸 *Screenshot: Home dashboard showing Toronto-CR1 with green Up status*

4. Click on **Toronto-CR1** to open the Node Details page
5. Confirm the following are populating with data:

| Section | Expected |
|---|---|
| Node Status | Up (green) |
| CPU Load | % value shown |
| Memory Usage | % value shown |
| Interfaces | fa0/0, fa1/0, fa2/0 listed with traffic graphs |
| Response Time | ms value shown |

> 📸 *Screenshot: Toronto-CR1 Node Details page with all resources populated*

---

## Step 8 — Rename Node (Optional but Recommended)

SolarWinds may display the node by IP. Set the correct hostname:

1. On the **Node Details** page click **Edit Node** (top right)
2. Under **Node Name** change to `Toronto-CR1`
3. Click **Submit**

> 📸 *Screenshot: Edit Node page with hostname field*

---

## Verification Summary

| Test | Result |
|---|---|
| SNMP test during Add Node wizard | ✅ Successful |
| Node shows Up status on dashboard | ✅ Green |
| CPU and Memory data populating | ✅ Confirmed |
| All interfaces visible | ✅ fa0/0, fa1/0, fa2/0, Loopback0 |
| Response time graphing | ✅ Active |

---

## Notes

- The same onboarding process applies to all remaining EVE-NG devices (Toronto-West-SW1, Toronto-East-SW2, R-37257, R-37258) — use each device's management IP and the same community string `DaVince-SNMP-RO`
- If the SNMP test fails during Step 4, verify pfSense has the static route for `192.168.10.0/24` → `10.1.50.100` and that UDP 161 is permitted from MON01 to the device subnet
- Loopback addresses (e.g. `1.1.1.1` for Toronto-CR1) can be added as a second node entry in SolarWinds for a more stable always-up monitoring endpoint

---

## References

- HL-SRV-003 — SolarWinds Observability Self-Hosted Deployment
- HL-NET-004-CFG-001 — EVE-NG Topology Device Configuration
- SolarWinds Orion Node Management: https://documentation.solarwinds.com
