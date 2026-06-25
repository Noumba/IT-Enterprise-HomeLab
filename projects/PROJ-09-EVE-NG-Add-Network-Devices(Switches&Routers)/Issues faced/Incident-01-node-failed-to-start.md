# EVE-NG — IOL Switch Node Failed to Start
## Incident & Resolution

---

## Document Control

| Field | Value |
|---|---|
| Environment | EVE-NG Host — 10.1.50.10 — VLAN50 Networking |
| Image Affected | i86bi-linux-l2-adventerprisek9-15.1a.bin / i86bi-linux-l2-adventerprise-15.1b.bin |
| Status | Resolved |

---

## Issue Summary

After uploading Cisco IOL switch images to EVE-NG, the switch nodes failed to start. The node status remained red and the console showed no output.

---

## Errors Encountered

```
# Wrapper log
ERR    Error while connecting local AF_UNIX: No such file or directory
ERR    Cannot open AF_UNIX sockets
ERR    Failed to create AF_UNIX socket file

# Running binary directly
libcrypto.so.4: cannot open shared object file: No such file or directory

# After library fix
IOU License Error: invalid license
License for key 7f03a4 required on host "eve-ng01"

# After wrong license key
Permission denied
```

---

## Root Causes

| # | Cause |
|---|---|
| 1 | Missing 32-bit library (`libcrypto.so.4`) — IOL images are 32-bit binaries |
| 2 | Incorrect iourc license key — wrong generation algorithm used |
| 3 | Incorrect file permissions on the IOL binary |

---

## What Did NOT Fix the Issue

- Manually writing `echo 1 > /proc/sys/net/ipv4/ip_forward` — unrelated to this issue
- Generating license using basic hostname hash algorithms — produced wrong key format
- Copying a pre-made iourc file with mismatched hostname
- Writing license to `/root/iourc` only — EVE-NG runs IOL as `unl1` user, not root

---

## Fix — Step by Step

### 1 — Fix Missing 32-bit Library

```bash
# Create symlink for missing library
ln -s /opt/unetlab/addons/iol/lib/libcrypto.so.4 /usr/lib/i386-linux-gnu/libcrypto.so.4
ldconfig
```

### 2 — Generate Correct IOL License Using CiscoIOUKeygen Algorithm

```bash
python3 -c "
import os, socket, hashlib, struct

hostid = os.popen('hostid').read().strip()
hostname = socket.gethostname()
ioukey = int(hostid, 16)
for x in hostname:
    ioukey = ioukey + ord(x)

iouPad1 = b'\x4B\x58\x21\x81\x56\x7B\x0D\xF3\x21\x43\x9B\x7E\xAC\x1D\xE6\x8A'
iouPad2 = b'\x80' + 39*b'\x00'
md5input = iouPad1 + iouPad2 + struct.pack('!i', ioukey) + iouPad1
iouLicense = hashlib.md5(md5input).hexdigest()[:16]

content = '[license]\n' + hostname + ' = ' + iouLicense + ';\n'
paths = [
    '/root/iourc',
    '/opt/unetlab/tmp/1/iourc',
    '/opt/unetlab/addons/iol/bin/iourc',
    '/etc/iourc'
]
for path in paths:
    with open(path, 'w') as f:
        f.write(content)
    print('Written: ' + path)
print('License key: ' + iouLicense)
"
```

### 3 — Fix File Permissions

```bash
# Fix binary permissions
chmod 755 /opt/unetlab/addons/iol/bin/*.bin

# Fix iourc ownership for unl1 user
chown unl1:unl1 /opt/unetlab/tmp/1/iourc
chmod 644 /root/iourc /opt/unetlab/tmp/1/iourc \
          /opt/unetlab/addons/iol/bin/iourc /etc/iourc

# Fix all EVE-NG permissions
/opt/unetlab/wrappers/unl_wrapper -a fixpermissions
```

### 4 — Verify Binary Runs Correctly

```bash
sudo -u unl1 bash -c 'export HOME=/opt/unetlab/tmp/1 && \
  cd /opt/unetlab/tmp/1 && \
  /opt/unetlab/addons/iol/bin/i86bi-linux-l2-adventerprisek9-15.1a.bin 1'
```

Expected output — license accepted:
```
netio error: unable to open NETMAP: No such file or directory
```
> This NETMAP error is normal when running manually — EVE-NG creates it automatically at startup.

---

## Key Takeaways

- **IOL images require the CiscoIOUKeygen MD5-based algorithm** — simpler hostname hash methods produce invalid keys
- **EVE-NG runs IOL as the `unl1` user** — iourc must be readable by that user, not just root
- **libcrypto.so.4 ships with EVE-NG** at `/opt/unetlab/addons/iol/lib/` — it just needs symlinking into the system library path
- **Permission denied on the binary** is always fixable with `chmod 755` followed by `unl_wrapper -a fixpermissions`

