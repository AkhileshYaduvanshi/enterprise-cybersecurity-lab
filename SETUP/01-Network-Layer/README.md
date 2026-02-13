
# Network Layer - pfSense Firewall & Gateway

## Overview

The Network Layer is the foundation of your lab infrastructure. It provides:
- **Firewall & Security:** Protect your lab from external threats
- **Gateway & Routing:** Route traffic between management and lab networks
- **VPN Support:** Secure remote access to your lab
- **DHCP (Optional):** Can provide IP addresses to lab VMs
- **DNS Forwarding:** Can act as a DNS proxy

## What You Will Learn

This section covers:
1. Creating pfSense VM in Proxmox
2. Configuring WAN and LAN interfaces
3. Initial pfSense setup and configuration
4. Firewall rules for lab security
5. VPN setup (optional)

### IP Scheme

Management Network: 192.168.1.0/24 (external)
Lab Network: 172.16.1.0/24 (internal)
pfSense WAN IP: 192.168.1.101 (DHCP/static)
pfSense LAN IP: 172.16.1.1/24 (gateway)
