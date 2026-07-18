# Ubuntu Desktop Setup

## Role

The Ubuntu Desktop VM represents a standard client workstation within the homelab.

It is primarily used to:

* Test client-to-server connectivity
* Access services hosted by Ubuntu Server
* Practice network troubleshooting
* Generate normal client activity
* Serve as a workstation for future IT and networking exercises

## System Configuration

| Property         | Value              |
| ---------------- | ------------------ |
| Operating System | Ubuntu Desktop     |
| Hostname         | `UbuntuDesktop`   |
| Role             | Client Workstation |
| CPU              | 2 cores            |
| Memory           | 4 GB               |
| Storage          | 30 GB              |

> Update the hardware values to match your actual VM configuration.

## VirtualBox Network Configuration

The VM uses two network adapters:

| Adapter   | Network Type     | Purpose                              |
| --------- | ---------------- | ------------------------------------ |
| Adapter 1 | NAT              | Internet access and system updates   |
| Adapter 2 | Internal Network | Communication with other homelab VMs |

After installing Ubuntu Desktop, the system was updated:

```bash
sudo apt update
sudo apt upgrade
```

## Current State

The Ubuntu Desktop VM currently functions as the primary client workstation in the homelab.

It is used to test services hosted on Ubuntu Server and to practice network troubleshooting from the perspective of a normal client device.
