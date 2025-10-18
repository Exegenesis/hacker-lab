# Troubleshooting Guide

**Comprehensive troubleshooting guide for your cybersecurity home lab**

---

## Quick Diagnostic Checklist

When something isn't working, follow this systematic approach:

### 1. Basic Checks
- [ ] Is the VM powered on?
- [ ] Is the network adapter enabled in VM settings?
- [ ] Is VirtualBox up to date?
- [ ] Are Guest Additions installed?
- [ ] Is there enough RAM/CPU allocated?
- [ ] Is there enough disk space?

### 2. Network Checks
- [ ] Can you ping the gateway?
- [ ] Can you ping other VMs on the same network?
- [ ] Is DNS resolving correctly?
- [ ] Are the network adapters attached to the correct networks?
- [ ] Is the firewall blocking connections?

### 3. Service Checks
- [ ] Is the service running? (`systemctl status` or Services.msc)
- [ ] Are there any error messages in logs?
- [ ] Are the required ports open?
- [ ] Is the service configured correctly?

---

## Common Issues and Solutions

### VirtualBox Issues

#### Issue: "VT-x/AMD-V is not available"

**Symptoms:**
- VM fails to start
- Error message mentions VT-x or AMD-V

**Solutions:**

1. **Enable in BIOS/UEFI**
   ```
   Steps:
   1. Restart computer
   2. Press Del/F2/F12 during boot (varies by manufacturer)
   3. Navigate to CPU Configuration or Advanced Settings
   4. Enable "Intel VT-x" or "AMD-V"
   5. Enable "Intel VT-d" or "AMD IOMMU" (if available)
   6. Save and exit (usually F10)
   ```

2. **Disable Hyper-V (Windows only)**
   ```powershell
   # Run as Administrator
   bcdedit /set hypervisorlaunchtype off
   # Restart computer
   ```

3. **Check if another hypervisor is running**
   ```
   - VMware Workstation
   - Docker Desktop with Hyper-V
   - Windows Subsystem for Linux 2 (WSL2)
   
   Disable these or use VirtualBox in compatibility mode
   ```

#### Issue: "VM runs very slowly"

**Symptoms:**
- Sluggish performance
- High CPU usage on host
- Laggy mouse/keyboard

**Solutions:**

1. **Increase VM Resources**
   ```
   - Shutdown VM
   - Settings → System → Motherboard
   - Increase Base Memory (2-4 GB minimum per VM)
   - Settings → System → Processor
   - Increase Processors (2-4 CPUs)
   ```

2. **Enable 3D Acceleration**
   ```
   - Settings → Display → Screen
   - Video Memory: 128 MB
   - Graphics Controller: VMSVGA
   - Enable 3D Acceleration: Checked
   ```

3. **Optimize Host System**
   ```
   - Close unnecessary applications
   - Ensure host has adequate cooling
   - Check for malware/resource hogs
   - Consider upgrading host RAM
   ```

4. **Use SSD for VM Storage**
   ```
   - VMs should be on SSD, not HDD
   - Move VM files if necessary
   - Use VDI format with dynamic allocation
   ```

#### Issue: "Network adapter failed to start"

**Symptoms:**
- Network adapter shows as disconnected
- No IP address assigned
- Cannot ping anything

**Solutions:**

1. **Check Network Settings**
   ```
   - Shutdown VM
   - Settings → Network
   - Verify adapter is enabled
   - Check "Cable Connected" is checked
   - Verify correct network type (NAT, Internal, Host-only)
   ```

2. **Reinstall Guest Additions**
   ```
   Linux:
   sudo apt remove virtualbox-guest-*
   # Reboot VM
   # Devices → Insert Guest Additions CD
   cd /media/cdrom
   sudo ./VBoxLinuxAdditions.run
   sudo reboot
   
   Windows:
   # Uninstall Guest Additions from Control Panel
   # Reboot
   # Devices → Insert Guest Additions CD
   # Run VBoxWindowsAdditions.exe
   # Reboot
   ```

