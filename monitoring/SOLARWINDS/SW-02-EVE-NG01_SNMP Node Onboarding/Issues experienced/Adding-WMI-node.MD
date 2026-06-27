```md
# Resolving "0x800706BA - The RPC Server is Unavailable" Error

## Issue

The following error was encountered while attempting WMI polling from SolarWinds:

> Number: 0x800706ba  
> Facility: Win32  
> Description: The RPC server is unavailable.

## Resolution

The issue was resolved by performing the following actions on the target Windows server:

- Enabled the **Windows Management Instrumentation (WMI)** firewall rules for the **Domain** and **Private** firewall profiles.
- Started the **Remote Registry** service.
- Configured the **Remote Registry** service startup type to **Automatic**.

## Outcome

After implementing the above changes, WMI communication was successfully established, and the Windows server was successfully added and monitored in SolarWinds.
```
