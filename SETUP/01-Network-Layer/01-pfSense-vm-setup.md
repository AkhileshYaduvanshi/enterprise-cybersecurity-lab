# pfSense VM Creation in Proxmox - Complete Guide

## Overview

pfSense is a free, open-source firewall and router based on FreeBSD. It provides enterprise-grade features for your lab:
- Firewall with stateful filtering
- Network address translation (NAT)
- VPN (OpenVPN, WireGuard)
- DHCP server
- DNS forwarding
- Advanced routing

## VM Specifications

Your pfSense VM is configured for optimal performance in a resource-constrained lab environment:

M Configuration Summary

VM ID 100
Name pfSense
Node pve

SYSTEM
Machine Type q35 (modern chipset)
BIOS/Firmware SeaBIOS
Qemu Agent Enabled
TPM Not enabled

CPU & MEMORY
CPU Sockets 1
CPU Cores 1 (total 1 vCPU)
CPU Type x86-64-v2-AES
Memory 1024 MiB (1 GB)

STORAGE
Boot Disk scsi0
Size 32 GB
Storage Backend local-lvm
Filesystem Format Raw disk image (raw)
Controller VirtIO SCSI single
Optimizations Discard enabled, I/O thread enabled

OPERATING SYSTEM
OS pfSense 2.8.1-RELEASE
ISO netgate-installer-v1.1.1-RELEASE-amd64.iso
Boot Device CD/DVD (ide2)

NETWORK INTERFACES
Interface 1 (WAN) net0 → vmbr0 (management network)
Model: VirtIO (paravirtualized)
MAC: auto-generated
Firewall: Enabled

Interface 2 (LAN) net1 → vmbr1 (lab network)
Model: VirtIO (paravirtualized)
MAC: auto-generated
Firewall: Enabled


## Why These Settings?

### CPU: 1 Core (Not 2)
- **Why 1 core:** pfSense is I/O-bound, not CPU-bound
- **Packet processing:** Single core handles routing/filtering efficiently
- **Resource savings:** Leaves more cores for DC-01, Wazuh, Jenkins
- **Lab environment:** Sufficient for firewall duties in internal network

**Note:** If you add heavy VPN usage or advanced rules, can increase to 2 cores later.

### Memory: 1 GB
- **Base pfSense:** 256-512 MB minimum
- **VPN Support:** +256-512 MB for encryption/decryption
- **Logs & State Tables:** +256 MB for connections tracking
- **Total:** 1 GB is optimal for lab firewall with VPN
- **Headroom:** Not excessive, balanced for your 16GB system

### Storage: 32 GB
- **pfSense install:** ~2-3 GB
- **Logs:** Room for 6-12 months of logs (depending on traffic)
- **RRD graphs:** Historical traffic graphs stored locally
- **Backups:** Configuration backups (optional)
- **Safe margin:** 32 GB ensures no disk space issues

### Machine Type: q35
- **Modern:** Latest Intel Q35 chipset (recommended)
- **Performance:** Better than legacy i440fx
- **Consistency:** Matches DC-01 and future VMs
- **Compatibility:** Works perfectly with FreeBSD/pfSense

### Network Interfaces: Two VirtIO NICs
- **net0 (WAN):** vmbr0 → Connects to management network (192.168.1.x)
  - Gets IP from home router or static assignment
  - Communicates with Proxmox host and external networks
  
- **net1 (LAN):** vmbr1 → Connects to lab network (172.16.1.x)
  - Will be configured as 172.16.1.1 (gateway)
  - All lab VMs (DC-01, etc) use this as default gateway

**VirtIO Model:** Best performance for network I/O, low overhead

---

## Step-by-Step: pfSense VM Creation in Proxmox

### Step 1: Access Proxmox Web UI

1. Open browser and navigate to `https://192.168.1.100:8006/`
2. Login with `root` and your Proxmox password
3. You're now in the Proxmox VE dashboard

### Step 2: Create New VM

1. Click **"Create VM"** button (top right)
2. The VM creation wizard opens

### Step 3: General Tab

**Values to Enter:**
- **Node:** `pve` (already selected)
- **VM ID:** `100`
- **Name:** `pfSense`
- **Add to HA:** leave unchecked
- **Resource Pool:** leave empty

**Click: Next**

### Step 4: OS Tab

**Values to Select:**
- **Use CD/DVD disc image file (iso):** selected
- **Storage:** `local` (where ISOs are stored)
- **ISO image:** `netgate-installer-v1.1.1-RELEASE-amd64.iso`
- **Guest OS Type:** `Linux`
- **Version:** `6.x - 2.6 Kernel`

