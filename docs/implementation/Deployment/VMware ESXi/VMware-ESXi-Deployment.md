# VMware ESXi 8.0.3 Deployment Guide (HP Bare Metal Servers)

## 1. Overview

This document provides a complete end-to-end guide for installing VMware ESXi 8.0 U3 (8.0.3) on HP bare-metal servers. It is designed for lab environments, enterprise deployments, and virtualization foundations.

The guide covers:
- Pre-installation checks
- BIOS/UEFI configuration
- HP firmware considerations
- ESXi installation steps
- Post-install configuration
- Troubleshooting common HP-specific issues

---

## 2. Architecture Assumptions

This guide assumes:
- HP ProLiant server (Gen9 / Gen10 / Gen10 Plus / Gen11)
- Single or dual NIC setup
- Local SSD or RAID storage (Smart Array or HBA mode)
- VMware ESXi 8.0.3 ISO (preferably HP Custom Image)
- USB boot or iLO Virtual Media access

---

## 3. Prerequisites

### 3.1 Hardware Requirements

- Minimum 2 CPU cores (4+ recommended)
- Minimum 8 GB RAM (16 GB+ recommended)
- Minimum 32 GB boot device (SSD or RAID volume recommended)
- Supported NICs (Broadcom / Intel recommended)

> ⚠ HP Note: ESXi 8 is strict on HCL compliance. Unsupported NICs may fail installation or lose network post-boot.

---

### 3.2 Firmware Requirements (Critical for HP)

Before installation, update via **HPE SPP (Service Pack for ProLiant)**:

- BIOS latest version
- iLO firmware updated
- Smart Array controller firmware updated
- NIC firmware updated

This prevents:
- Boot failures
- PSOD (Purple Screen of Death)
- Missing storage devices

---

### 3.3 BIOS / UEFI Configuration

Enter BIOS (F9 during boot on HP servers) and configure:

#### Boot Mode
- Set to **UEFI Mode ONLY**
- Disable Legacy BIOS

#### CPU Settings
- Enable Intel VT-x / AMD-V
- Enable VT-d / IOMMU

#### Power Management
- Set to **Maximum Performance**

#### Secure Boot
- Disable Secure Boot (recommended for lab setups)

#### Storage Controller Mode
- Set Smart Array to:
  - RAID mode (recommended for production)
  - HBA mode (for lab passthrough environments)

---

## 4. Preparing ESXi Installation Media

### USB Boot

Use Rufus:
- Select ESXi 8.0.3 ISO
- Partition scheme: MBR
- Target system: UEFI or BIOS
![Rufus ESXi 8.0.3 Media](../../../../diagrams-images/deployments/VMware%20deployment/Rufus.JPG)

### How to download Free ESXi 8.0 Update 3e
- Log in to Broadcom Support Portal
- Go to My Downloads
- Click on the box that says "Free Software Downloads available HERE"
![Download ESXi 8.0.3](../../../../diagrams-images/deployments/VMware%20deployment/download%20vmware%20esxi.jpg)

- Search for "VMware vSphere Hypervisor"
- Click "I agree to the Terms and Conditions " and download the ISO
---

## 5. ESXi 8.0.3 Installation Steps

### Step 1: Boot Installer

- Boot from USB media
- Wait for ESXi installer to load
![Boot from Installer](../../../../diagrams-images/deployments/VMware%20deployment/Boot%20from%20Installer.JPG)
![Loading ESXi](../../../../diagrams-images/deployments/VMware%20deployment/Loading%20esxi.JPG)
---

### Step 2: Start Installation

- Press **Enter** to begin installation
![Start Installation](../../../../diagrams-images/deployments/VMware%20deployment/Begin%20Installation.JPG)

- Press **F11** to accept EULA
![Accept License Agreement](../../../../diagrams-images/deployments/VMware%20deployment/License%20agreement.JPG)
---

### Step 3: Select Installation Disk

- Choose local SSD / RAID logical drive
- Avoid USB drives for production installs
![Select Installation Disk](../../../../diagrams-images/deployments/VMware%20deployment/Select%20Disk.JPG)
---

### Step 4: Keyboard Layout

- Select preferred layout (US recommended)

---

### Step 5: Root Password

Set a strong root password:
- Minimum 8 characters
- Include uppercase, lowercase, numbers, symbols
![Set root password](../../../../diagrams-images/deployments/VMware%20deployment/root%20password.JPG)
---

### Step 6: Confirm Installation

- Press **F11**
- ESXi will partition disk and install base system
![Confirm Install](../../../../diagrams-images/deployments/VMware%20deployment/confirm%20installation.JPG)

![Installation in Progress](../../../../diagrams-images/deployments/VMware%20deployment/Installation%20progress.JPG)
---

### Step 7: Reboot

- Remove installation media
- Reboot system
![Installation Completed](../../../../diagrams-images/deployments/VMware%20deployment/Installation%20Complete.JPG)
---

## 6. First Boot Configuration

After installation completes:

### 6.1 DCUI Screen

You will see the Direct Console UI:

- DHCP-assigned IP OR static IP prompt
![DCUI Screen](../../../../diagrams-images/deployments/VMware%20deployment/DCUI%20Screen.JPG)
---

### 6.2 Configure Management Network

Press **F2 → Configure Management Network**

Set:
- Static IP address
- Subnet mask
- Default gateway
- DNS servers
- Hostname

![Configure ESXi](../../../../diagrams-images/deployments/VMware%20deployment/DCUI%20Screen.JPG)
---

### 6.3 Test Management Access

From another machine:

```bash
https://<ESXi-IP>