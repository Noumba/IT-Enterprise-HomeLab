# Windows Server 2022 Deployment on VMware ESXi

**Author:** Leonard Noumba  
**Platform:** VMware ESXi 8.0 U2  
**Guest OS:** Microsoft Windows Server 2022 (64-bit) Standard Evaluation  
**Document Version:** 1.0  
**Date:** June 2026

---

## Overview

This guide walks through the full end-to-end deployment of Windows Server 2022 as a virtual machine (VM) on VMware ESXi 8.0 U2. It covers VM creation using the ESXi web client, OS installation from an ISO, and initial post-installation configuration including network setup and hostname assignment.

---

## Prerequisites

Before you begin, ensure the following are in place:

| Requirement | Details |
|---|---|
| ESXi Host | VMware ESXi 8.0 U2 (or later) |
| Datastore | At least 60 GB free space |
| Windows Server 2022 ISO | Uploaded to the ESXi datastore (ISO folder) |
| Browser | Modern browser with access to the ESXi web client |
| Network | A virtual switch and port group configured on ESXi |

> **ISO Used in this lab:** `SERVER_EVAL_x64FRE_en-us.iso` (4.7 GB) — Windows Server 2022 Standard Evaluation

---

## Part 1 — Create the Virtual Machine in ESXi

### Step 1.1 — Launch the New VM Wizard

1. Log in to the **ESXi web client** (`https://<your-esxi-ip>/ui`)
2. Navigate to **Virtual Machines** in the left sidebar
3. Click **Create / Register VM**
4. Select **Create a new virtual machine** and click **Next**

---

### Step 1.2 — Name the VM and Select Guest OS

Configure the VM identity and operating system type:

| Field | Value Used |
|---|---|
| VM Name | `SolarWinds VM` *(rename to your preference)* |
| Compatibility | ESXi 8.0 U2 virtual machine |
| Guest OS Family | Windows |
| Guest OS Version | Microsoft Windows Server 2022 (64-bit) |


> **Tip:** Enabling **Windows Virtualization Based Security** is optional for lab environments. For production, consider enabling it for enhanced kernel protection.

Click **Next** to proceed.

---

### Step 1.3 — Select Storage

1. Choose your datastore (e.g., `datastore1`)
2. Click **Next**

---

### Step 1.4 — Customize VM Hardware

Configure virtual hardware resources to match your workload. The following settings were used in this deployment:

| Hardware | Value |
|---|---|
| vCPUs | 4 |
| Memory | 16 GB |
| Hard Disk 1 | 40 GB |
| Network Adapter 1 | `vcentertnetwork` (VMXNET 3) |
| SCSI Controller | VMware Paravirtual |
| SATA Controller | New SATA controller |

**Attach the Windows Server 2022 ISO:**

1. Under **CD/DVD Drive**, select **Datastore ISO file**
2. Click **Browse** to open the Datastore browser

3. Navigate to the **ISO** folder in the datastore
4. Select `SERVER_EVAL_x64FRE_en-us.iso` and click **Select**

5. Ensure **Connect at power on** is checked for the CD/DVD drive

Click **Next** to proceed.

---

### Step 1.5 — Review and Finish

The **Ready to Complete** screen shows a full summary of the VM configuration before it is created:

Verify all settings are correct:

- **Name:** SolarWinds VM
- **Datastore:** datastore1
- **Guest OS:** Microsoft Windows Server 2022 (64-bit)
- **vCPUs:** 4
- **Memory:** 16 GB
- **Hard Disk:** 40 GB
- **Network Adapter:** VMXNET 3

Click **Finish** to create the VM.

---

## Part 2 — Install Windows Server 2022

### Step 2.1 — Power On and Boot to ISO

1. From the **Virtual Machines** list, select your newly created VM
2. Click **Power On**
3. Open the **VM console** to interact with the installation

The VM will boot from the attached ISO

---

### Step 2.2 — Accept the License Agreement

The Windows Server Setup wizard will start. When presented with the **License Terms** screen:

1. Read through the Microsoft Software License Terms
2. Check **"I accept the Microsoft Software License Terms"**
3. Click **Next**

---

### Step 2.3 — Select Installation Drive

On the **"Where do you want to install the operating system?"** screen:

1. Select **Drive 0 Unallocated Space** (40 GB)
2. Click **Next** — Windows will automatically partition the drive

> **Note:** You can also use **New**, **Format**, or **Delete** to manage partitions manually. For a clean installation on a fresh VM, selecting the unallocated space and clicking Next is sufficient.

---

### Step 2.4 — Windows Installation Progress

Windows Server 2022 will now copy files and install. This process typically takes **5–15 minutes** depending on host performance. The VM will reboot automatically.

After reboot, the Windows logo boot screen confirms the OS is loading:

---

