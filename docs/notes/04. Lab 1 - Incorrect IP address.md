# Network Troubleshooting Lab 01 - Incorrect IP Address

## Objective

Practice identifying and resolving a network connectivity issue caused by an incorrect static IP address. The goal of this lab was to follow the structured troubleshooting methodology that would be used in real scenario.

---

# Lab Environment

| Machine | Role | Internal IP |
|----------|------|-------------|
| Ubuntu Server | Web/SSH Server | 10.0.0.10/24 |
| Ubuntu Desktop | Client | 10.0.0.20/24 |
| Kali Linux | Testing Machine | 10.0.0.30/24 |

**VirtualBox Network Configuration**

- Adapter 1 (NAT): `10.0.2.0/24`
  - Provides Internet access.
- Adapter 2 (Internal Network): `10.0.0.0/24`
  - Used for communication between lab machines.

---

# Scenario

A user reports that the Ubuntu Desktop can no longer communicate with the Ubuntu Server.

### Problem Statement

> Ubuntu Desktop cannot ping or connect to Ubuntu Server (10.0.0.10).

---

# Baseline Verification

Before introducing the issue, all systems were communicating successfully.

### Ubuntu Desktop

```bash
ping -c 4 10.0.0.10
```

Result:

```
4 packets transmitted
4 packets received
0% packet loss
```

This confirmed that the network was functioning correctly before making any changes.

---

# Introducing the Fault

The static IP address of the Ubuntu Desktop's internal network adapter (`enp0s8`) was intentionally changed.

### Original Configuration

| Interface | Address |
|-----------|----------|
| enp0s8 | 10.0.0.20/24 |

### Incorrect Configuration

| Interface | Address |
|-----------|----------|
| enp0s8 | 10.0.1.20/24 |

This placed the Ubuntu Desktop on a different subnet than the Ubuntu Server.

---

# Symptoms

Attempting to ping the Ubuntu Server produced:

```bash
ping -c 4 10.0.0.10
```

Result:

```
100% packet loss
```

<img width="538" height="91" alt="image" src="https://github.com/user-attachments/assets/77ea332c-224a-4077-ad8e-af6ef2ff71f2" />


The Desktop was no longer able to communicate with the server.

---

# Troubleshooting Process

## Step 1 - Verify the Symptoms

Confirmed that communication from Ubuntu Desktop to Ubuntu Server failed.

Result:

- Ping failed
- 100% packet loss

---

## Step 2 - Check IP Configuration

Command:

```bash
ip -br addr
```

Output:


<img width="641" height="99" alt="image" src="https://github.com/user-attachments/assets/dedfb62f-a39c-4d94-a9c3-adfad5c479ac" />


Observation:

The Ubuntu Desktop was configured with the IP address `10.0.1.20/24`.

The Ubuntu Server was still using:

```
10.0.0.10/24
```

These systems were now on different subnets.

---

## Step 3 - Check the Routing Table

Command:

```bash
ip route
```

Output:


<img width="603" height="77" alt="image" src="https://github.com/user-attachments/assets/2d55d7be-309a-446a-89b0-66b58d824ada" />


Observation:

Ubuntu believed its local network was:

```
10.0.1.0/24
```

Since the server exists on:

```
10.0.0.0/24
```

there was no directly connected route to reach the server.

---

## Step 4 - Verify the Server

To determine whether the issue was isolated to the Desktop or affected the entire network, connectivity was tested from Kali Linux.

Command:

```bash
ping -c 4 10.0.0.10
```

Result:


<img width="530" height="193" alt="image" src="https://github.com/user-attachments/assets/84412e30-c204-4cf1-8a36-70a71f38fbe4" />


Observation:

The server was functioning normally.

The internal network was also functioning normally.

This isolated the issue to the Ubuntu Desktop.

---

# Root Cause Analysis

The Ubuntu Desktop was manually configured with an incorrect static IP address.

Instead of:

```
10.0.0.20/24
```

it was configured as:

```
10.0.1.20/24
```

Because the Desktop and Server were on different subnets, they could not communicate directly.

---

# Resolution

The Desktop's IP address was restored to the correct value.

| Interface | Correct Address |
|-----------|-----------------|
| enp0s8 | 10.0.0.20/24 |

After reconnecting the network interface, connectivity was tested again.

Command:

```bash
ping -c 4 10.0.0.10
```

Result:


<img width="518" height="179" alt="image" src="https://github.com/user-attachments/assets/27e94883-baea-474e-8792-11c0100da2bb" />


The issue was successfully resolved.

---

# Commands Used

## Check interface configuration

```bash
ip -br addr
```

## View routing table

```bash
ip route
```

## Display assigned IP addresses

```bash
hostname -I
```

## Test connectivity

```bash
ping -c 4 <IP Address>
```

---

# Key Troubleshooting Lessons

- Always establish a known-good baseline before making configuration changes.
- Verify the reported issue before attempting a fix.
- Check IP addressing before investigating services such as SSH or Apache.
- Use `ip route` to understand how the operating system is routing traffic.
- Test connectivity from another machine to determine whether the issue is isolated or network-wide.
- Gather evidence before making changes to avoid unnecessary troubleshooting.

---

# Skills Practiced

- Static IP configuration
- Network troubleshooting methodology
- IPv4 subnetting concepts
- Routing table analysis
- Linux networking commands
- ICMP connectivity testing
- Root cause analysis
- Structured documentation

---

# Outcome

Successfully diagnosed and resolved a network connectivity issue caused by an incorrect static IP configuration by following a systematic troubleshooting process instead of immediately correcting the problem.
