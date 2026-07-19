# Changelog

## v0.1
- Ubuntu Server
- Kali VM
- Tested several commands
  ```
  ping - Tested layer 3 connectivity
  ip addr - IP address
  ip route - Check routing table
  traceroute - See the path on network
  arp -a - Viewed ARP table
  ss -tuln - Checked listening services
  ```

## v0.2
- Ubuntu Desktop

## v0.3
- Assigned static IP addressing for easier documentation and experimenting

| Machine | NAT Network | Internal Homelab Network |
|---|---|---|
| Ubuntu Server | `10.0.2.15` | `10.0.0.10` |
| Ubuntu Desktop | `10.0.2.15` | `10.0.0.20` |
| Kali Linux | `10.0.2.15` | `10.0.0.30` |