**Why Linux type?** FreeBSD (pfSense base) is compatible with Linux kernel option in Proxmox

**Click: Next**

### Step 5: System Tab

**Values to Configure:**
- **Graphic card:** `Default`
- **Machine:** Change to `q35` (important for modern performance)
- **Firmware (BIOS):** `Default (SeaBIOS)`
- **SCSI Controller:** `VirtIO SCSI single`
- **Qemu Agent:** Check ✓ (enables better integration)
- **Add TPM:** Leave unchecked (not needed for firewall)

**Click: Next**

### Step 6: Disks Tab

**Values to Configure:**
- **Bus/Device:** `SCSI`
- **SCSI Controller:** `VirtIO SCSI single`
- **Storage:** `local-lvm`
- **Disk size (GiB):** `32`
- **Format:** `Raw disk image (raw)`
- **Cache:** `Default (No cache)`
- **Discard:** Check ✓ (helps with thin provisioning)
- **IO thread:** Check ✓ (improves I/O performance)

**Click: Next**

### Step 7: CPU Tab

**Values to Configure:**
- **Sockets:** `1`
- **Cores:** `1` (keep minimal, firewall doesn't need much CPU)
- **CPU Type:** `x86-64-v2-AES`

**Click: Next**

### Step 8: Memory Tab

**Values to Configure:**
- **Memory (MiB):** `1024` (1 GB - perfect for pfSense with VPN)
- **Minimum memory:** leave empty (no ballooning)

**Click: Next**

### Step 9: Network Tab

**First Interface (WAN):**
- **Bridge:** `vmbr0` (management network)
- **Model:** `VirtIO (paravirtualized)`
- **VLAN Tag:** leave empty
- **Firewall:** Check ✓
- **MAC address:** `auto`

**Do NOT click Next yet!** We need to add second interface.

**Add Second Interface (LAN):**
- Look for an "Add" button or similar option
- If not available in this tab, we'll add after VM creation ✓

**Click: Next**

### Step 10: Confirm Tab

Review all settings in the summary table. Should show:

cores 1
cpu x86-64-v2-AES
ide2 local:iso/netgate-installer...
machine q35
memory 1024
name pfSense
net0 virtio,bridge=vmbr0,firewall=1
nodename pve
ostype l26
scsi0 local-lvm:32,discard=on,iothread=on
scsihw virtio-scsi-single
sockets 1
vmid 100

**Click: Finish** to create the VM

### Step 11: Add Second Network Interface (LAN)

After VM is created:

1. Click on **VM 100 (pfSense)** in the left panel
2. Go to **Hardware** tab
3. Look for "Add" button to add a device
4. Select "Network Device"
5. Configure as:
   - **Bridge:** `vmbr1` (lab network)
   - **Model:** `VirtIO (paravirtualized)`
   - **Firewall:** Check ✓
   - **MAC address:** `auto`
6. Click **Add**

Now pfSense has both interfaces:
- **net0:** WAN (vmbr0 - management)
- **net1:** LAN (vmbr1 - lab network)

---

## Verification

After VM creation, verify in Proxmox:

1. **VM 100 (pfSense)** appears in left panel ✓
2. **Summary tab** shows:
   - Memory: 1.00 GiB
   - Processors: 1 (1 sockets, 1 core)
   - Status: Stopped (ready for install)
3. **Hardware tab** shows:
   - Network Device (net0) on vmbr0
   - Network Device (net1) on vmbr1
   - scsi0 disk (32GB)
   - ide2 CD drive (pfSense ISO)

---

## Resource Allocation Summary

Your lab uses these resources for pfSense:

CPU: 1 out of available cores (light footprint)
RAM: 1 GB out of 16 GB (6.25% of available)
Storage: 32 GB allocated (actual use ~3-5 GB)

Leaves available for:

DC-01: 2 cores, 2 GB RAM

Wazuh: 2 cores, 2 GB RAM

Jenkins: 2 cores, 2 GB RAM

Others: remaining resources

---

## Troubleshooting

### Issue: VM won't start
- Check if Proxmox host has sufficient resources
- Verify ISO file exists and is accessible

### Issue: Network interfaces not visible
- Ensure bridges (vmbr0, vmbr1) exist in Proxmox Network configuration
- Verify network bridges are properly configured on the host

### Issue: Too slow during installation
- Allocate more vCPU cores temporarily (can reduce later)
- Check host storage I/O isn't bottlenecked
