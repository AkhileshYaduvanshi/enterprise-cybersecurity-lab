# SSH Access Configuration

## Overview

This guide covers SSH setup for secure remote access to lab systems. SSH was configured on the Bastion Host (Debian) and DC-01 (Windows Server 2022 Core) to enable direct terminal access from external networks via VPN.

## What We Accomplished

✅ **Bastion Host:** SSH server verified and SSH key authentication configured  
✅ **DC-01:** OpenSSH Server installed and configured on Windows Server Core  
✅ **Passwordless Access:** SSH keys set up for bastion host  
✅ **Firewall Rules:** SSH access allowed on both systems  
✅ **Tested & Working:** Verified SSH connectivity to both systems  

***

## Part 1: Bastion Host SSH Configuration

### Prerequisites

- Bastion host (172.16.1.20) running Debian 12
- SSH server installed during Debian installation
- VPN connected to internal network

### Step 1: Verify SSH Service

**SSH server was already installed** during Debian installation (selected in software selection).

**Verify SSH is running:**

```bash
# Check SSH service status
sudo systemctl status ssh

# Should show: Active: active (running)
```

**Default SSH configuration:**
- Port: 22
- Password authentication: Enabled
- Root login: Permitted
- Key authentication: Enabled

***

### Step 2: Generate SSH Key Pair (Client Side)

**On your local machine (Mac/Linux/Windows):**

```bash
# Generate ED25519 key (recommended - most secure)
ssh-keygen -t ed25519 -C "labadmin@homelab"

# Or use RSA if ED25519 not supported
ssh-keygen -t rsa -b 4096 -C "labadmin@homelab"
```

**Prompts:**
```
Enter file in which to save the key (/Users/yourname/.ssh/id_ed25519): [Press Enter]
Enter passphrase (empty for no passphrase): [Press Enter or set passphrase]
Enter same passphrase again: [Press Enter]
```

**Note:** The `-C "labadmin@homelab"` is just a comment/label for identification - it's optional.

**Keys generated:**
- **Private key:** `~/.ssh/id_ed25519` (keep secret!)
- **Public key:** `~/.ssh/id_ed25519.pub` (safe to share)

***

### Step 3: Copy Public Key to Bastion

**Method 1: Using ssh-copy-id (Recommended)**

```bash
ssh-copy-id -i ~/.ssh/id_ed25519.pub labadmin@172.16.1.20
```

**Enter password** when prompted (last time!)

**Method 2: Manual Copy**

```bash
# Display public key
cat ~/.ssh/id_ed25519.pub

# Copy the output, then SSH to bastion and add it manually:
ssh labadmin@172.16.1.20

# On bastion:
mkdir -p ~/.ssh
chmod 700 ~/.ssh
nano ~/.ssh/authorized_keys
# Paste the public key, save and exit

chmod 600 ~/.ssh/authorized_keys
```

***

### Step 4: Test SSH Key Authentication

**Connect without password:**

```bash
ssh labadmin@172.16.1.20
```

**Expected result:** Login successful without password prompt! ✅

**What happened:**
1. SSH client presents your private key
2. Server verifies against public key in `~/.ssh/authorized_keys`
3. Authentication succeeds without password

***

### Step 5: Initial Connection (First Time)

**On first connection, you'll see:**

```
The authenticity of host '172.16.1.20 (172.16.1.20)' can't be established.
ED25519 key fingerprint is SHA256:NFPVfLNnzTOHVe0EA+xpUEG6dt/dvkX5+NyGIgc2m/Y.
Are you sure you want to continue connecting (yes/no/[fingerprint])?
```

**Type:** `yes`

**This adds the host to your `~/.ssh/known_hosts` file**

**Login prompt:**

```
labadmin@172.16.1.20's password:
```

**Enter password** set during Debian installation

**Welcome message:**

```
Linux bastion 6.1.0-43-amd64 #1 SMP PREEMPT_DYNAMIC Debian 6.1.162-1 (2026-02-08) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
labadmin@bastion:~$
```

✅ **SSH to Bastion Host: Working**

***

## Part 2: DC-01 SSH Configuration (Windows Server 2022 Core)

### Prerequisites

- DC-01 (172.16.1.10) running Windows Server 2022 Core
- Access to DC-01 console via Proxmox
- Administrator credentials

### Step 1: Access DC-01 Console

**Method:** Proxmox Web UI → VM 102 (DomainController) → Console

**You'll see:**

```
PS C:\Users\Administrator>
```

***

### Step 2: Check OpenSSH Availability

**Command:**

```powershell
Get-WindowsCapability -Online | Where-Object Name -like 'OpenSSH*'
```

**Output:**

```
Name  : OpenSSH.Client~~~~0.0.1.0
State : Installed

Name  : OpenSSH.Server~~~~0.0.1.0
State : Installed
```

**Result:** OpenSSH Client and Server available

***

### Step 3: Install OpenSSH Server

**Command:**

```powershell
Add-WindowsCapability -Online -Name OpenSSH.Server~~~~0.0.1.0
```

**Output:**

