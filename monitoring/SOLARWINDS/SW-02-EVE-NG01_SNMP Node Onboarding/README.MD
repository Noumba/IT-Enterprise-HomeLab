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
| Toronto-East-Edge-R1 IP | 192.168.40.2 |
| SNMP Version | v2c |
| Community String | `SNMP-RO` |
| MON01 reachable from EER1 | Confirmed — ping 10.1.20.20 succeeds |
| pfSense static route added | 192.168.40.0/24 → 10.1.50.100 |
| SNMP port open | UDP 161 confirmed via |

---

## Step 1 — Log into Orion Web Console

1. Open a browser from any device on the lab network
2. Navigate to `https://10.1.20.20`
3. Log in with administrator credentials

![Login page](../SW-02-EVE-NG01_SNMP%20Node%20Onboarding/Images/login%20page.JPG)

---

## Step 2 — Navigate to Manage Nodes

1. Click **Settings** in the top navigation bar
2. Select **All Settings** from the dropdown
3. Under the **Node & Group Management** section, click **Manage Nodes**

![Navigate to manage nodes](../SW-02-EVE-NG01_SNMP%20Node%20Onboarding/Images/navigate_to_manage_nodes.JPG)

---

## Step 3 — Add New Node

1. Click the **Add Node** button (top left of the Manage Nodes page)
2. The **Add Node** wizard opens

![Add New Node](../SW-02-EVE-NG01_SNMP%20Node%20Onboarding/Images/Add%20node%20page.JPG)

---

## Step 4 — Enter Node Details

1. In the **Hostname or IP Address** field enter:
```
192.168.40.2
```
2. Under **Polling Method** select **SNMP**
3. Set **SNMP Version** to **SNMPv2c**
4. Enter the community string:
```
SNMP-RO
```
5. Leave all other fields at default
6. Click **Test** to verify SNMP connectivity

**Expected Test Result:**
```
Test Successful
```

![Enter Node Details](../SW-02-EVE-NG01_SNMP%20Node%20Onboarding/Images/snmp%20test.JPG)

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

![Select Resources](../SW-02-EVE-NG01_SNMP%20Node%20Onboarding/Images/Select%20Resources.JPG)

1. Check all items listed above
2. Click **Next**

---

## Step 6 — Review and Add Node

1. Review the node summary:

| Field | Value |
|---|---|
| Node Name | Toronto-East-Edge-R1 |
| IP Address | 192.168.40.2 |
| Polling Method | SNMPv2c |
| Community String | SNMP-RO |
| Resources | ICMP, CPU, Memory, Interfaces |

2. Click **Add Node**
3. SolarWinds begins polling immediately

![Add Node](../SW-02-EVE-NG01_SNMP%20Node%20Onboarding/Images/Add%20node.JPG)

---

## Step 7 — Verify Node is Being Monitored

1. Click **My Dashboards → NPM Summary** in the top navigation
2. Expand the grouping and confirm node was added
3. Confirm the node shows a **green** Up status

![Node Added](../SW-02-EVE-NG01_SNMP%20Node%20Onboarding/Images/node%20added.JPG)

4. Click on **Toronto-East-Edge-R1** to open the Node Details page
5. Confirm the following are populating with data:

| Section | Expected |
|---|---|
| Node Status | Up (green) |
| CPU Load | % value shown |
| Memory Usage | % value shown |
| Interfaces | fa0/0 |
| Response Time | ms value shown |

![Node Details](../SW-02-EVE-NG01_SNMP%20Node%20Onboarding/Images/Node%20details.JPG)

---

## Verification Summary

| Test | Result |
|---|---|
| SNMP test during Add Node wizard | ✅ Successful |
| Node shows Up status on dashboard | ✅ Green |
| CPU and Memory data populating | ✅ Confirmed |
| All interfaces visible | ✅ fa0/0, Loopback0 |
| Response time graphing | ✅ Active |

---

## Notes

- The same onboarding process applies to all remaining EVE-NG devices (Toronto-West-SW1, Toronto-East-SW2) — use each device's management IP and the same community string `SNMP-RO`
- If the SNMP test fails during Step 4, verify connectivity via ping and that UDP 161 is permitted from MON01 to the device subnet
- Loopback addresses (e.g. `1.1.1.1`) can be added as a second node entry in SolarWinds for a more stable always-up monitoring endpoint

---

## References

- [SolarWinds Observability Self-Hosted Deployment](../SW-01-SolarWinds-Observability-2025.4.0-Deployment/README.MD)
- [EVE-NG01 Lab](../../../eve-ng%20labs/EVE-NG01-Core-Access%20Network%20Simulation%20Lab/README.MD)