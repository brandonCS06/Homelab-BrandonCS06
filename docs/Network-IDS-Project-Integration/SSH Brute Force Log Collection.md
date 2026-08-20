# Network Security Lab - SSH Brute-Force Log Collection

## Overview

This lab integrates the VirtualBox homelab with the local Network IDS
project by collecting real SSH authentication failures from the Ubuntu
Server and transferring the resulting log file to the physical host
machine.

**Link to Network IDS project** - https://github.com/brandonCS06/Network-IDS-Simulator

The goal of this phase was **log collection and transfer**. IDS parsing
and detection are handled separately.

### Lab Topology

  Machine          IP Address    Purpose
  ---------------- ------------- -------------------------------
  Ubuntu Server    `10.0.0.10`   SSH target and log source
  Ubuntu Desktop   `10.0.0.20`   Homelab workstation
  Kali Linux       `10.0.0.30`   Attack/testing machine
  Windows Host     `10.0.0.1`    Physical host and IDS machine

The Ubuntu Server uses:

-   Adapter 1: NAT Network
-   Adapter 2: VirtualBox Host-Only Network

The Host-Only network uses the `10.0.0.0/24` subnet.

------------------------------------------------------------------------

## Objective

The objective was to:

1.  Generate deliberate failed SSH authentication attempts from Kali.
2.  Confirm that Ubuntu Server records those failures in
    `/var/log/auth.log`.
3.  Extract the relevant SSH failures into a smaller log file.
4.  Verify Host-Only connectivity between the Windows host and Ubuntu
    Server.
5.  Use `scp` to transfer the extracted log from Ubuntu Server to the
    Windows host.
6.  Store the log in the homelab's IDS log directory.

The resulting workflow is:

``` text
Kali
  │
  │ Failed SSH attempts
  ▼
Ubuntu Server
  │
  │ /var/log/auth.log
  ▼
ssh_failed.log
  │
  │ SCP
  ▼
Windows Host (My computer)
  │
  ▼
IDS homelab_logs directory
```

------------------------------------------------------------------------

# 1. Generate Failed SSH Attempts

From Kali Linux, SSH to the Ubuntu Server:

``` bash
ssh vboxuser@10.0.0.10
```

When prompted for the password, deliberately enter an incorrect password
several times.

The Network IDS project's `BruteForceRule` is configured to detect:

-   5 or more failed login events
-   From the same source IP
-   Within a 60-second window

Therefore, generating at least 5 failures within a short period provides
enough activity for the later IDS detection phase.

------------------------------------------------------------------------

# 2. Verify SSH Failures on Ubuntu Server

On Ubuntu Server, the SSH authentication logs are stored in:

``` text
/var/log/auth.log
```

A normal `grep` command reported:

``` text
grep: /var/log/auth.log: binary file matches
```

Ubuntu's log file was therefore searched using `grep -a`, which forces
the file to be treated as text:

``` bash
sudo grep -a "Failed password" /var/log/auth.log | tail -20
```

The command successfully displayed the SSH failures.

Example entries observed during the lab:

``` text
2026-08-20T16:20:58.328020+00:00 UbuntuServer sshd-session[1680]: Failed password for vboxuser from 10.0.0.30 port 52604 ssh2
2026-08-20T16:21:19.293843+00:00 UbuntuServer sshd-session[1680]: Failed password for vboxuser from 10.0.0.30 port 52604 ssh2
2026-08-20T16:21:21.634293+00:00 UbuntuServer sshd-session[1680]: Failed password for vboxuser from 10.0.0.30 port 52604 ssh2
```

This confirmed that:

-   Kali's source IP was `10.0.0.30`
-   The SSH target was Ubuntu Server
-   The attempted username was `vboxuser`
-   Ubuntu was successfully recording failed SSH authentication attempts

------------------------------------------------------------------------

# 3. Extract the SSH Failures

Instead of transferring the entire `/var/log/auth.log` file, the
relevant failed SSH entries were extracted into a separate file.

On Ubuntu Server:

``` bash
sudo grep -a "Failed password" /var/log/auth.log > ~/ssh_failed.log
```

The resulting file was:

``` text
/home/vboxuser/ssh_failed.log
```

Verify its contents:

``` bash
cat ~/ssh_failed.log
```

Check the number of extracted entries:

``` bash
wc -l ~/ssh_failed.log
```

This creates a small, controlled log file containing only the SSH
authentication failures relevant to this lab.

------------------------------------------------------------------------

# 4. Configure Host-Only Networking

Initially, the VirtualBox Host-Only adapter on the Windows host used:

``` text
IPv4 Address: 192.168.56.1
Network Mask: 255.255.255.0
```

However, the homelab VMs were using:

``` text
10.0.0.0/24
```

with Ubuntu Server configured as:

``` text
10.0.0.10/24
```

The Host-Only network was therefore changed to:

``` text
IPv4 Address: 10.0.0.1
Network Mask: 255.255.255.0
```

The resulting network was:

``` text
Windows Host
10.0.0.1
    │
    │ VirtualBox Host-Only Network
    │
    ├── Ubuntu Server
    │   10.0.0.10
    │
    ├── Ubuntu Desktop
    │   10.0.0.20
    │
    └── Kali Linux
        10.0.0.30
```

Ubuntu Server's Host-Only interface was confirmed with:

