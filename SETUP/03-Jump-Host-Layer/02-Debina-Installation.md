## Debian 12 Installation Process

### Starting the Installation

**Step 1: Boot the VM**
- Start VM 101 (bastionHost) from Proxmox
- Open Console (noVNC viewer)
- Debian 12 installer boot menu appears

**Step 2: Select Installation Method**
- Boot Menu Options:
  - Graphical install
  - **Install** ✅ (Selected)
  - Advanced options
  - Help

**Recommendation:** Select **"Install"** (text-based installer)
- Lighter on resources (512 MB RAM)
- More stable for server installations
- Professional approach

**Action:** Arrow down to "Install" → Press **Enter**

***

### Low Memory Mode

**Notice Displayed:**
```
[!!!] Low memory

Entering low memory mode

This system has relatively little free memory, so it will enter low memory mode.
Among other things, this means that English will be the language used during
the installation and you should set up swap space as soon as possible.
```

**What This Means:**
- ✅ Normal and expected with 512 MB RAM
- Language locked to English during installation
- Installer will create swap space automatically
- Installation will proceed successfully

**Action:** Press **Enter** on <Continue>

***

### Location Selection

**Step 1: Select Continent**
- **Continent:** Asia
- **Reason:** System located in Lucknow, Uttar Pradesh, India

**Step 2: Select Country**
- **Country:** India
- Sets timezone to Asia/Kolkata
- Helps select appropriate mirrors

**Action:** Use arrow keys to select, press **Enter**

***

### Network Configuration

#### Hostname Configuration

**Prompt:** Please enter the hostname for this system

**Hostname:** `bastion`

**Explanation:**
- Identifies the server as bastion/jump host
- Short, simple, and descriptive
- Follows naming convention (pfSense, DC-01, bastion)

**Action:** Type `bastion` → Press **Enter**

#### Domain Name Configuration

**Prompt:** Please enter the domain name

**Domain Name:** `lab.internal`

**Explanation:**
- Maintains consistency across lab infrastructure
- `.internal` TLD reserved for internal networks
- **Full FQDN:** bastion.lab.internal

**Full Hostname Structure:**
```
pfSense:  pfSense.lab.internal
DC-01:    dc-01.lab.internal (will be configured)
Bastion:  bastion.lab.internal
```

**Action:** Type `lab.internal` → Press **Enter**

***

### User Account Setup

#### Root Password

**Prompt:** Set root password

**Action:** Enter and confirm root password
- Used for system administration
- Keep secure and document safely

#### Regular User Account

**Prompt:** Full name for the new user

**Full Name:** `labadmin`

**Details:**
- **Username:** labadmin (auto-generated)
- **Password:** Set during installation
- **Purpose:** Non-root administrative user (security best practice)

**Why Create Regular User?**
- Avoid using root for daily tasks
- Better security posture
- Sudo access for administrative tasks
- Audit trail for user actions

**Action:** Type `labadmin` → Press **Enter** → Set password

***

### Disk Partitioning

#### Partitioning Method

**Options Available:**
1. Guided - use entire disk ✅ (Selected)
2. Guided - use entire disk and set up LVM
3. Guided - use entire disk and set up encrypted LVM
4. Manual

**Selected:** Guided - use entire disk

**Why This Choice?**
- Simple and straightforward
- Automatic swap creation (critical for 512 MB RAM)
- Suitable for single-disk bastion host
- No need for LVM complexity in lab

**Action:** Press **Enter** on "Guided - use entire disk"

#### Disk Selection

**Disk:** SCSI3 (0,0,0) (sda) - 21.5 GB (20 GB configured)

**Partitioning Scheme:** All files in one partition (recommended)

**Auto-Created Layout:**
```
/dev/sda1  -  Boot/EFI partition
/dev/sda2  -  Swap partition (~512 MB-1 GB)
/dev/sda3  -  Root partition (/) - remaining ~18-19 GB
```

**Confirmation:** Write changes to disk? **Yes**

**Action:** Select **Yes** → Press **Enter**

***

### Base System Installation

**Phase:** Installing the base system

**Process:**
1. Formatting partitions
2. Installing core Debian packages
3. Configuring package manager (APT)
4. Installing kernel and bootloader

**Progress Example:**
- "Unpacking libuuid1:amd64..." (40%)
- Installing base system packages
- Configuring system components

**Duration:** ~5-10 minutes (depends on network speed with netinst ISO)

