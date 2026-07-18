# Virtual Machine Creation

## Purpose

This document describes the common process used to create the virtual
machines in the homelab.

## Virtualization Software

- VirtualBox
- ISO images downloaded from official distribution sources

## VM Resource Allocation

| VM | RAM | CPU | Disk |
|---|---:|---:|---:|
| Ubuntu Server | 2 GB | 2 cores | 25 GB |
| Ubuntu Desktop | 4 GB | 2 cores | 30 GB |
| Kali Linux | 2 GB | 2 cores | 30 GB |

## Network Adapters

Each VM uses:

### Adapter 1: NAT

Purpose:
- Internet access
- System updates
- Package installation

### Adapter 2: Internal Network

Network Name:

Homelab-NAT

Purpose:
- Isolated VM-to-VM communication
- Network experiments
- Security testing

## VM Creation Process

1. Create a new VirtualBox VM.
2. Select the appropriate operating system.
3. Allocate CPU, memory, and storage.
4. Configure the NAT adapter.
5. Configure the Internal Network adapter.
6. Install the operating system.
7. Complete the OS-specific configuration.

## Validation

After installation, verify:

```bash
ip addr
ip route
