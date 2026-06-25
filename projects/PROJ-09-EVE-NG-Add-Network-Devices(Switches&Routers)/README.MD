# EVE-NG Network Device Deployment Documentation

## Project Title

**Deployment and Integration of Cisco Routers and Switches in EVE-NG Network Simulation Platform**

---

# 1. Project Overview

## Project Summary

This project documents the deployment of Cisco network devices within the EVE-NG network emulation platform.

The objective was to build a realistic enterprise network simulation environment capable of supporting:

* Network infrastructure testing
* CCNA/CCNP practical training
* VLAN and routing implementation
* Firewall and security testing
* Network monitoring integration
* Troubleshooting scenarios

The EVE-NG environment provides a virtualized networking platform where enterprise devices such as routers and switches can be deployed without requiring physical hardware.

---

## Business and Technical Value

This implementation provides the following benefits:

* Reduces the cost of physical networking equipment
* Provides a safe environment for configuration testing
* Enables enterprise network design validation
* Supports professional networking skill development
* Allows integration with monitoring platforms such as SolarWinds
* Supports future automation and cloud networking projects

---

# 2. Environment Overview

## Platform

| Component          | Details                    |
| ------------------ | -------------------------- |
| Network Emulator   | EVE-NG                     |
| Hypervisor         | VMware ESXi                |
| Firewall           | pfSense                    |
| Network Simulation | Cisco IOS / IOL / Dynamips |
| Purpose            | Enterprise Network Lab     |

---

# 3. Network Device Images Deployed

## Cisco Layer 2 Switch

### Image

```
i86bi-linux-l2-adventerprise-15.1b.bin
```

## Cisco Catalyst 3750 Layer 3 Switch

### Image

```
c3750-ipservicesk9-mz.122-55.SE10.bin
```
---

# 4. EVE-NG Image Directory Structure

## IOL Switch Images

Location:

```
/opt/unetlab/addons/iol/bin/
```

Example:

```
/opt/unetlab/addons/iol/bin/

└── i86bi-linux-l2-adventerprise-15.1b.bin
```

---

## Dynamips Router/Switch Images

Location:

```
/opt/unetlab/addons/dynamips/
```

Example:

```
/opt/unetlab/addons/dynamips/

    └── c3750-ipservicesk9-mz.122-55.SE10.bin
```

---

# 5. Adding Cisco IOL Switch Image

## Step 1 - Accessed EVE-NG Server

SSH into EVE-NG:

```
ssh root@<EVE-NG-IP>
```

Example:

```
ssh root@10.1.50.10
```

---

## Step 2 - Navigated to IOL Directory

```
cd /opt/unetlab/addons/iol/bin/
```

---

## Step 3 - Uploaded Image

Upload:

```
i86bi-linux-l2-adventerprise-15.1b.bin
```

using:

* WinSCP

Destination:

```
/opt/unetlab/addons/iol/bin/
```

---

## Step 4 - Set Permissions

Run:

```
chmod 755 i86bi-linux-l2-adventerprise-15.1b.bin
```

---

# 6. Adding Cisco Dynamips Switch Image

## Step 1 - Image Directory

Example:

```
/opt/unetlab/addons/dynamips/
```

---

## Step 2 - Upload IOS Image

Upload:

```
c3750-ipservicesk9-mz.122-55.SE10.bin
```

into:

```
/opt/unetlab/addons/dynamips/
```

---

# 7. Fix EVE-NG Permissions

After adding any network image:

Run:

```
/opt/unetlab/wrappers/unl_wrapper -a fixpermissions
```

Purpose:

* Correct file ownership
* Allow EVE-NG services to access images
* Register new devices

---

# 8. Sample Enterprise Topology

```
                 pfSense Firewall

                       |

                 LAB VLAN 50

                       |

                    EVE-NG

                       |

                 Cisco Router

                       |

                    Trunk Link

                       |

              Cisco 3750 Core Switch

                       |

             -------------------------

             |                       |

        IOL Access SW          IOL Access SW

             |                       |

           VPC01                 VPC02


        VLAN 52                VLAN 53

     USERS Network        SERVER Network

```

---

# 9. Validation Testing

## Device Boot Verification

Confirm:

* Device starts successfully
* Console access available
* IOS loads correctly

---

## Connectivity Testing

Example:

From switch:

```
pinged 10.1.50.1
```

Expected:

```
Success rate is 100 percent
```
---

# 10. Future Improvements

Planned enhancements:

* Add FortiGate firewall appliance
* Integrate SolarWinds monitoring
* Configure SNMP
* Implement OSPF routing
* Add network automation testing
* Add security monitoring tools

---

# Conclusion

The successful deployment of Cisco routers and switches inside EVE-NG establishes a scalable enterprise network simulation environment.

This platform enables realistic infrastructure testing, troubleshooting practice, and continuous improvement of network engineering skills without requiring physical Cisco hardware.
