# Identity Layer - Active Directory Domain Controller

## Overview

The Identity Layer provides centralized identity and authentication services using Active Directory. This layer is essential for:
- User and computer authentication
- DNS services for the lab domain
- Group Policy management
- Domain-joined VM configuration
- Central authentication point for all lab systems

## Architecture

```
┌─────────────────────────────────────────────────┐
│                 pfSense Firewall                 │
│            (Network Layer - 172.16.1.1)         │
└──────────────────────┬──────────────────────────┘
                       │
                       │ Internal Lab Network
                       │ 172.16.1.0/24
                       │
        ┌──────────────▼──────────────┐
        │     DC-01 (Domain Controller)│
        │   172.16.1.10 (lab.internal) │
        │                              │
        │  Windows Server 2022 Core    │
        │  - Active Directory          │
        │  - DNS Server                │
        │  - DHCP Server               │
        │  - Group Policy              │
        └──────────────┬───────────────┘
                       │
        ┌──────────────┴──────────────────────────┐
        │                                         │
    Lab VMs (domain-joined)         Management VMs
    - Client machines              - Monitoring
    - Servers                      - Services
    - Workstations
```

## Components Covered

1. **01-Proxmox-DC-VM-Creation-Setup.md** - VM creation in Proxmox
2. **02-Windows-Server-Installation.md** - Windows Server 2022 Core installation
3. **03-Active-Directory-Setup.md** - AD promotion and domain configuration

## Key Information

### VM Specifications

```
VM ID:           102
Name:            DC-01
OS:              Windows Server 2022 Core
CPU:             2 vCPU (1 socket, 2 cores)
RAM:             2 GB
Storage:         60 GB (local-lvm)
Network:         1 NIC on vmbr1 (172.16.1.0/24)
```

### Network Configuration

```
Hostname:        DC-01
FQDN:            DC-01.lab.internal
Domain:          lab.internal
NetBIOS Name:    LAB
IP Address:      172.16.1.10/24
Gateway:         172.16.1.1 (pfSense)
DNS:             127.0.0.1 (local, AD DNS)
```

### Active Directory

```
Forest:          lab.internal
Domain:          lab.internal
NetBIOS:         LAB
Forest Level:    Windows Server 2022
Domain Level:    Windows Server 2022
```

### Admin Credentials

```
Domain Admin:    LAB\Administrator (or admin@lab.internal)
Local Admin:     labadmin (domain admin user for remote access)
```

## Next Steps After Setup

1. **Create User and Computer OUs** in Active Directory
2. **Configure Group Policy** for lab machines
3. **Domain-join other VMs** to lab.internal
4. **Set up DHCP** for automatic IP assignment
5. **Configure DNS** for lab services
6. **Backup AD database** regularly

## Security Notes

- DC-01 is the critical identity service - protect and backup regularly
- Use strong passwords for all AD accounts
- Enable Windows Firewall rules for lab VMs
- Consider enabling AD recycle bin for accidental deletions
- Regular AD health checks recommended