3. **Reset Network Stack**
   ```bash
   # Linux
   sudo systemctl restart NetworkManager
   sudo ip link set eth0 down
   sudo ip link set eth0 up
   sudo dhclient eth0
   ```
   
   ```powershell
   # Windows (as Administrator)
   netsh int ip reset
   netsh winsock reset
   ipconfig /release
   ipconfig /renew
   ```

#### Issue: "Shared clipboard/drag-and-drop not working"

**Symptoms:**
- Cannot copy/paste between host and guest
- Cannot drag files between host and guest

**Solutions:**

1. **Verify Guest Additions**
   ```
   - Guest Additions must be installed
   - Check: Devices → Shared Clipboard → Bidirectional
   - Check: Devices → Drag and Drop → Bidirectional
   ```

2. **Restart VBoxClient (Linux guests)**
   ```bash
   killall VBoxClient
   VBoxClient --clipboard
   VBoxClient --draganddrop
   ```

3. **Restart VM**
   ```
   Sometimes a simple reboot fixes it
   ```

---

### Network Issues

#### Issue: "Cannot ping other VMs"

**Symptoms:**
- Ping requests timeout
- No connectivity between VMs

**Diagnostic Steps:**

1. **Verify Network Configuration**
   ```
   Both VMs on same network? (e.g., both on "intnet")
   Check: VM Settings → Network → Adapter 1
   
   Network types must match:
   - Internal Network: VMs can talk to each other
   - NAT Network: VMs can talk to each other and internet
   - NAT: VMs isolated from each other
   - Host-only: VMs and host can communicate
   ```

2. **Check IP Addresses**
   ```bash
   # Linux
   ip addr show
   # Should show IP in correct subnet
   
   # Windows
   ipconfig
   # Should show IP in correct subnet
   ```

3. **Check Firewalls**
   ```bash
   # Linux (temporarily disable for testing)
   sudo ufw disable
   
   # Windows (temporarily disable for testing)
   netsh advfirewall set allprofiles state off
   
   # Don't forget to re-enable!
   ```

4. **Verify Routes**
   ```bash
   # Linux
   ip route show
   
   # Windows
   route print
   ```

**Solutions:**

1. **Correct Network Configuration**
   ```
   Recommended setup:
   - pfSense em0 (WAN): NAT
   - pfSense em1 (LAN): Internal Network "intnet"
   - pfSense em2 (OPT1): Host-only Adapter
   - Windows Server: Internal Network "intnet"
   - Windows Client: Internal Network "intnet"
   - Metasploitable: Internal Network "intnet"
   - Kali Linux: Host-only Adapter
   ```

2. **Set Static IPs Correctly**
   ```
   Ensure all VMs have:
   - Correct IP address for their subnet
   - Correct subnet mask
   - Correct gateway
   - Correct DNS servers
   ```

#### Issue: "No internet access from VMs"

**Symptoms:**
- Can ping internal IPs but not internet
- Cannot download updates
- DNS lookups fail for internet domains

**Solutions:**

1. **Check pfSense WAN**
   ```
   pfSense Console:
   - Option 1: View interfaces
   - Verify WAN (em0) has an IP address
   - If not, try DHCP renewal
   
   pfSense Web Interface:
   - Status → Interfaces
   - Check WAN status
   ```

2. **Check Gateway Configuration**
   ```bash
   # From client VMs
   # Linux
   ip route show
   # Should show default via 192.168.1.1
   
   # Windows
   route print
   # Should show 0.0.0.0 via 192.168.1.1
   ```

3. **Check DNS**
   ```bash
   # Test DNS resolution
   nslookup google.com
   
   # If fails, check DNS settings
   # Linux
   cat /etc/resolv.conf
   # Should contain nameserver entries
   
   # Windows
   ipconfig /all
   # Check DNS Servers line
   ```

4. **Check pfSense Firewall Rules**
   ```
   Web Interface → Firewall → Rules → LAN
   - Ensure rule allows LAN to Any
   - Action should be "Pass"
   - Destination should be "Any"
   ```

