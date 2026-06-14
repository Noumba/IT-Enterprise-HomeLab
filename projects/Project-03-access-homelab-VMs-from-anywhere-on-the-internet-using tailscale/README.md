# Tailscale Setup — pfSense (CGNAT Workaround)

## Introduction

We experienced a drawback while using Wireguard in pfSense as we were unable to access our HomeLab VMs via the internet. The ISP router's WAN interface shows `10.160.168.23`, a private IP address, confirming the home lab is behind Carrier-Grade NAT (CGNAT). This means the ISP does not assign a public IP to the home network, so traditional port forwarding for WireGuard cannot work and incoming internet traffic never reaches the ISP router. 

Tailscale solves this by establishing an **outbound** connection from pfSense to Tailscale's coordination servers, creating a secure mesh VPN that works through CGNAT without any port forwarding. Installing Tailscale on pfSense will allow remote access to all home lab VLANs and VMs from anywhere on the internet.

---

## Prerequisites

| Item | Value |
|---|---|
| pfSense GUI | `https://10.10.10.1` |
| Tailscale account | Free — sign up at tailscale.com |
| pfSense version | 23.09+ (for Tailscale package support) |

---

## Step 1 — Create a Tailscale Account

1. Go to [https://tailscale.com](https://tailscale.com)
2. Sign up using Google, Microsoft, GitHub, or email
3. This creates your **Tailnet** (your private mesh network)

---

## Step 2 — Install Tailscale Package on pfSense

1. Log into pfSense GUI → `https://10.10.10.1`
2. Go to **System → Package Manager → Available Packages**
3. Search for `Tailscale`
4. Click **Install** → **Confirm**

![Install Tailscale](../Project-03-access-homelab-VMs-from-anywhere-on-the-internet-using%20tailscale/Images/Install.JPG)
---

## Step 3 — Authenticate pfSense with Tailscale

1. Go to **VPN → Tailscale**
2. Click the **Authentication** tab
3. **Login to Tailscale** and generate an authentication key. 
4. Copy the newly generated AUth key
5. Go to **VPN → Tailscale -> Authentication**
6. Paste the Auth Key on the Pre-auth Key field
7. Save the changes.

![Generate Auth Key](../Project-03-access-homelab-VMs-from-anywhere-on-the-internet-using%20tailscale/Images/tailscale%20auth%20key.JPG)
![Paste Auth Key](../Project-03-access-homelab-VMs-from-anywhere-on-the-internet-using%20tailscale/Images/Auth%20Key.JPG)

---

## Step 4 — Advertise Your VLAN Subnets as Routes

By default, Tailscale only gives access to pfSense itself. To reach the VMs, we need to advertise our home lab subnets.

1. In pfSense, go to **VPN → Tailscale → Settings**
2. Find **Advertise Routes** (subnet routes field)
3. Add all VLAN subnets:

![Advertise Routes](../Project-03-access-homelab-VMs-from-anywhere-on-the-internet-using%20tailscale/Images/advertise%20routes.JPG)

4. Click **Save**

---

## Step 5 — Approve Subnet Routes in Tailscale Admin Console

1. Go to [https://login.tailscale.com/admin/machines](https://login.tailscale.com/admin/machines)
2. Find your pfSense device in the list
3. Click the **⋯** menu → **Edit route settings**
4. Toggle **ON** for each subnet:

![Approve Subnet Routes](../Project-03-access-homelab-VMs-from-anywhere-on-the-internet-using%20tailscale/Images/approve%20subnets.JPG)

![Confirm Subnets](../Project-03-access-homelab-VMs-from-anywhere-on-the-internet-using%20tailscale/Images/subnets.JPG)

5. Save

---

## Step 6 — Configure Firewall Rule for Tailscale Interface

1. Go to **Firewall → Rules**
2. Click the **Tailscale** tab (new interface created automatically)
3. Click **+ Add**:

| Field | Value |
|---|---|
| Action | Pass |
| Protocol | Any |
| Source | Tailscale net |
| Destination | Any |
| Description | Allow Tailscale clients to access VLANs |

4. Click **Save** → **Apply Changes**

---

## Step 7 — Add VLAN Firewall Rules (Allow Tailscale Source)

Repeat for each VLAN tab under **Firewall → Rules**:

| Field | Value |
|---|---|
| Action | Pass |
| Protocol | Any |
| Source | Tailscale subnet (e.g. 100.64.0.0/10) |
| Destination | Any |
| Description | Allow Tailscale access |

| VLAN | Tab |
|---|---|
| VLAN10 | MGMT |
| VLAN20 | INFRA |
| VLAN30 | DMZ |
| VLAN50 | LAB |

> Tailscale's default address range is `100.64.0.0/10`. You can confirm your exact range in the Tailscale admin console under **DNS** settings.

---

## Step 8 — Install Tailscale on Your Laptop

1. Download from [https://tailscale.com/download](https://tailscale.com/download)
2. Install and sign in with the **same Tailscale account** used on pfSense
3. Your laptop now appears as a second device in your Tailnet

---

## Step 9 — Enable Subnet Routing on Laptop Client

By default, the laptop client may not use advertised subnet routes.

### Windows
1. Right-click the Tailscale tray icon
2. Ensure **"Use exit node"** is off (not needed)
3. Subnet routes are accepted automatically once approved in Step 5

---

## Step 10 — Test the Connection

From your laptop (on any network — home, mobile hotspot, etc.):

```bash
# Check Tailscale status
tailscale status

# Ping pfSense VLAN gateways
ping 10.10.10.1
ping 10.10.20.1

### RDP Test

Open **Remote Desktop Connection** (`mstsc`) and connect to:

| VM | IP |
|---|---|
| MGMT VM | 10.10.10.100 |
| INFRA VM | 10.10.20.100 |
| DMZ VM | 10.10.30.100 |
| LAB VM | 10.10.50.100 |

---

## Troubleshooting

| Problem | Likely Cause | Fix |
|---|---|---|
| pfSense not appearing in Tailscale admin | Auth not completed | Re-do Step 3 |
| Can ping pfSense, not VMs | Subnet routes not approved | Re-check Step 5 |
| Subnet routes approved, still no ping | VLAN firewall rule missing | Re-check Step 7 |
| `tailscale status` shows no subnet routes | Routes not advertised | Re-check Step 4 |
| RDP refused | RDP not enabled on VM | Enable via `Enable-NetFirewallRule -DisplayGroup "Remote Desktop"` |

---

## How Traffic Flows

```
Your Laptop (Tailscale client — anywhere on internet)
        │
        │  Encrypted WireGuard tunnel via Tailscale relay
        ▼
pfSense (Tailscale node — outbound connection only)
        │
        │  Routes to advertised VLAN subnets
        ▼
VM (e.g. 10.10.20.100)
```

---

## Why This Works Behind CGNAT

Tailscale establishes the connection **outbound** from pfSense to Tailscale's coordination servers — no inbound port forwarding required. This bypasses the CGNAT limitation entirely (`10.160.168.23` WAN IP is irrelevant to this setup).
