# Kali Linux Setup

## Role

Kali Linux is used for controlled security testing and network
analysis within the isolated homelab environment.

## System Information

| Property | Value |
|---|---|
| OS | Kali Linux |
| Hostname | kali |
| Internal IP | 10.0.2.15 |
| Role | Security Testing |

## Network Configuration

The VM uses:

- NAT for updates and package installation
- Internal Network for isolated lab testing
  
## Basic Verification

Verify connectivity to the server:

```bash
ping 10.0.0.10