#### Issue: "DNS not resolving domain names"

**Symptoms:**
- Can ping IPs but not hostnames
- `nslookup` fails
- Web browsing doesn't work

**Solutions:**

1. **Check DNS Server Settings**
   ```bash
   # Linux
   cat /etc/resolv.conf
   # Should show DNS servers
   
   # If empty or wrong, edit:
   sudo nano /etc/resolv.conf
   # Add:
   nameserver 192.168.1.10
   nameserver 192.168.1.1
   nameserver 8.8.8.8
   ```
   
   ```powershell
   # Windows
   Get-DnsClientServerAddress
   
   # Set DNS
   Set-DnsClientServerAddress -InterfaceAlias "Ethernet" -ServerAddresses 192.168.1.10,192.168.1.1
   ```

2. **Verify DNS Server is Running**
   ```
   # On Domain Controller (Windows Server)
   Get-Service DNS
   # Should be Running
   
   # If stopped:
   Start-Service DNS
   ```

3. **Test DNS Directly**
   ```bash
   # Test against specific server
   nslookup google.com 8.8.8.8
   # If this works but normal nslookup doesn't, DNS settings are wrong
   
   # Test against DC
   nslookup labdomain.local 192.168.1.10
   # If this fails, DC DNS not configured correctly
   ```

---

### Windows Active Directory Issues

#### Issue: "Cannot join domain"

**Symptoms:**
- Error: "The specified domain either does not exist or could not be contacted"
- Error: "The trust relationship between this workstation and the primary domain failed"

**Solutions:**

1. **Verify Network Connectivity**
   ```powershell
   # Test connectivity to DC
   Test-Connection 192.168.1.10
   
   # Should reply successfully
   ```

2. **Verify DNS Configuration**
   ```powershell
   # Check DNS servers
   Get-DnsClientServerAddress
   
   # Primary should be DC (192.168.1.10)
   # If not, set it:
   Set-DnsClientServerAddress -InterfaceAlias "Ethernet" -ServerAddresses 192.168.1.10,192.168.1.1
   ```

3. **Test DNS Resolution**
   ```powershell
   # Resolve domain
   Resolve-DnsName labdomain.local
   
   # Should return DC's IP (192.168.1.10)
   
   # If fails, DNS is misconfigured
   ```

4. **Synchronize Time**
   ```powershell
   # Check time difference
   # Client and DC must be within 5 minutes
   
   # Sync time
   w32tm /resync
   
   # Or configure NTP
   w32tm /config /manualpeerlist:192.168.1.10 /syncfromflags:manual /reliable:yes /update
   w32tm /resync
   ```

5. **Try Join Again with Full Domain Name**
   ```powershell
   Add-Computer -DomainName labdomain.local -Credential (Get-Credential) -Restart
   
   # Enter LABDOMAIN\Administrator when prompted
   ```

#### Issue: "Domain user cannot login to workstation"

**Symptoms:**
- "The trust relationship between this workstation and the primary domain failed"
- "No logon servers available"

**Solutions:**

1. **Re-join Domain**
   ```powershell
   # Remove from domain (local admin required)
   Remove-Computer -UnjoinDomainCredential (Get-Credential) -WorkgroupName WORKGROUP -Restart
   
   # After restart, join again
   Add-Computer -DomainName labdomain.local -Credential (Get-Credential) -Restart
   ```

2. **Reset Computer Account**
   ```powershell
   # On Domain Controller
   Reset-ComputerMachinePassword -Server DC01 -Credential LABDOMAIN\Administrator
   ```

3. **Check DNS and Connectivity Again**
   ```powershell
   Test-Connection 192.168.1.10
   Resolve-DnsName labdomain.local
   ```

#### Issue: "Group Policy not applying"

**Symptoms:**
- Settings not being enforced
- `gpupdate` doesn't apply policies

**Solutions:**

1. **Force Group Policy Update**
   ```powershell
   gpupdate /force
   
   # Check results
   gpresult /r
   ```

