# Lesson Learned - Same VLAN Virtual Machines Cannot Ping Each Other


# 1. Incident Summary

Two virtual machines deployed within the same VMware ESXi Port Group were unable to communicate with each other using ICMP ping.

The affected machines were connected to the same VLAN and subnet.

However, communication with devices located in different VLANs was successful.

This initially suggested that the issue was not related to routing or pfSense configuration.

---

# 2. Environment Details

## Network Architecture

```text
VMware ESXi
     |
     |
vSwitch-LAN
     |
     |
PG-INFRA
     |
     |
VLAN20-INFRA
     |
     |
10.1.20.0/24
```

---

## Affected Systems

| Device | IP Address | Port Group | VLAN |
|---|---|---|---|
| VM-MON01 | 10.1.20.20 | PG-INFRA | VLAN20 |
| VM-DC01 | 10.1.20.21 | PG-INFRA | VLAN20 |

---

# 3. Symptoms

Observed behavior:

- VM-MON01 could not ping VM-DC01.
- VM-DC01 could not ping VM-MON01.
- Both machines could ping the pfSense gateway.
- Both machines could access systems in other VLANs.
- Internet access was working.

Example:

Successful:

```text
VM-MON01
 |
 |
pfSense Gateway
 |
 |
Internet
```

Failed:

```text
VM-MON01
 |
 |
VM-DC01
```

---

# 4. Initial Troubleshooting

## Network Validation

Verified:

- IP addressing
- Subnet mask
- Default gateway
- VLAN assignment
- ESXi Port Group assignment

Result:

All network settings were correct.

---

# 5. Root Cause Analysis

## Root Cause

The issue was caused by Windows Firewall blocking inbound ICMP echo requests.

Windows systems allow outgoing ping traffic by default but may block incoming ping requests depending on firewall profile settings.

The virtual networking infrastructure was functioning correctly.

---

# 6. Resolution

The Windows network sharing settings were modified to allow network communication.

Configuration changed:

```
Control Panel
    |
Network and Sharing Center
    |
Advanced Sharing Settings
    |
Private Network Profile
```

Enabled:

```
File and Printer Sharing
```

After applying the changes:

- ICMP communication was restored.
- Virtual machines successfully communicated within the same VLAN.

![Advanced sharing](../Unable%20to%20ping%20between%20two%20servers%20in%20the%20same%20port%20group/Screenshots/Advanced%20sharing%20settings.JPG)


This can also be enabled via windows firewall

![Windows firewall](../Unable%20to%20ping%20between%20two%20servers%20in%20the%20same%20port%20group/Screenshots/Windows%20firewall%20settings.JPG)

---

# 7. Validation Testing

## Ping Test

Before:

![Ping Failed](../Unable%20to%20ping%20between%20two%20servers%20in%20the%20same%20port%20group/Screenshots/ping%20failed.JPG)


After:

![Ping Successful](../Unable%20to%20ping%20between%20two%20servers%20in%20the%20same%20port%20group/Screenshots/ping%20successful.JPG)

---

## Network Validation Checklist

| Test | Result |
|---|---|
| VM-to-Gateway communication | PASS |
| VM-to-VM same VLAN communication | PASS |
| Inter-VLAN communication | PASS |
| Internet access | PASS |
| pfSense routing | PASS |
| ESXi switching | PASS |

---

# 8. Lessons Learned

## Key Finding

Not every connectivity issue is caused by:

- VLAN configuration
- Firewall rules
- Routing
- ESXi networking

Endpoint security controls can prevent communication even when the network infrastructure is correctly configured.

---

## Troubleshooting Approach Improvement

Future troubleshooting should follow the OSI model:

```text
Layer 7
Application

Layer 6
Presentation

Layer 5
Session

Layer 4
Transport

Layer 3
Routing
(pfSense / VLANs)

Layer 2
Switching
(ESXi vSwitch / Port Groups)

Layer 1
Physical
```

For same VLAN communication:

Priority checks:

1. Host firewall
2. IP configuration
3. VLAN assignment
4. Port Group configuration
5. ESXi networking

---

# 9. Preventive Actions

Future VM deployments should include:

- Verify Windows Firewall profile.
- Enable required ICMP rules.
- Validate network discovery settings.
- Document firewall exceptions.
- Perform connectivity testing after deployment.

---

# 10. Standard Validation Procedure For New VMs

After deploying a new VM:

## Step 1

Verify IP:

```powershell
ipconfig
```

---

## Step 2

Test gateway:

```powershell
ping <gateway-ip>
```

Example:

```powershell
ping 10.1.20.1
```

---

## Step 3

Test same VLAN communication:

```powershell
ping <another-vm-ip>
```

---

## Step 4

Test DNS:

```powershell
nslookup google.com
```

---

## Step 5

Confirm firewall rules.

---

# Final Status

```text
RESOLVED

Root Cause:
Windows Firewall / Network Discovery settings

Resolution:
Enabled file and printer sharing

Impact:
No infrastructure changes required
```

---

# References

- VMware ESXi Virtual Networking Design
- pfSense VLAN Architecture
- Windows Firewall Configuration
- VLAN20 Infrastructure Design