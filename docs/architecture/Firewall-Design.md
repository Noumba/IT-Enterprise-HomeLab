# Firewall Design Overview

## 1. Introduction

A firewall is a critical component of any secure network architecture. It acts as a barrier between trusted and untrusted networks, enforcing security policies that control inbound and outbound traffic based on predefined rules.

This document outlines the firewall design for the lab environment, focusing on segmentation, security policies, traffic control, and scalability. The design follows best practices used in enterprise network environments to ensure security, visibility, and performance.

---

## 2. Design Objectives

The firewall design is built around the following objectives:

- Protect internal network resources from unauthorized access
- Segment network traffic into secure zones
- Enable controlled internet access for internal users
- Provide secure remote access capabilities
- Support logging, monitoring, and traffic inspection
- Ensure scalability for future expansion (cloud and hybrid integration)

---

## 3. Firewall Placement in Network Architecture

The firewall is positioned at the network edge, between the external network (Internet/ISP) and the internal LAN.

### Logical Placement

```
INFRA VM (10.1.20.100)
    │  Layer 2 — tagged VLAN20 on PG-INFRA
    ▼
vSwitch-LAN (ESXi)
    │  802.1Q trunk frame, VLAN20 tag preserved
    ▼
pfSense vtnet1 (LAN trunk interface)
    │  pfSense routes packet; source NAT applied
    ▼
pfSense vtnet0 (WAN: 192.168.1.50)
    │  Masqueraded as 192.168.1.50
    ▼
ISP Router (192.168.1.1)
    │
    ▼
INTERNET
```

---

## 4. Network Segmentation (Zones)

The network is divided into security zones to enforce isolation and control traffic flow.

### 4.1 WAN (Untrusted Zone)
- Internet-facing interface
- All inbound traffic is blocked by default
- Only explicitly allowed services are permitted

### 4.2 LAN (Trusted Zone)
- Internal users and devices
- Full access to approved internal services
- Controlled outbound internet access

### 4.3 DMZ (Demilitarized Zone)
- Hosts public-facing services (e.g., web server, DNS, VPN portal)
- Limited access to internal LAN
- Strict inbound and outbound rules applied

### 4.4 Management Zone
- Used for administrative access to network devices
- Restricted to IT/admin users only
- Access via secure protocols (SSH, HTTPS, VPN)

---

## 5. Firewall Policy Design

### 5.1 Default Policy

- **Inbound traffic:** DENY by default
- **Outbound traffic:** ALLOW with restrictions
- **Inter-zone traffic:** DENY unless explicitly permitted

---

### 5.2 Key Rules

| Rule ID | Source Zone | Destination Zone | Service | Action | Description |
|----------|------------|------------------|---------|--------|-------------|
| 1 | LAN | WAN | HTTP/HTTPS | ALLOW | Internet browsing |
| 2 | LAN | WAN | DNS | ALLOW | Name resolution |
| 3 | WAN | DMZ | HTTP/HTTPS | ALLOW | Public web access |
| 4 | DMZ | LAN | ANY | DENY | Prevent lateral movement |
| 5 | Admin VLAN | All | SSH/HTTPS | ALLOW | Device management |
| 6 | Any | Any | ANY | DENY | Default deny rule |

---

## 6. NAT Design

Network Address Translation (NAT) is used to allow private IP addresses to access the internet.

- **Source NAT (SNAT):** Internal devices access the internet using firewall public IP
- **Destination NAT (DNAT):** Port forwarding for DMZ services (e.g., web server)
- NAT rules are tightly controlled and logged

---

## 7. Security Features Enabled

The firewall configuration includes the following security controls:

- Stateful packet inspection
- Intrusion detection/prevention system (IDS/IPS)
- Geo-IP blocking (optional)
- VPN access (IPSec/OpenVPN/WireGuard)
- Traffic logging and monitoring
- Anti-spoofing protection
- DoS/DDoS basic mitigation

---

## 8. Logging and Monitoring

All critical traffic is logged for analysis and troubleshooting:

- Allowed and denied traffic logs
- VPN connection logs
- Administrative access logs
- System alerts and firewall events

Logs can be integrated with monitoring tools such as:
- SolarWinds
- ELK Stack (Elasticsearch, Logstash, Kibana)
- Graylog

---

## 9. Scalability Considerations

The firewall design supports future growth:

- Addition of new VLANs and security zones
- Integration with cloud environments (AWS, Azure)
- Support for SD-WAN expansion
- High availability (HA) firewall clustering

---

## 10. Conclusion

This firewall design provides a structured and secure foundation for the lab environment. It enforces segmentation, minimizes attack surfaces, and ensures controlled communication between network zones while maintaining flexibility for future expansion.