2. **Check Network Location**
   ```
   # Network must be "Domain" not "Public" or "Private"
   # Settings → Network & Internet → Ethernet
   # Should show "Domain network"
   ```

3. **Verify Time Sync**
   ```powershell
   w32tm /query /status
   # If not synced:
   w32tm /resync
   ```

4. **Check DNS**
   ```powershell
   nslookup labdomain.local
   # Should resolve to DC
   ```

---

### Kali Linux Issues

#### Issue: "Tools not working or missing"

**Symptoms:**
- Command not found errors
- Tools crashing
- Missing dependencies

**Solutions:**

1. **Update System**
   ```bash
   sudo apt update
   sudo apt full-upgrade -y
   sudo apt autoremove -y
   sudo reboot
   ```

2. **Reinstall Tool**
   ```bash
   # Remove tool
   sudo apt remove <tool-name> --purge
   
   # Install tool
   sudo apt install <tool-name> -y
   ```

3. **Fix Broken Packages**
   ```bash
   sudo apt install -f
   sudo dpkg --configure -a
   ```

4. **Update Tool Databases**
   ```bash
   # Metasploit
   sudo msfdb reinit
   
   # Locate
   sudo updatedb
   
   # Man pages
   sudo mandb
   ```

#### Issue: "Metasploit database errors"

**Symptoms:**
- "Database not connected"
- "Failed to connect to the database"

**Solutions:**

1. **Check Database Status**
   ```bash
   sudo msfdb status
   ```

2. **Initialize Database**
   ```bash
   sudo msfdb init
   ```

3. **Reinitialize if Needed**
   ```bash
   sudo msfdb delete
   sudo msfdb init
   ```

4. **Start PostgreSQL Manually**
   ```bash
   sudo systemctl start postgresql
   sudo systemctl enable postgresql
   ```

5. **Verify in Metasploit**
   ```bash
   msfconsole
   db_status
   # Should show "Connected to msf"
   ```

#### Issue: "OpenVAS/GVM won't start"

**Symptoms:**
- Services fail to start
- Web interface inaccessible
- Errors about missing feeds

**Solutions:**

1. **Check Setup**
   ```bash
   sudo gvm-check-setup
   # Follow any recommendations
   ```

2. **Update Feeds**
   ```bash
   # This takes a long time (hours)
   sudo greenbone-feed-sync --type GVMD_DATA
   sudo greenbone-feed-sync --type SCAP
   sudo greenbone-feed-sync --type CERT
   ```

3. **Restart Services**
   ```bash
   sudo gvm-stop
   sudo gvm-start
   ```

4. **Check Service Status**
   ```bash
   sudo systemctl status ospd-openvas
   sudo systemctl status gvmd
   sudo systemctl status gsad
   ```

5. **Reset Admin Password**
   ```bash
   sudo gvmd --user=admin --new-password=newpassword
   ```

---

### pfSense Issues

#### Issue: "Can't access web interface"

**Symptoms:**
- Browser can't connect to https://192.168.56.2
- Timeout or connection refused

**Solutions:**

1. **Verify IP Address**
   ```
   pfSense Console → Option 2: Set interface IP address
   - Verify OPT1 (em2) is 192.168.56.2
   ```

2. **Check Host Network**
   ```bash
   # From host machine
   # Verify host-only adapter exists
   
   # Windows
   ipconfig | findstr 192.168.56
   
   # Linux
   ip addr | grep 192.168.56
   
   # Should show host IP (usually 192.168.56.1)
   ```

3. **Test Connectivity**
   ```bash
   ping 192.168.56.2
   # Should respond
   ```

4. **Check Firewall**
   ```
   Temporarily disable host firewall to test
   
   Windows: Windows Defender Firewall → Turn off
   Linux: sudo ufw disable
   
   Try accessing again, then re-enable firewall
   ```

5. **Reset Web Configurator**
   ```
   pfSense Console → Option 11: Restart webConfigurator
   Wait for service restart
   Try accessing again
   ```

