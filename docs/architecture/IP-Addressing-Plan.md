## 1. IP Addressing Scheme

### Overview

The IP Addressing Plan defines how IP networks and addresses are allocated throughout the home lab environment. A structured addressing scheme is essential for ensuring reliable communication between systems, simplifying troubleshooting, supporting future growth, and maintaining consistency across the infrastructure.

This document serves as the authoritative reference for all network segments, VLANs, gateways, static assignments, and dynamic address allocations within the lab. It provides both technical and operational guidance to ensure that devices can be deployed, managed, and monitored in a predictable manner.

From a technical perspective, the addressing plan establishes clear Layer 3 boundaries, supports network segmentation and security policies, and enables efficient routing between network zones. From an operational perspective, it provides administrators with a centralized source of truth for network assignments, reducing configuration errors and simplifying change management activities.

The addressing strategy used in this environment follows common enterprise networking practices by separating management, infrastructure, cloud, DMZ, and lab workloads into dedicated subnets. This approach improves security, scalability, and operational visibility while creating a realistic environment for learning, testing, and professional development.

### Objectives

The primary objectives of the IP Addressing Plan are:

- Establish a structured and scalable IP addressing scheme.
- Support network segmentation through dedicated VLANs and subnets.
- Simplify administration, troubleshooting, and documentation efforts.
- Enable effective implementation of security policies and firewall controls.
- Provide predictable addressing for infrastructure, management, monitoring, and application services.
- Reserve sufficient address space to support future expansion.
- Maintain alignment with enterprise networking standards and best practices.

### Full Address Table

| Device / Interface | IP Address | Subnet Mask | Gateway | VLAN | Notes |
|---|---|---|---|---|---|
| ISP Router (LAN) | 192.168.1.1 | /24 | — | WAN | ISP-provided gateway |
| pfSense WAN (vmnic0) | 192.168.1.50 | /24 | 192.168.1.1 | WAN | Static or DHCP from ISP router |
| pfSense VLAN10 | 10.1.10.1 | /24 | — | 10 | MGMT gateway |
| pfSense VLAN20 | 10.1.20.1 | /24 | — | 20 | INFRA gateway |
| pfSense VLAN30 | 10.1.30.1 | /24 | — | 30 | DMZ gateway |
| pfSense VLAN50 | 10.1.50.1 | /24 | — | 50 | LAB gateway |
| ESXi MGMT VM | 10.1.10.100 | /24 | 10.1.10.1 | 10 | Admin workstation / jump host |
| ESXi INFRA VM (example) | 10.1.20.100 | /24 | 10.1.20.1 | 20 | DNS/NTP/monitoring server |
| ESXi DMZ VM (example) | 10.1.30.100 | /24 | 10.1.30.1 | 30 | Web server / exposed service |
| ESXi CLOUD VM (example) | 10.1.40.100 | /24 | 10.1.40.1 | 30 | Linux / Cloud Integration |
| ESXi LAB VM (example) | 10.1.50.100 | /24 | 10.1.50.1 | 50 | Test/lab node |

### DHCP Scopes (pfSense)

| VLAN | DHCP Range | Excluded / Reserved |
|---|---|---|
| VLAN10 MGMT | 10.1.10.50 – 10.1.10.200 | .1 (GW), .100 reserved for MGMT VM |
| VLAN20 INFRA | 10.1.20.50 – 10.1.20.200 | .1 (GW), .100 reserved for INFRA VM |
| VLAN30 DMZ | 10.1.30.50 – 10.1.30.200 | .1 (GW), .100 reserved for DMZ VM |
| VLAN40 CLOUD | 10.1.40.50 – 10.1.40.200 | .1 (GW), .100 reserved for CLOUD/DEVOPs VM |
| VLAN50 LAB | 10.1.50.50 – 10.1.50.200 | .1 (GW), .100 reserved for LAB VM |
