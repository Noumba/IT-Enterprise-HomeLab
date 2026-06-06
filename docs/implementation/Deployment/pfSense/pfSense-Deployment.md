# pfSense Deployment Guide on VMware ESXi 8.x

## Document Information

| Item | Value |
|--------|--------|
| Document Name | pfSense Deployment Guide on VMware ESXi 8.0.3 |
| Version | 1.0 |
| Platform | VMware ESXi 8.0.3 |
| Firewall | pfSense CE |
| Purpose | Firewall, Routing, VLAN Segmentation, and Network Security |

---

# 1. Introduction

This document provides a complete step-by-step guide for deploying pfSense as a virtual firewall on VMware ESXi 8.0.3.

The deployment is designed for:

- Home Labs
- Enterprise Labs
- Small Business Networks
- VMware Learning Environments
- Network Engineering Training

Upon completion, pfSense will provide:

- Firewall Services
- NAT
- Inter-VLAN Routing
- DHCP Services
- DNS Services
- VPN Services
- Internet Access Control
- Network Segmentation

---

# 2. Architecture Overview

## Logical Topology

```text
                              Internet
                                  |
                                  |
                          ISP Router / ONT
                                  |
                          WAN Network (WAN NIC)
                                  |
                      +----------------------+
                      |      pfSense VM      |
                      |   VMware ESXi Host   |
                      +----------------------+
                           |            |
                           |            |
                                      LAN NIC
                                        |
                                        |
                              VLAN Trunk Network
                                        |
       +------------+------------+------------+------------+
       |            |            |            |            |
    VLAN 10      VLAN 20      VLAN 30      VLAN 40      VLAN 50
   Management    Servers       Users       Voice         DMZ
```

---

# 3. Prerequisites

## VMware ESXi Requirements

- VMware ESXi 8.x installed
- Datastore configured
- Management Network configured
- Administrator access available

## Recommended Resources

| Resource | Recommended |
|-----------|------------|
| vCPU | 2-4 |
| Memory | 8 GB |
| Disk | 40 GB |
| Network Adapters | 2 |
| Adapter Type | VMXNET3 |

---

# 4. VMware Networking Design

## Port Groups

### Management Network

Used for:

- ESXi Management
- vCenter Management

### WAN Port Group

```text
PG-WAN
```

Purpose:

```text
Internet Connectivity
```

### LAN Trunk Port Group

```text
PG-LAN-TRUNK
```

Purpose:

```text
Internal Network Transport
```

Carries:

- VLAN 10
- VLAN 20
- VLAN 30
- VLAN 40
- VLAN 50
- VLAN 99

---

# 5. Upload pfSense ISO

1. Open ESXi Host Client.
2. Navigate to:

```text
Storage
└── Datastore Browser
    └── Upload
```

3. Upload the pfSense ISO.
![Upload ISO to ESXi](../../../../diagrams-images/deployments/pfSense/Upload%20ISO.JPG)
---

# 6. Create the Virtual Machine

Navigate to:

```text
Virtual Machines
└── Create/Register VM
```

Select:

```text
Create a New Virtual Machine
```
![Create VM](../../../../diagrams-images/deployments/pfSense/create%20VM.JPG)
---

# 7. Virtual Machine Configuration

## General

| Setting | Value |
|----------|---------|
| Name | PFSENSE-01 |
| Compatibility | ESXi 8.x |
| Guest OS Family | Other |
| Guest OS Version | FreeBSD 14 (64-bit) |

![VM Name and OS](../../../../diagrams-images/deployments/pfSense/vm%20name%20and%20OS.JPG)

## CPU

```text
1 vCPU
```

Recommended:

```text
2 vCPU
```

## Memory

```text
1024 MB
```

Recommended:

```text
2048 MB
```

## Hard Disk

```text
8 GB
```

Provisioning:

```text
Thin Provision
```

---

# 8. Configure Network Adapters

## Adapter 1 (WAN)

| Setting | Value |
|----------|---------|
| Port Group | PG-WAN |
| Adapter Type | VMXNET3 |

## Adapter 2 (LAN)

| Setting | Value |
|----------|---------|
| Port Group | PG-LAN-TRUNK |
| Adapter Type | VMXNET3 |

![VM Settings](../../../../diagrams-images/deployments/pfSense/VM%20Settings.JPG)


---

# 9. Mount Installation Media

Select:

```text
Datastore ISO File
```

Choose:

```text
pfSense-CE.iso
```

Enable:

```text
Connect At Power On
```

![VM Settings](../../../../diagrams-images/deployments/pfSense/mount%20ISO%20to%20VM.JPG)

---

# 10. Install pfSense

1. Power on VM.
2. Open Console.
3. Select:

```text
Install
```

4. Accept License Agreement.
![License](../../../../diagrams-images/deployments/pfSense/license%20agreement.JPG)


5. Select WAN and LAN Interfaces and Proceed
![WAN Interface](../../../../diagrams-images/deployments/pfSense/select%20WAN%20Interface.JPG)

![LAN Interface](../../../../diagrams-images/deployments/pfSense/Select%20LAN%20Interface.JPG)

Click OK and proceed with the installation.

6. Ensure the WAN and LAN interface Assignments are correct and Proceed
![Interface Confirmation](../../../../diagrams-images/deployments/pfSense/Interface%20Assignment%20Confirmation.JPG)


7. Proceed and Install pfSense
![Start Install CE](../../../../diagrams-images/deployments/pfSense/Start%20Install.JPG)

8. Select the File System Type and the Partition Scheme and Proceed
![File System Type and Partition Scheme](../../../../diagrams-images/deployments/pfSense/File%20System%20type.JPG)

9. Confirm the Installation Disk and Proceed
![Confirm Disk](../../../../diagrams-images/deployments/pfSense/Confirm%20Disk.JPG)


10. Select Software version (Current Stable Version) and Proceed
![Confirm Disk](../../../../diagrams-images/deployments/pfSense/select%20software%20version.JPG)


11. Installation Completed
![Installation Complete](../../../../diagrams-images/deployments/pfSense/Installation%20Done.JPG)


12. Disconnect ISO.
9. Reboot.
![pfSense DCUI screen](../../../../diagrams-images/deployments/pfSense/DCUI%20screen.JPG)
---


# 11. Access the Web Interface

Connect a workstation VM to the LAN network.

Configure:

```text
IP Address : 192.168.1.100
Subnet     : 255.255.255.0
Gateway    : 192.168.1.1
```

Browse to:

```text
https://LAN_IP
```

---

# 12. Default Login

```text
Username : admin
Password : pfsense
```

---

# 13. Initial Setup Wizard

## Hostname

```text
pfsense01
```

## Domain

```text
homelab.local
```

## DNS Servers

```text
1.1.1.1
8.8.8.8
```

## Time Zone

Select your local timezone.

## WAN Configuration

### DHCP

```text
Automatic ISP Assignment
```

### Static

```text
Manual Configuration
```

---