``` bash
ip addr
```

The relevant interface was:

``` text
enp0s8
10.0.0.10/24
```

------------------------------------------------------------------------

# 5. Verify Host-to-VM Connectivity

From the Windows host, connectivity to Ubuntu Server was tested with:

``` cmd
ping 10.0.0.10
```

The Windows host successfully reached Ubuntu Server.

A reverse ping from Ubuntu Server to the Windows Host-Only address
resulted in 100% packet loss. This did not prevent the lab from
continuing because the required direction for log transfer was:

``` text
Windows Host → Ubuntu Server
```

and SSH/SCP connectivity from Windows to Ubuntu was successfully
established.

------------------------------------------------------------------------

# 6. Test SSH From the Windows Host

From Windows CMD or PowerShell:

``` cmd
ssh vboxuser@10.0.0.10
```

The connection successfully reached the Ubuntu Server and authenticated
using the `vboxuser` account.

Exit the SSH session with:

``` bash
exit
```

This confirmed that the Windows host could communicate with Ubuntu
Server over the Host-Only network.

------------------------------------------------------------------------

# 7. Transfer the Log With SCP

The first SCP attempt omitted the destination:

``` cmd
scp vboxuser@10.0.0.10:/home/vboxuser/ssh_failed.log
```

This produced the `scp` usage message because SCP requires both a source
and a destination.

The correct command is:

``` cmd
scp vboxuser@10.0.0.10:/home/vboxuser/ssh_failed.log .
```

The final `.` means:

``` text
Copy the file into the current Windows directory.
```

The transfer completed successfully.

------------------------------------------------------------------------

# 8. Copy Directly to the IDS Log Directory

The local IDS homelab log directory is:

``` text
C:\Users\brand\CS Projects\IDS simulator\IDS-simulator\homelab_logs
```

Because the path contains spaces, it should be enclosed in quotes.

The log can be copied directly to that directory with:

``` cmd
scp vboxuser@10.0.0.10:/home/vboxuser/ssh_failed.log "C:\Users\brand\CS Projects\IDS simulator\IDS-simulator\homelab_logs\ssh_failed.log"
```

Alternatively, change into the directory first:

``` cmd
cd /d "C:\Users\brand\CS Projects\IDS simulator\IDS-simulator\homelab_logs"
```

Then use:

``` cmd
scp vboxuser@10.0.0.10:/home/vboxuser/ssh_failed.log .
```

The second approach is useful when repeatedly collecting logs because
the destination does not have to be typed every time.

------------------------------------------------------------------------

# 9. Final Result

The completed log collection pipeline is:

``` text
┌──────────────┐
│ Kali Linux   │
│ 10.0.0.30    │
└──────┬───────┘
       │
       │ Failed SSH attempts
       ▼
┌──────────────┐
│ Ubuntu       │
│ Server       │
│ 10.0.0.10    │
└──────┬───────┘
       │
       │ /var/log/auth.log
       ▼
┌────────────────────┐
│ ssh_failed.log     │
│ extracted SSH logs │
└──────────┬─────────┘
           │
           │ SCP
           ▼
┌──────────────────────────────┐
│ Windows Host                 │
│                              │
│ IDS-simulator/               │
│ └── homelab_logs/            │
│     └── ssh_failed.log       │
└──────────────────────────────┘
```

At this point, the physical host has successfully received real SSH
authentication logs generated by activity inside the VirtualBox homelab.

## IDS Integration Status

  Component                       Status
  ------------------------------- -------------------------
  Kali → Ubuntu SSH activity      Complete
  Ubuntu SSH logging              Complete
  SSH log extraction              Complete
  Host-Only connectivity          Complete
  Windows → Ubuntu SSH            Complete
  Ubuntu → Windows SCP transfer   Complete
  Log stored on host              Complete
  IDS log parsing                 Deferred to IDS project
  Brute-force alert generation    Next phase


## Confirmation of Success

<img width="1408" height="777" alt="image" src="https://github.com/user-attachments/assets/71292ee5-bab6-4a01-a8a6-f4e33d112eb9" />
<img width="1417" height="526" alt="image" src="https://github.com/user-attachments/assets/a6e14a4c-2e85-410a-9457-f0580ce5da3a" />

## Useful Commands

### Ubuntu Server

View recent failed SSH attempts:

``` bash
sudo grep -a "Failed password" /var/log/auth.log | tail -20
```

Extract failed SSH attempts:

``` bash
sudo grep -a "Failed password" /var/log/auth.log > ~/ssh_failed.log
```

View extracted log:

``` bash
cat ~/ssh_failed.log
```

Count extracted events:

``` bash
wc -l ~/ssh_failed.log
```

### Windows

SSH into Ubuntu:

``` cmd
ssh vboxuser@10.0.0.10
```

Copy a file to the current directory:

``` cmd
scp vboxuser@10.0.0.10:/home/vboxuser/ssh_failed.log .
```

Change to the IDS log directory:

``` cmd
cd /d "C:\Users\brand\CS Projects\IDS simulator\IDS-simulator\homelab_logs"
```

Copy directly to the IDS log directory:

``` cmd
scp vboxuser@10.0.0.10:/home/vboxuser/ssh_failed.log "C:\Users\brand\CS Projects\IDS simulator\IDS-simulator\homelab_logs\ssh_failed.log"
```
