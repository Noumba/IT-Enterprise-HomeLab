# WireGuard VPN Setup — pfSense + Home Lab

## Introduction

The home lab environment runs VMware ESXi with pfSense as the core firewall and inter-VLAN router, segmenting all virtual machines across four isolated VLANs (MGMT, INFRA, DMZ, and LAB) in the `10.1.x.0/24` address space. The local PC is connected to the ISP router on the `192.168.1.0/24` network, which has no direct route to any of these internal VLAN subnets. To bridge this gap securely and enable full RDP and management access to all virtual machines from the local PC, we are deploying WireGuard VPN on pfSense. WireGuard will create an encrypted tunnel between the local PC and pfSense, assigning the PC a dedicated VPN IP (`10.200.0.2`) that pfSense will treat as a trusted client, granting it routed access to all internal VLANs without exposing any VM directly to the ISP network.

> **Goal:** Secure VPN access from my laptop to all home lab VLANs/VMs via pfSense.

---

## Prerequisites

| Item | Value |
|---|---|
| pfSense GUI | `https://10.1.1.1` |
| pfSense WAN IP | `192.168.1.143` |
| Admin access | Required |
| pfSense version | 2.5+ (WireGuard supported natively) |

---

## Network Reference

| Component | IP |
|---|---|
| pfSense WAN | 192.168.1.50 |
| pfSense VPN IP | 10.200.0.1 |
| Laptop VPN IP | 10.200.0.2 |
| VPN Tunnel Subnet | 10.200.0.0/24 |
| WireGuard Port | UDP 51820 |

---

## Step 1 — Install WireGuard on pfSense

1. Log into pfSense GUI → `https://10.10.10.1`
2. Go to **System → Package Manager → Available Packages**
3. Search for `WireGuard`
4. Click **Install** → **Confirm**
5. Wait for installation to complete

![Wireguard Installed](../Project-02-secure-vpn-access%20from-local-PC%20to%20homelab-VMs-via%20pfSense/Images/Install%20Wireguard.JPG)
---

## Step 2 — Create the WireGuard Tunnel

1. Go to **VPN → WireGuard → Tunnels tab**
2. Click **+ Add Tunnel**
3. Configure:

| Field | Value |
|---|---|
| Description | HomeLab-VPN |
| Listen Port | 51820 |
| Interface Keys | Click **Generate** |

4. Note down the **Server Public Key** — needed later on your laptop
5. Click **Save Tunnel**

![Create Wireguard Tunnel](../Project-02-secure-vpn-access%20from-local-PC%20to%20homelab-VMs-via%20pfSense/Images/Create%20Tunnel.JPG)
---

## Step 3 — Add Your Laptop as a Peer

In **VPN → WireGuard → Tunnels**, click **+ Add Peer**:

| Field | Value |
|---|---|
| Description | My-Laptop |
| Dynamic Endpoint | ✅ Checked |
| Public Key | *(paste laptop public key — generated in Step 7)* |
| Allowed IPs | 10.200.0.2/32 |

> **Note:** `/32` means only this one device is allowed to use this peer slot.

Click **Save Peer**

![Add laptop as Peer](../Project-02-secure-vpn-access%20from-local-PC%20to%20homelab-VMs-via%20pfSense/Images/Add%20Peer.JPG)
---

## Step 4 — Create the WireGuard Interface

1. Go to **Interfaces → Assignments**
2. Find the new interface (e.g. `tun_wg0`) → click **+ Add**
3. Click on the newly created interface (e.g. `OPT1`) and configure:

| Field | Value |
|---|---|
| Enable | ✅ Checked |
| Description | VPN_WG |
| IPv4 Config Type | Static IPv4 |
| IPv4 Address | 10.200.0.1/24 |

4. Click **Save** → **Apply Changes**

![Create Wireguard Interface](../Project-02-secure-vpn-access%20from-local-PC%20to%20homelab-VMs-via%20pfSense/Images/Wireguard%20Interface.JPG)
---

## Step 5 — Configure Firewall Rules

### 5a — WAN Rule (allow VPN traffic in)

**Firewall → Rules → WAN** → click **+ Add**:

| Field | Value |
|---|---|
| Action | Pass |
| Protocol | UDP |
| Destination | WAN Address |
| Destination Port | 51820 |
| Description | Allow WireGuard VPN |

Click **Save** → **Apply Changes**

![WAN Rule](../Project-02-secure-vpn-access%20from-local-PC%20to%20homelab-VMs-via%20pfSense/Images/WAN%20Rule%202.JPG)

![WAN Rule](../Project-02-secure-vpn-access%20from-local-PC%20to%20homelab-VMs-via%20pfSense/Images/WAN%20Rule.JPG)


### 5b — WireGuard Interface Rule (allow VPN clients to reach VLANs)

**Firewall → Rules → VPN_WG** → click **+ Add**:

| Field | Value |
|---|---|
| Action | Pass |
| Protocol | Any |
| Source | 10.200.0.0/24 |
| Destination | Any |
| Description | VPN clients access all VLANs |