6. **Try Different Browser**
   ```
   Chrome, Firefox, Edge
   Use incognito/private mode
   ```

#### Issue: "pfSense not routing traffic"

**Symptoms:**
- VMs can't access internet through pfSense
- Internal network isolated

**Solutions:**

1. **Check Interface Assignments**
   ```
   pfSense Console → Option 1: Assign Interfaces
   
   Verify:
   - WAN = em0 (NAT)
   - LAN = em1 (Internal Network)
   - OPT1 = em2 (Host-only)
   ```

2. **Verify Gateway**
   ```
   Web Interface:
   - System → Routing → Gateways
   - WAN gateway should be online
   ```

3. **Check NAT Rules**
   ```
   Web Interface:
   - Firewall → NAT → Outbound
   - Should have automatic outbound NAT rules
   - If not, switch to "Automatic outbound NAT"
   ```

4. **Check Firewall Rules**
   ```
   Web Interface:
   - Firewall → Rules → LAN
   - Should have rule allowing LAN to Any
   - If not, add rule:
     - Action: Pass
     - Source: LAN net
     - Destination: any
   ```

5. **Test from pfSense**
   ```
   pfSense Console → Option 7: Ping host
   - Try pinging 8.8.8.8
   - If pfSense can't ping, WAN problem
   - If pfSense can ping but VMs can't, routing/firewall problem
   ```

---

### Performance Issues

#### Issue: "Lab very slow overall"

**Symptoms:**
- Everything runs slowly
- High host CPU/RAM usage
- VMs frequently freeze

**Solutions:**

1. **Don't Run All VMs at Once**
   ```
   Run only what you need:
   - Minimum: Kali + 1 target
   - Recommended: pfSense + Kali + 1-2 targets
   - Full lab: Only when needed
   ```

2. **Increase Host RAM**
   ```
   16 GB minimum
   32 GB recommended
   64 GB ideal
   ```

3. **Optimize VM Settings**
   ```
   Each VM:
   - Settings → System → Motherboard
     - Disable Floppy
     - Enable I/O APIC
   - Settings → System → Processor
     - Enable PAE/NX
     - Max CPUs = host physical cores - 1
   - Settings → Display
     - Video Memory: 128 MB
   - Settings → Storage
     - Enable Host I/O Cache
   ```

4. **Use Snapshots Wisely**
   ```
   - Too many snapshots slow down VMs
   - Delete old snapshots you don't need
   - Right-click snapshot → Delete
   ```

5. **Defragment VDI Files (Occasionally)**
   ```bash
   # Shutdown VM first
   VBoxManage modifymedium disk "path/to/disk.vdi" --compact
   ```

6. **Monitor Host Resources**
   ```
   Windows: Task Manager (Ctrl+Shift+Esc)
   Linux: htop or top
   macOS: Activity Monitor
   
   Check for:
   - CPU usage (should not be constantly at 100%)
   - RAM usage (some free RAM should be available)
   - Disk usage (should not be at 100% constantly)
   ```

---

## Lab-Specific Issues

### Issue: "Can't exploit Metasploitable"

**Symptoms:**
- Exploits fail
- Meterpreter sessions die immediately
- Connection refused

**Solutions:**

1. **Verify Metasploitable is Running**
   ```bash
   # From Kali
   nmap -sV 192.168.1.50
   # Should show many open ports
   ```

2. **Check Services on Metasploitable**
   ```bash
   # Login to Metasploitable
   # Username: msfadmin, Password: msfadmin
   
   # Check services
   sudo service --status-all
   ```

3. **Verify Network Connectivity**
   ```bash
   # From Kali
   ping 192.168.1.50
   
   # From Metasploitable
   ping 192.168.56.10
   ```

4. **Use Correct LHOST**
   ```bash
   # In Metasploit
   # LHOST must be IP that target can reach
   
   # If Metasploitable is on Internal Network (192.168.1.0/24)
   # And Kali is on Host-only (192.168.56.0/24)
   # They can't communicate directly
   
   # Solution: Put Kali on Internal Network too
   # Or add second adapter to Kali
   ```

