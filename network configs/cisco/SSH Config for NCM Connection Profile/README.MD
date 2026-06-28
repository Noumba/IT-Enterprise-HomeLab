# Objective

To configure SSH on Cisco network devices to enable secure communication and configuration management from SolarWinds Network Configuration Manager (NCM).

# Activities Performed

The following SSH configurations were applied on the Cisco device.

## SSH Configuration

```cisco
ip domain-name homelab.local

crypto key generate rsa modulus 2048

ip ssh version 2

username admin privilege 15 secret xxxxxxxx

line vty 0 4
 login local
 transport input ssh

end

write memory
```

## Configuration Explanation

### Configure Device Domain Name

```cisco
ip domain-name homelab.local
```

Configured the device domain name required for generating RSA keys used by SSH.

---

### Generate RSA Encryption Keys

```cisco
crypto key generate rsa modulus 2048
```

Generated RSA cryptographic keys to enable SSH encryption and secure communication between SolarWinds NCM and the Cisco device.

---

### Enable SSH Version 2

```cisco
ip ssh version 2
```

Enabled SSH version 2 to provide secure remote access using modern encryption methods.

---

### Create Local Administrator Account

```cisco
username admin privilege 15 secret xxxxxxxx
```

Created a local administrator account used by SolarWinds NCM to authenticate and perform configuration management tasks.

Privilege level 15 provides full administrative access required for:

- Configuration backups
- Configuration changes
- Device management

---

### Configure VTY Lines for SSH Access

```cisco
line vty 0 4
 login local
 transport input ssh
```

Configured the remote access lines to:

- Authenticate using the local user database
- Allow only SSH connections
- Prevent insecure remote access methods such as Telnet

---

### Save Configuration

```cisco
end

write memory
```

Saved the configuration to the startup configuration so the SSH settings remain after a device reboot.

---

# Verification

The SSH configuration was verified using:

```cisco
show ip ssh
```

Expected output:

```text
SSH Enabled - version 2.0
```

NCM Connection Profile

![NCM Profile](../../vmware-esxi/Images/NCM%20SSH%20Connection%20Test.JPG)

---

# Outcome

SSH was successfully configured on the Cisco network device.

The device is now ready to be added to SolarWinds Network Configuration Manager (NCM) for:

- Configuration backups
- Configuration management
- Configuration changes
- Network compliance monitoring
- Secure remote administration