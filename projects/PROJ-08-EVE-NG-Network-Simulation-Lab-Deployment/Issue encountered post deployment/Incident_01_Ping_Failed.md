# EVE-NG — VPC Ping Issue Resolution

---

## Issue Summary

After deploying EVE-NG and configuring two VPCs (`10.1.50.20` and `10.1.50.21`) connected via Cloud0 (pnet0) to VLAN50, pings from the VPCs to the lab gateway (`10.1.50.1`) and other VLAN devices were failing with **"host unreachable"** — despite correct IP addressing, gateway configuration, and confirmed EVE-NG host reachability.

---

## Root Cause

Linux kernel IP forwarding was disabled on the EVE-NG host. Without it, the host drops all transit traffic from simulated nodes instead of forwarding it to the real network.

```bash
# Confirmed disabled
cat /proc/sys/net/ipv4/ip_forward
# Output: 0
```

---

## Initial Fix (Temporary)

```bash
echo 1 > /proc/sys/net/ipv4/ip_forward
```

This enabled forwarding immediately but **did not survive a reboot** — the value reverted to `0` after the EVE-NG server restarted.

---

## Permanent Fix

Added the parameter to `/etc/sysctl.conf` so it persists across all future reboots:

```bash
echo "net.ipv4.ip_forward=1" >> /etc/sysctl.conf && sysctl -p
```

---

## Verification

```bash
# Confirm value is active
cat /proc/sys/net/ipv4/ip_forward
# Output: 1
```

Post-reboot check confirmed the value persisted. VPC pings to gateway, MON01, and DC01 all successful after fix.

---

## Key Takeaway

Always persist kernel parameters via `/etc/sysctl.conf` — writing to `/proc/sys/` is temporary and lost on reboot.