5. **Try Different Exploits**
   ```bash
   # Some exploits are more reliable than others
   # Try these on Metasploitable 2:
   
   use exploit/unix/ftp/vsftpd_234_backdoor
   use exploit/multi/samba/usermap_script
   use auxiliary/scanner/smb/smb_version
   ```

### Issue: "Burp Suite certificate errors"

**Symptoms:**
- HTTPS sites show security warnings
- Can't intercept HTTPS traffic

**Solutions:**

1. **Export Burp Certificate**
   ```
   Burp Suite:
   - Proxy → Options
   - Proxy Listeners → Import / export CA certificate
   - Export in DER format
   - Save as burp-cert.der
   ```

2. **Import in Firefox**
   ```
   Firefox:
   - Settings → Privacy & Security → Certificates
   - View Certificates → Authorities
   - Import → Select burp-cert.der
   - Trust for websites → OK
   ```

3. **Import in Chrome/Edge**
   ```
   Windows:
   - Double-click burp-cert.der
   - Install Certificate
   - Local Machine → Next
   - Place in: Trusted Root Certification Authorities
   - Finish
   
   Linux:
   sudo cp burp-cert.der /usr/local/share/ca-certificates/burp-cert.crt
   sudo update-ca-certificates
   ```

---

## Getting Help

### Information to Gather

When asking for help, provide:

1. **System Information**
   ```
   - Host OS and version
   - VirtualBox version
   - VM configuration (RAM, CPU, network)
   - Guest OS and version
   ```

2. **Error Messages**
   ```
   - Exact error text
   - Screenshots of errors
   - VirtualBox log files
   ```

3. **What You've Tried**
   ```
   - Steps to reproduce
   - Solutions already attempted
   - Any changes made before problem started
   ```

4. **Log Files**
   ```
   VirtualBox logs:
   - VM → Show Log
   
   Linux logs:
   - /var/log/syslog
   - /var/log/messages
   - journalctl -xe
   
   Windows Event Viewer:
   - eventvwr.msc
   - Windows Logs → System
   - Windows Logs → Application
   ```

### Resources for Help

- **VirtualBox Forums**: https://forums.virtualbox.org/
- **Kali Linux Forums**: https://forums.kali.org/
- **Reddit**: r/virtualbox, r/Kali4noobs, r/AskNetsec
- **Stack Exchange**: Unix & Linux, Super User, Information Security
- **GitHub Issues**: For specific tool problems

---

## Prevention Tips

### Before Making Changes

1. **Take Snapshots**
   ```
   Before any major configuration:
   - Right-click VM → Snapshots
   - Take snapshot
   - Name it appropriately
   - Can restore if something breaks
   ```

2. **Document Current State**
   ```
   - Note current configuration
   - Save working configs to files
   - Screenshot working settings
   ```

3. **Test in Stages**
   ```
   - Make one change at a time
   - Test after each change
   - Easier to identify what broke
   ```

### Regular Maintenance

1. **Keep Systems Updated**
   ```bash
   # Kali Linux
   sudo apt update && sudo apt full-upgrade -y
   
   # Windows
   # Settings → Windows Update → Check for updates
   ```

2. **Update VirtualBox**
   ```
   - Check for updates monthly
   - Update Extension Pack too
   - Read release notes for changes
   ```

3. **Clean Up**
   ```
   - Delete old snapshots
   - Remove unused VMs
   - Clean up ISO files
   - Free up disk space
   ```

4. **Backup Configurations**
   ```
   - Export working VMs (File → Export Appliance)
   - Save pfSense config (Diagnostics → Backup/Restore)
   - Document custom configurations
   - Keep backups on external drive
   ```

---

**Troubleshooting Guide Version**: 1.0  
**Last Updated**: October 2025  

For setup instructions, see [SETUP_GUIDE.md](SETUP_GUIDE.md)