**Status:** Wait for completion...

***

### Package Manager Configuration

#### Mirror Selection

**Step 1:** Scan another installation medium? → **No**

**Step 2:** Debian archive mirror country → **India** (or appropriate for your location)

**Step 3:** Mirror selection → **deb.debian.org** (default)

**Step 4:** HTTP proxy → **(leave blank)** for direct connection

**Process:** Configuring apt...scanning the mirror...

***

### ⚠️ Critical Issue: DNS Resolution Failure

#### Problem Encountered

**Error Message:**
```
[!!!] Bad archive mirror

An error has been detected while trying to use the specified Debian archive mirror.

Possible reasons:
- Incorrect mirror specified
- Mirror not available (unreliable network connection)
- Mirror does not support correct Debian version
```

**Root Cause:**
- Bastion installer couldn't reach Debian mirrors
- DNS resolution failing
- pfSense DNS configuration issue

#### The Fix (Performed on pfSense)

**Problem Identified:**
- pfSense DNS Servers had **172.16.1.10 (DC-01) as PRIMARY**
- DC-01 had NO DNS service running
- All DNS queries failed at first server

**Solution:**
1. Accessed pfSense Web UI: `https://192.168.1.101`
2. Navigated to **System → General Setup**
3. **Removed 172.16.1.10** from DNS Servers list
4. Kept only **8.8.8.8** (Google DNS)
5. Clicked **Save**

**Result:** ✅ DNS resolution working, DHCP clients now get correct DNS

#### Installation Retry

**Action:** Pressed **<Go Back>** on error screen

**Result:** Installer automatically retried mirror configuration

**Status:** ✅ **SUCCESS!** - Debian mirror accessible, installation continuing

***

### Package Popularity Contest

**Prompt:** Participate in the package usage survey?

**Purpose:** Anonymously send package usage statistics to Debian project

**Selected:** **No** (typical for lab/internal systems)

**Note:** Optional, doesn't affect functionality

**Action:** Tab to <No> → Press **Enter**

***

### Software Selection (Critical Step)

**Prompt:** Choose software to install

#### Initial Selection (Incorrect for Bastion)
```
[*] Debian desktop environment  ← SELECTED (need to remove)
[ ] ... GNOME
[ ] ... Xfce
[ ] ... GNOME Flashback
[ ] ... KDE Plasma
[ ] ... Cinnamon
[ ] ... MATE
[ ] ... LXDE
[ ] ... LXQt
[ ] web server
[ ] SSH server                   ← NOT SELECTED (need to add!)
[*] standard system utilities    ← SELECTED (keep this)
```

#### Required Changes for Jump Host

**1. UNSELECT Debian desktop environment**
- Navigate to "Debian desktop environment"
- Press **Space** to unselect (remove asterisk)

**Why Remove Desktop?**
- No GUI needed for SSH-only server
- Saves ~2-3 GB RAM
- Reduces attack surface
- Faster performance
- Professional server practice

**2. SELECT SSH server**
- Navigate to "SSH server"  
- Press **Space** to select (add asterisk)

**Why SSH Server is Essential?**
- **Core purpose** of jump/bastion host
- Enables remote access
- Required for administration
- Main service for the server

**3. KEEP standard system utilities**
- Already selected (has asterisk)
- Provides basic command-line tools
- Essential system management utilities

#### Final Configuration
```
[ ] Debian desktop environment  (UNSELECTED) ✅
[ ] SSH server                  (SELECTED)   ✅
[*] standard system utilities   (SELECTED)   ✅
```

**Action:** Tab to <Continue> → Press **Enter**

***

### Package Installation

**Phase:** Installing selected software

**Process:**
- Downloading packages from Debian mirror
- Installing SSH server (openssh-server)
- Installing standard system utilities
- Configuring services

**Duration:** ~10-15 minutes (netinst downloads packages over network)

**Packages Installed:**
- SSH daemon (sshd)
- Basic networking tools
- Text editors (vim, nano)
- System utilities
- Essential command-line tools

**Status:** Wait for completion...

***

### GRUB Bootloader Installation

**Prompt:** Install the GRUB boot loader to your primary drive?

**Selected:** **Yes**

**Device:** /dev/sda (primary disk)

**Process:**
- Installing GRUB bootloader
- Configuring boot menu
- Making system bootable

**Action:** Press **Enter** on **Yes**

***

### Installation Complete

**Message:** Installation complete!

**Prompt:** Continue to reboot