Click **Save** → **Apply Changes**

![WG Interface Rule](../Project-02-secure-vpn-access%20from-local-PC%20to%20homelab-VMs-via%20pfSense/Images/WG%20Int-Rule.JPG)

### 5c — VLAN Rules (allow VPN subnet into each VLAN)

Repeat for each VLAN tab under **Firewall → Rules**:

| Field | Value |
|---|---|
| Action | Pass |
| Protocol | Any |
| Source | 10.200.0.0/24 |
| Destination | Any |
| Description | Allow VPN access |

| VLAN | Tab | pfSense Gateway |
|---|---|---|
| VLAN10 | MGMT | 10.10.10.1 |
| VLAN20 | INFRA | 10.10.20.1 |
| VLAN30 | DMZ | 10.10.30.1 |
| VLAN50 | LAB | 10.10.50.1 |

---

## Step 6 — Enable WireGuard Service

1. Go to **VPN → WireGuard → Settings tab**
2. Check **Enable WireGuard**
3. Click **Save**
4. Go to **Status → Services** → confirm `WireGuard` shows **Running**

![Enable Wireguard Service](../Project-02-secure-vpn-access%20from-local-PC%20to%20homelab-VMs-via%20pfSense/Images/Enable%20Wireguard%20Service.JPG)
---

## Step 7 — Configure Your Laptop (Windows)

### 7a — Install WireGuard Client

Download and install from: [https://wireguard.com/install](https://wireguard.com/install)

### 7b — Generate Laptop Keys

1. Open WireGuard app
2. Click **Add Tunnel → Create from scratch**
3. Your laptop's key pair is auto-generated

### 7c — Write the Tunnel Config

```ini
[Interface]
PrivateKey = <your-laptop-private-key>
Address = 10.200.0.2/24 **Wireguard Client IP**
DNS = pfSense LAN IP

[Peer]
PublicKey = <pfSense-server-public-key>
Endpoint = pfSense-WAN-IP:51820
AllowedIPs = 10.10.10.0/24, 10.10.20.0/24, 10.200.0.0/24 **VMs and Wireguard Client Network**
PersistentKeepalive = 25
```
![Wireguard Client](../Project-02-secure-vpn-access%20from-local-PC%20to%20homelab-VMs-via%20pfSense/Images/Wireguard%20Client%20setup.JPG)

### 7d — Copy Laptop Public Key to pfSense

1. In the WireGuard app, copy your **Public Key** shown at the top of the config
2. Go back to pfSense → **VPN → WireGuard → Edit Peer (My-Laptop)**
3. Paste the public key into the **Public Key** field
4. Click **Save**

---

## Step 8 — Enable RDP on Each VM

### Via PowerShell (run on each VM)

```powershell
# Enable RDP
Set-ItemProperty -Path 'HKLM:\System\CurrentControlSet\Control\Terminal Server' -Name "fDenyTSConnections" -Value 0

# Allow RDP through Windows Firewall
Enable-NetFirewallRule -DisplayGroup "Remote Desktop"
```

---

## Step 9 — Test the Connection

1. Open WireGuard app on your laptop
2. Click **Activate**
3. Status should show **Active** with a handshake timestamp

### RDP Test

Open **Remote Desktop Connection** (`mstsc`) and connect to each VM:

| VM | IP |
|---|---|
| MGMT VM | 10.10.10.100 |
| INFRA VM | 10.10.20.100 |
| DMZ VM | 10.10.30.100 |
| LAB VM | 10.10.50.100 |

---

## Optional — Create .rdp Shortcut Files

Save a `.rdp` file per VM for one-click access. Example `INFRA-VM.rdp`:

```
full address:s:10.10.20.100
username:s:Administrator
desktopwidth:i:1920
desktopheight:i:1080
session bpp:i:32
audiomode:i:0
```

---

## Troubleshooting

| Problem | Likely Cause | Fix |
|---|---|---|
| No handshake | WAN firewall rule missing | Re-check Step 5a |
| Handshake but no ping | WireGuard interface rule missing | Re-check Step 5b |
| Can reach pfSense, not VMs | VLAN rule missing | Re-check Step 5c |
| RDP refused | RDP not enabled on VM | Re-do Step 8 |
| RDP times out | Windows Firewall blocking port 3389 | Run `Enable-NetFirewallRule -DisplayGroup "Remote Desktop"` on VM |
| Wrong credentials | Use `.\Administrator` or `HOSTNAME\user` format | — |

---

## Traffic Flow Summary

```
Your Laptop (192.168.1.x)
        │
        │  WireGuard Tunnel (UDP 51820)
        ▼
pfSense WAN (192.168.1.50)
        │
        │  Routes to VLAN subnets
        ▼
pfSense VLAN Gateway (e.g. 10.10.20.1)
        │
        │  RDP Port 3389
        ▼
VM (e.g. 10.10.20.100)
```