### Step 2.5 — Initial Setup — Set Administrator Password

After the first boot:

1. You will be prompted to set a password for the built-in **Administrator** account
2. Enter a strong password (minimum 8 characters, mixed case, numbers, symbols)
3. Confirm and press **Finish**

---

### Step 2.6 — Windows Lock Screen

Once setup completes, the Windows Server 2022 lock screen appears:

Press **Ctrl + Alt + Delete** (in the VM console, use the **Send Ctrl+Alt+Del** button) to unlock and log in with the Administrator credentials.

---

### Step 2.7 — Windows Server Desktop

A successful login lands you on the Windows Server 2022 desktop:

The watermark in the bottom-right corner confirms:
- **Edition:** Windows Server 2022 Standard Evaluation
- **Build:** 20348.fe_release (21H2)
- **Windows License valid for 180 days**

---

## Part 3 — Post-Installation Configuration

### Step 3.1 — Configure Network (TCP/IP)

By default, the network adapter may be set to DHCP. To verify or set a static IP:

1. Right-click the **Network** icon in the taskbar → **Open Network & Internet Settings**
2. Click **Change adapter options** → Right-click **Ethernet0** → **Properties**

3. Select **Internet Protocol Version 4 (TCP/IPv4)** → click **Properties**

For a **DHCP** setup (default lab configuration):
- Select **"Obtain an IP address automatically"**
- Select **"Obtain DNS server address automatically"**

For a **static IP** (recommended for servers):
- Select **"Use the following IP address"**
- Enter your IP, Subnet Mask, and Default Gateway
- Enter your DNS server addresses
- Click **OK** → **Close**

---

### Step 3.2 — Explore Windows Server Tools

Windows Server 2022 comes with a full suite of management tools accessible from the Start menu. Right-clicking the Start button reveals quick access to key tools:

Key tools available:
- **Server Manager** — Add roles and features, manage the server
- **Windows PowerShell** / **PowerShell ISE** — Scripting and automation
- **Windows Administrative Tools** — MMC snap-ins
- **Device Manager** — Hardware management
- **Disk Management** — Volume and partition management
- **Event Viewer** — System and application logs
- **Remote Desktop Connection** — Connect to other servers
- **Task Manager** — Monitor processes and performance

---

### Step 3.3 — Rename the Server

Renaming the server to a meaningful hostname is an important step before joining a domain or deploying any services:

1. Right-click **Start** → **System** (or go to Settings → System → About)
2. Under **Device specifications**, click **Rename this PC**
3. Enter your desired hostname (e.g., `SOLARWINDSLAB`)
4. Click **Next** → **Restart now**

After the restart, the server will boot with the new hostname.

**System details confirmed in this deployment:**

| Property | Value |
|---|---|
| Device Name (Before) | WIN-C44JG8Q0BTG |
| New Hostname | SOLARWINDSLAB |
| Processor | Intel(R) Core(TM) i5-8500T @ 2.10GHz |
| Edition | Windows Server 2022 Standard Evaluation |
| Version | 21H2 |
| OS Build | 20348.587 |
| Installed On | 1/10/2026 |

---

## Part 4 — Final Verification

After renaming and rebooting, log back in to confirm the deployment is complete:

The VM console shows the fully deployed Windows Server 2022 running inside ESXi with the correct VM name (`SolarWinds VM`) visible in the console title bar.

---

## Summary — Deployment Checklist

| Step | Task | Status |
|---|---|---|
| 1 | Create VM in ESXi (name, guest OS, storage, hardware) | ✅ |
| 2 | Attach Windows Server 2022 ISO from datastore | ✅ |
| 3 | Boot VM and launch Windows Setup | ✅ |
| 4 | Accept license agreement | ✅ |
| 5 | Select installation drive (40 GB unallocated) | ✅ |
| 6 | Complete installation and set Administrator password | ✅ |
| 7 | Log in to Windows Server 2022 desktop | ✅ |
| 8 | Configure network adapter (DHCP or static IP) | ✅ |
| 9 | Rename server hostname | ✅ |
| 10 | Verify OS and build information | ✅ |

---

## Recommended Next Steps

After completing this base deployment, consider the following:

- **Install VMware Tools** — Improves VM performance and enables guest OS integration
- **Activate Windows** — Use a KMS server or enter a product key to activate beyond the 180-day evaluation
- **Join Active Directory Domain** — If deploying in an enterprise lab
- **Enable Remote Desktop (RDP)** — For remote management without the ESXi console
- **Install server roles** via **Server Manager** — e.g., DNS, DHCP, IIS, Active Directory DS
- **Configure Windows Firewall** — Restrict inbound/outbound access as appropriate
- **Enable automatic updates** or configure a WSUS server for patching

---

*Documentation by @tech.with.leo5 | TikTok & YouTube*
