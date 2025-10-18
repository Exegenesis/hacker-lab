# Cybersecurity Home Lab

**A comprehensive guide to building and configuring a complete cybersecurity home lab for ethical hacking training and penetration testing practice.**

## 🎯 Overview

This repository contains detailed, step-by-step instructions for setting up a professional-grade cybersecurity home lab environment. Perfect for:
- **Aspiring Ethical Hackers** preparing for certifications (CEH, OSCP, Security+)
- **Security Professionals** practicing penetration testing techniques
- **Students** learning cybersecurity fundamentals
- **IT Professionals** developing security skills

## 🛠️ What's Included

### Virtual Machines
- **pfSense** - Firewall and network gateway
- **Kali Linux** - Primary penetration testing platform
- **Windows 10/11** - Client systems for testing
- **Windows Server** - Active Directory domain controller
- **Metasploitable** - Intentionally vulnerable target

### Tools Covered
#### Network & Scanning
- Nmap - Network discovery and port scanning
- Wireshark - Packet capture and analysis

#### Penetration Testing
- Metasploit Framework - Exploitation framework
- Burp Suite - Web application testing
- OWASP ZAP - Web security scanner
- OpenVAS - Vulnerability scanner

#### Development & Scripting
- Python - Scripting and automation
- Visual Studio Code - Code editor and IDE

#### Documentation
- Obsidian - Note-taking and knowledge management
- Flameshot - Screenshot and annotation tool

## 📚 Documentation

### [→ Complete Setup Guide](SETUP_GUIDE.md)

The comprehensive guide includes:
1. **Prerequisites** - Hardware and knowledge requirements
2. **Foundation Setup** - VirtualBox installation and configuration
3. **Virtual Machines** - Step-by-step VM creation and setup
4. **Tools Installation** - Installing and configuring all required tools
5. **Network Configuration** - Setting up network topology and connectivity
6. **Documentation Setup** - Creating a professional documentation system
7. **Testing & Validation** - Practice exercises and labs
8. **Troubleshooting** - Common issues and solutions

## 🏗️ Lab Architecture

```
                    Internet
                        |
                 [Host Machine]
                        |
       +----------------+----------------+
       |                |                |
  [NAT Network]   [Internal Net]  [Host-Only]
       |                |                |
  +----------+     +----------+     +----------+
  | pfSense  |     | Windows  |     |  Kali    |
  | Firewall |     | Server   |     |  Linux   |
  +----------+     | (AD/DC)  |     +----------+
       |           +----------+          |
       |                |                |
  +----------+     +----------+     +----------+
  | Windows  |     | Domain   |     | Meta-    |
  | 10/11    |     | Clients  |     |sploitable|
  | Client   |     +----------+     +----------+
  +----------+
```

## 💻 System Requirements

### Minimum Specifications
- **CPU**: Intel i5/AMD Ryzen 5 (with VT-x/AMD-V)
- **RAM**: 16 GB
- **Storage**: 250 GB free SSD space
- **Network**: Ethernet adapter

### Recommended Specifications
- **CPU**: Intel i7/AMD Ryzen 7 (8+ cores)
- **RAM**: 32-64 GB
- **Storage**: 500 GB - 1 TB SSD
- **Network**: Gigabit Ethernet

## 🚀 Quick Start

1. **Clone this repository**
   ```bash
   git clone https://github.com/Exegenesis/hacker-lab.git
   cd hacker-lab
   ```

2. **Read the complete setup guide**
   ```bash
   # Open SETUP_GUIDE.md in your preferred markdown viewer
   ```

3. **Follow the step-by-step instructions**
   - Start with Phase 1: Foundation Setup
   - Proceed through each phase systematically
   - Take snapshots before major changes
   - Document your progress

4. **Practice and learn**
   - Complete the included exercises
   - Experiment safely in your isolated lab
   - Document your findings

## 📋 What You'll Learn

- Virtual machine setup and management
- Network design and configuration
- Firewall configuration and rule creation
- Active Directory deployment and management
- Network scanning and reconnaissance
- Vulnerability assessment and scanning
- Web application security testing
- Password cracking techniques
- Exploitation with Metasploit
- Traffic analysis with Wireshark
- Professional documentation practices

## ⚠️ Important Notes

### Security and Legal
- ✅ **This lab is for educational purposes only**
- ✅ **Practice only on systems you own or have permission to test**
- ✅ **Never attack systems without explicit authorization**
- ✅ **Keep vulnerable VMs isolated from the internet**
- ✅ **Follow responsible disclosure for any real vulnerabilities found**

### Best Practices
- Take VM snapshots before major changes
- Document everything using the provided templates
- Keep non-vulnerable systems updated
- Use evaluation licenses legally
- Back up your lab configurations regularly

## 🎓 Recommended Learning Path

1. **Foundation** (Week 1-2)
   - Set up the complete lab environment
   - Verify all VMs and tools are working
   - Complete network connectivity tests

2. **Basics** (Week 3-4)
   - Network discovery with Nmap
   - Packet capture with Wireshark
   - Basic vulnerability scanning

3. **Intermediate** (Week 5-8)
   - Web application testing with Burp Suite
   - Active Directory enumeration
   - Exploitation with Metasploit

4. **Advanced** (Week 9-12)
   - Custom exploit development
   - Advanced AD attacks
   - Full penetration testing exercises

## 🤝 Contributing

Contributions are welcome! If you have improvements, additional exercises, or corrections:
1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Submit a pull request

## 📞 Support

- **Issues**: Use the GitHub Issues tab for questions or problems
- **Discussions**: Share your lab setups and experiences
- **Updates**: Watch the repository for updates and improvements

## 📖 Additional Resources

### Learning Platforms
- [TryHackMe](https://tryhackme.com) - Interactive cybersecurity training
- [Hack The Box](https://www.hackthebox.com) - Penetration testing labs
- [OverTheWire](https://overthewire.org) - Wargames for learning
- [PentesterLab](https://pentesterlab.com) - Web penetration testing exercises

### Certifications
- **CEH** - Certified Ethical Hacker
- **OSCP** - Offensive Security Certified Professional
- **Security+** - CompTIA Security+
- **GPEN** - GIAC Penetration Tester

### Communities
- Reddit: [r/netsec](https://reddit.com/r/netsec), [r/AskNetsec](https://reddit.com/r/AskNetsec)
- Discord: NetSecFocus, The Many Hats Club
- Forums: Kali Linux Forums, Metasploit Community

## 📄 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## 🙏 Acknowledgments

- Offensive Security for Kali Linux
- Metasploit team at Rapid7
- pfSense project
- All open-source security tool developers
- The cybersecurity community

---

**Disclaimer**: This guide is for educational purposes only. Always obtain proper authorization before testing any systems. Unauthorized access to computer systems is illegal and punishable by law.

---

**Version**: 1.0  
**Last Updated**: October 2025  
**Maintained by**: Exegenesis