```
Path          :
Online        : True
RestartNeeded : False
```

**Result:** ✅ OpenSSH Server installed successfully

***

### Step 4: Start SSH Service

**Start the service:**

```powershell
Start-Service sshd
```

**Set to start automatically:**

```powershell
Set-Service -Name sshd -StartupType 'Automatic'
```

**Verify service status:**

```powershell
Get-Service sshd
```

**Output:**

```
Status   Name      DisplayName
------   ----      -----------
Running  sshd      OpenSSH SSH Server
```

**Result:** ✅ SSH service running and set to automatic startup

***

### Step 5: Configure Windows Firewall

**Create firewall rule:**

```powershell
New-NetFirewallRule -Name sshd -DisplayName 'OpenSSH**' -Enabled True -Direction Inbound -Protocol TCP -Action Allow -LocalPort 22
```

**Output:**

```
Name                  : sshd
DisplayName           : OpenSSH**
Description           :
DisplayGroup          :
Group                 :
Enabled               : True
Profile               : Any
Platform              : {}
Direction             : Inbound
Action                : Allow
EdgeTraversalPolicy   : Block
LooseSourceMapping    : False
LocalOnlyMapping      : False
Owner                 :
PrimaryStatus         : OK
Status                : The rule was parsed successfully from the store. (65536)
EnforcementStatus     : NotApplicable
PolicyStoreSource     : PersistentStore
PolicyStoreSourceType : Local
RemoteDynamicKeywordAddresses : {}
```

**Result:** ✅ Firewall rule created - SSH port 22 accessible

***

### Step 6: Test SSH Connection to DC-01

**From your local machine (with VPN connected):**

```bash
ssh administrator@172.16.1.10
```

**On first connection:**

```
The authenticity of host '172.16.1.10 (172.16.1.10)' can't be established.
ECDSA key fingerprint is SHA256:...
Are you sure you want to continue connecting (yes/no/[fingerprint])?
```

**Type:** `yes`

**Password prompt:**

```
administrator@172.16.1.10's password:
```

**Enter** Administrator password

**Welcome message:**

```
Microsoft Windows [Version 10.0.20348.2849]
(c) Microsoft Corporation. All rights reserved.

administrator@DC-01 C:\Users\Administrator>
```

✅ **SSH to DC-01: Working**

***

## SSH Access Summary

### Current Configuration

| System | IP | User | Authentication | Port | Status |
|--------|----|----|---------------|------|--------|
| **Bastion** | 172.16.1.20 | labadmin | SSH Keys (passwordless) | 22 | ✅ Working |
| **DC-01** | 172.16.1.10 | administrator | Password | 22 | ✅ Working |

### SSH Commands

```bash
# Connect to Bastion Host (passwordless with SSH key)
ssh labadmin@172.16.1.20

# Connect to DC-01 (password required)
ssh administrator@172.16.1.10

# Connect with specific key
ssh -i ~/.ssh/id_ed25519 labadmin@172.16.1.20

# Copy file to bastion
scp file.txt labadmin@172.16.1.20:~/

# Copy file from bastion
scp labadmin@172.16.1.20:~/file.txt ./
```

***

## Security Best Practices Implemented

### Bastion Host
✅ SSH key authentication enabled  
✅ Passwordless login for convenience  
✅ Host key verification (known_hosts)  
✅ ED25519 encryption (modern, secure)  

### DC-01
✅ SSH service installed and running  
✅ Firewall rule configured  
✅ Service set to automatic startup  
✅ Password authentication (can add keys later)  

***

## Network Flow

```
Your Mac/PC (with VPN)
    ↓ VPN Tunnel (10.0.0.x)
pfSense OpenVPN Server
    ↓ Internal Network (172.16.1.0/24)
    ├── Bastion (172.16.1.20:22) - SSH with keys
    └── DC-01 (172.16.1.10:22) - SSH with password
```

***

## Troubleshooting

### Issue: "Connection refused"

**Check service status:**

**Bastion:**
```bash
sudo systemctl status ssh
sudo systemctl start ssh
```

**DC-01:**
```powershell
Get-Service sshd
Start-Service sshd
```

### Issue: "Permission denied (publickey)"

**Bastion - check authorized_keys:**
```bash
ls -la ~/.ssh/authorized_keys
cat ~/.ssh/authorized_keys
```

**Permissions should be:**
- `~/.ssh/` directory: 700
- `~/.ssh/authorized_keys`: 600

### Issue: "Host key verification failed"

**Remove old host key:**
```bash
ssh-keygen -R 172.16.1.20
```

Then reconnect and accept new fingerprint.

***

## Optional: Copy SSH Key to DC-01

**For passwordless access to DC-01 (optional):**

```powershell
# On DC-01, create .ssh directory
mkdir C:\Users\Administrator\.ssh

# Copy your public key content to:
# C:\Users\Administrator\.ssh\authorized_keys

# Set proper permissions
icacls C:\Users\Administrator\.ssh\authorized_keys /inheritance:r
icacls C:\Users\Administrator\.ssh\authorized_keys /grant "Administrator:F"
```
