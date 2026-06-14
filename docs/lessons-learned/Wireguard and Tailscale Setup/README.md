Two remote access technologies were evaluated: WireGuard VPN (self-hosted on pfSense) and Tailscale (mesh VPN overlay). 
WireGuard was successfully implemented for local network access but could not be extended to internet access due to the ISP assigning a Carrier-Grade NAT (CGNAT) address to the router's WAN interface. 
Tailscale was subsequently deployed as the production solution, as it establishes outbound-only connections that bypass CGNAT entirely without requiring port forwarding or a static public IP.

---

## 1. Extending WireGuard to Internet Access

### 1.1 Approach

To extend the tunnel beyond the local network, the standard approach is:

1. Forward UDP/51820 on the ISP router to the pfSense WAN address (`192.168.1.50`).
2. Update the client's `Endpoint` from the private LAN IP to the router's public WAN IP.
3. (Optionally) configure Dynamic DNS to handle a non-static public IP.

### 1.2 Issue Identified — Carrier-Grade NAT (CGNAT)

During implementation, port forwarding was configured correctly on the ISP router, but an external port-scan test (yougetsignal.com) reported the forwarded port as **closed**.

Investigation of the ISP router's WAN status revealed the WAN interface address was `10.160.168.23` — a private RFC 1918 address. This confirmed the router itself was not holding a public IP, but was instead sitting behind an additional NAT layer operated by the ISP (CGNAT).

### 1.3 Root Cause

```
Internet
   │
   ▼
ISP Carrier-Grade NAT (public IP held here — not visible to customer)
   │
   ▼
ISP Router (WAN: 10.160.168.23 — private)
   │
   ▼
pfSense (WAN: 192.168.1.50)
```

Because the customer-facing router does not hold a public-routable address, inbound port forwarding rules configured on it have no effect — there is no path for unsolicited inbound traffic from the internet to reach the router at all.

### 1.4 Resolution Path Considered

| Option | Outcome |
|---|---|
| Request public/static IP from ISP | Viable but dependent on ISP support and may incur cost |
| Cloud VPS as WireGuard relay | Technically viable, adds infrastructure overhead and ongoing cost |
| **Tailscale mesh VPN** | **Selected** — works through CGNAT via outbound-only connections, no cost, minimal config |

**Status: OBJ-03 not achievable via standard WireGuard + port forwarding under current ISP conditions.**

### 2. Implementation Steps

1. Created a Tailscale account (Tailnet).
2. Installed the Tailscale package on pfSense via **System → Package Manager**.
3. Authenticated pfSense to the Tailnet via **VPN → Tailscale → Authentication**.
4. Advertised all four VLAN subnets as routes from the pfSense node.
5. Approved the advertised subnet routes in the Tailscale admin console.
6. Created a firewall rule on the Tailscale interface permitting traffic from the Tailscale network to any destination.
7. Added permit rules on each VLAN interface allowing traffic sourced from the Tailscale subnet (`100.64.0.0/10`).
8. Installed the Tailscale client on the remote device and signed in with the same Tailnet account.
9. Verified subnet route acceptance and tested connectivity.

### 3. Result

The remote device successfully reached all pfSense VLAN gateways and target VMs from outside the local network, with no port forwarding, static IP, or DDNS configuration required. RDP sessions to all four VLAN VMs were established successfully.

**Status: OBJ-01 through OBJ-05 met.**

---

## 4. Lessons Learned

### 4.1 Technical

- **Always verify the WAN IP address class before designing remote access.** A private address (RFC 1918) on the router's WAN interface is a definitive indicator of CGNAT and should be checked at the start of any remote-access project, not after a failed port-forward test.
- **Port forwarding rules require a routable public IP upstream.** A correctly configured port-forward rule will silently fail if the device itself has no public address — there is no error message, only a failed connectivity test.
- **Firewall rules must be applied at every layer the traffic crosses.** Remote access traffic in this design traverses the VPN/Tailscale interface rule and the destination VLAN interface rule; missing either one results in partial connectivity (e.g., reaching the gateway but not the VM).
- **Outbound-initiated overlay networks (Tailscale) are a robust fallback for CGNAT environments**, and in many cases a simpler long-term solution than maintaining port forwards and DDNS records.
- **WireGuard's "Allowed IPs" field has context-dependent meaning** — on the server/peer definition it constrains which addresses a peer may use, while on the client it defines which subnets are routed through the tunnel. This distinction is a common source of misconfiguration.

### 4.2 Process

- **Test incrementally and locally before testing externally.** Validating WireGuard on the local network first isolated the configuration issues (firewall rules, RDP settings) from the external connectivity issue (CGNAT), preventing a compounded troubleshooting effort.
- **External port-scan tools (e.g., yougetsignal.com) are an efficient way to confirm or rule out ISP-side blocking** before assuming a local misconfiguration.
- **Document the failure path, not just the final solution.** The CGNAT discovery and the reasoning for pivoting to Tailscale demonstrate troubleshooting methodology and architectural decision-making — both directly relevant to NOC/Tier 2 and network engineering roles.

### 4.3 Recommendations for Future Iterations

- Periodically re-check ISP WAN IP allocation in case a public IP becomes available, which would allow a return to a self-hosted WireGuard model for full infrastructure control.
- Evaluate Tailscale ACL (access control list) features to apply more granular per-device, per-subnet access policies as the lab grows.
- Extend monitoring (SolarWinds) to alert on Tailscale node connectivity status as part of the existing observability stack.