**Action:** Remove installation media → Press **Enter**

**Process:**
1. System reboots automatically
2. Boot from installed system
3. Login prompt appears

***

## Post-Installation: First Boot

### Login Screen

```
Debian GNU/Linux 12 bastion tty1

bastion login: _
```

**System Information:**
- Hostname: bastion
- Kernel: Linux 6.1.0-43-amd64
- Distribution: Debian GNU/Linux 12 (Bookworm)

### Initial Login

**Login as:** `root`
- Enter root password set during installation

**Welcome Message:**
```
Linux bastion 6.1.0-43-amd64 #1 SMP PREEMPT_DYNAMIC Debian 6.1.162-1 (2026-02-08) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
root@bastion:~# _
```

**Status:** ✅ System successfully installed and operational

***

## Network Configuration - Setting Static IP

### Check Current Network Status

**Command:**
```bash
ip addr show
```

**Output:**
```
2: enp6s18: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP
    link/ether bc:24:11:ea:00:94 brd ff:ff:ff:ff:ff:ff
    inet 172.16.1.12/24 brd 172.16.1.255 scope global dynamic enp6s18
```

**Current Configuration:**
- Interface: **enp6s18**
- IP: 172.16.1.12/24 (DHCP-assigned)
- Status: UP and operational

***

### Configure Static IP Address

#### Step 1: Edit Network Interfaces File

**Command:**
```bash
nano /etc/network/interfaces
```

#### Step 2: Modify Configuration

**Find the DHCP configuration:**
```
allow-hotplug enp6s18
iface enp6s18 inet dhcp
```

**Replace with static configuration:**
```
allow-hotplug enp6s18
iface enp6s18 inet static
    address 172.16.1.20
    netmask 255.255.255.0
    gateway 172.16.1.1
    dns-nameservers 8.8.8.8
```

**Configuration Explained:**
- `address 172.16.1.20` - Static IP for bastion host
- `netmask 255.255.255.0` - /24 subnet  
- `gateway 172.16.1.1` - pfSense LAN interface
- `dns-nameservers 8.8.8.8` - Google DNS (reliable)

#### Step 3: Save and Exit

- Press **Ctrl+X** to exit
- Press **Y** to confirm save
- Press **Enter** to confirm filename

#### Step 4: Restart Networking

**Command:**
```bash
systemctl restart networking
```

**Alternative (if above fails):**
```bash
reboot
```

***

### Verify Static IP Configuration

#### Check IP Address

**Command:**
```bash
ip addr show
```

**Expected Output:**
```
2: enp6s18: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP
    link/ether bc:24:11:6c:5e:86 brd ff:ff:ff:ff:ff:ff
    inet 172.16.1.20/24 brd 172.16.1.255 scope global enp6s18
```

**Verification:** ✅ IP changed from 172.16.1.12 to **172.16.1.20**

#### Test Network Connectivity

**Test 1: Ping Gateway (pfSense)**
```bash
ping 172.16.1.1
```
**Result:** ✅ Success - packets transmitted and received

**Test 2: Ping Internet (Google DNS)**
```bash
ping 8.8.8.8
```

**Output:**
```
PING 8.8.8.8 (8.8.8.8) 56(84) bytes of data.
64 bytes from 8.8.8.8: icmp_seq=1 ttl=117 time=12.4 ms
64 bytes from 8.8.8.8: icmp_seq=2 ttl=117 time=11.4 ms
...
--- 8.8.8.8 ping statistics ---
22 packets transmitted, 22 received, 0% packet loss, time 21037ms
rtt min/avg/max/mdev = 11.179/11.648/12.383/0.247 ms
```

**Result:** ✅ **SUCCESS** - Internet connectivity confirmed, 0% packet loss

***

## Final System Status

### System Information

**Hostname:** bastion.lab.internal  
**IP Address:** 172.16.1.20/24 (Static)  
**Gateway:** 172.16.1.1 (pfSense)  
**DNS:** 8.8.8.8  
**OS:** Debian GNU/Linux 12 (Bookworm)  
**Kernel:** 6.1.0-43-amd64  
**Services:** SSH Server, Standard Utilities  

### Connectivity Status

✅ **Local Network:** Reachable (ping 172.16.1.1 successful)  
✅ **Internet:** Reachable (ping 8.8.8.8 successful)  
✅ **DNS:** Working (using 8.8.8.8)  
✅ **SSH Server:** Running (installed and enabled)
