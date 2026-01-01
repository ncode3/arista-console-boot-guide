# Booting an Arista DCS-7020TR-48 Switch: A Deep Dive into Enterprise Network Hardware

**Author:** Nolan | Atlanta AI & Robotics Initiative  
**Date:** December 31, 2024  
**Tags:** #Infrastructure #Networking #Linux #RHCSA #DataCenter

---

## Introduction

Most people think network switches are black boxes that "just work." Today, I'm pulling back the curtain to show you exactly what happens when enterprise network hardware boots up - and why understanding this process is critical for anyone serious about infrastructure.

This isn't just theory. This is me, console cable in hand, watching a $5,000+ Arista DCS-7020TR-48 switch bootstrap itself from bare metal using the same Linux kernel and tools you'd use to manage any server.

**What You'll Learn:**
- How to establish serial console access to enterprise network equipment
- The complete boot sequence of Arista EOS (Extensible Operating System)
- Why network switches are really just specialized Linux computers
- A repeatable process for initial switch configuration
- How this connects to broader infrastructure skills (RHCSA, OpenShift, automation)

---

## Hardware Overview

**Device:** Arista DCS-7020TR-48  
**Specs:**
- 48x 10GBASE-T ports
- 4x 40GbE QSFP+ uplinks
- 3.4GB flash storage
- Linux-based EOS operating system
- Built-in virtualization layer (QEMU)

**Use Case:** Physical data center switching for AARI's (Atlanta AI & Robotics Initiative) hands-on infrastructure lab, supporting OpenShift clusters, penetration testing environments, and HBCU student training.

---

## Prerequisites

Before you begin, you'll need:

### Hardware
1. **Console cable**: DB9-to-RJ45 serial console cable (or USB-to-RJ45 direct)
2. **USB-to-Serial adapter** (if using DB9 cable with modern laptop)
3. **Laptop** running Windows, Mac, or Linux

### Software
- **Windows**: PuTTY, Tera Term, or SecureCRT
- **Mac/Linux**: Screen, Minicom, or any serial terminal

### Network Knowledge
- Basic understanding of networking concepts (IP addressing, VLANs)
- Familiarity with Linux command line (helpful but not required)

---

## Step 1: Physical Connection

### Connecting the Console Cable

1. **Locate the console port** on the Arista switch (usually labeled "CONSOLE" - it's an RJ45 port, typically on the front or side)

2. **Connect your cable**:
   - RJ45 end → Switch console port
   - DB9/USB end → Your laptop

3. **Identify the COM port** (Windows):
   ```
   Windows Key + X → Device Manager → Ports (COM & LPT)
   ```
   Look for "USB Serial Port (COM3)" or similar

4. **Note the COM port number** - you'll need this for your terminal software

### Terminal Software Configuration

**For PuTTY (Windows):**
```
Connection Type: Serial
Serial line: COM3 (or your COM port number)
Speed: 9600
```

**Advanced settings (same for all terminal software):**
- Baud rate: 9600
- Data bits: 8
- Parity: None
- Stop bits: 1
- Flow control: None

**For Screen (Mac/Linux):**
```bash
screen /dev/ttyUSB0 9600
```

---

## Step 2: Power On and Initial Boot Sequence

### What Happens When You Power On

When you connect power to the Arista switch, you'll immediately see:
- **Physical indicators**: Fans spin up (they're LOUD), LEDs illuminate
- **Console output**: Boot messages start flowing

![Boot Sequence Start](screenshot_1.png)
*The initial boot sequence showing kernel initialization and PCI device enumeration*

### Understanding the Boot Messages

**Stage 1: Hardware Initialization (0-30 seconds)**

```
Press Ctrl+C now to enter Aboot shell
Booting flash:/EOS-4.20.5.1M.swi
0.156380:starting new kernel
1.798897:Running dosfsck on /mnt/flash
2.283067:Mounting SWIX filesystems on /mnt/flash
```

**What's happening:**
- The bootloader (Aboot) loads the EOS software image
- Kernel starts and mounts the flash filesystem
- System performs filesystem checks (dosfsck)

**Stage 2: PCI Device Enumeration**

You might see errors like:
```
[80:67d[01]] intelope 0000:80:1b.0: cannot remap PCI memory region
```

**Don't panic.** These are known PCI memory mapping quirks in some Arista hardware. The switch will continue booting normally. This is actually a great learning opportunity - enterprise hardware often has these kinds of non-fatal warnings that would scare beginners but are actually documented and benign.

![PCI Errors](screenshot_2.png)
*PCI memory region warnings - non-fatal and expected on some Arista models*

---

## Step 3: Linux System Initialization

### EOS is Linux

Here's where it gets interesting. Arista EOS isn't some proprietary black box - it's a hardened Linux distribution built on top of a standard Linux kernel.

![System Initialization](screenshot_3.png)
*User account initialization showing the Linux foundation of Arista EOS*

**You'll see familiar Linux components:**

```
systemd-coredump:x:990:990:systemd Core Dumper:/:/sbin/nologin
dbusx:x:81:81:System message bus:/:/sbin/nologin
ntpx:x:38:38::/etc/ntp:/sbin/nologin
sshd:x:74:74:Privilege-separated SSH:/var/empty/sshd:/sbin/nologin
rpcbind:x:32:32:Rpcbind Daemon:/var/lib/rpcbind:/sbin/nologin
redis:x:997:993:Redis Database Server:/var/lib/redis:/sbin/nologin
archive:x:997:997:archive user:/:/sbin/nologin
cvpadmin:x:72:72::/:/sbin/nologin
sessionuser:x:90:90:Internal Session User:/home/sessionuser:/usr/bin/SessionCli
ansible:x:1000:1001::/persist/local/ansible:/bin/bash
```

**Key observations:**
- **systemd**: Modern init system (RHCSA exam topic!)
- **sshd**: You'll be able to SSH to this switch once configured
- **redis**: Used for state management
- **ansible**: Built-in automation support (we'll use this later)

This is a full Linux environment with services you'd see on any enterprise server.

---

## Step 4: Virtualization Layer Initialization

### QEMU and Containerization

![QEMU Initialization](screenshot_4.png)
*QEMU processes starting - the switch runs its control plane in virtualized environments*

Modern network switches separate the **control plane** (management, routing protocols, configuration) from the **data plane** (actual packet switching hardware).

**You'll see QEMU processes:**
```
qemu-ga          qemu-io
qemu-img         qemu-nbd
qemu-pr-helper   qemu-system-x86_64
```

**Why this matters:**
- **Isolation**: Management software can't crash the switching fabric
- **Flexibility**: You can run custom applications alongside the switch OS
- **Resilience**: Control plane failures don't take down data forwarding

**Connection to your learning:**
- RHCSA covers KVM/QEMU virtualization
- OpenShift runs on similar virtualization stacks
- Understanding this architecture helps you see infrastructure as a unified system

---

## Step 5: System Restart and Watchdog Initialization

![Watchdog and Restart](screenshot_5.png)
*System restart sequence and watchdog processes*

```
Restarting system.
[19:30:55] watchdog_punch .
[19:30:55] watchdog_punch .
[19:30:57] watchdog_punch .
[19:30:58] watchdog_punch .
dpp_punch
[19:31:00] watchdog_punch .
```

**Watchdog processes** monitor system health:
- Automatically restart failed services
- Detect and recover from deadlocks
- Ensure high availability

This is enterprise-grade reliability engineering built into the hardware.

---

## Step 6: Boot Completion and Login

![Boot Complete](screenshot_6.png)
*Boot sequence complete - ready for login*

After 1-2 minutes, you'll see:

```
Welcome to Arista Networks EOS 4.20.5.1M
Installing EOS extensions
Error installing Aboot-patch-419257.1686.rpm: Installation failed: [Errno 2] No such file or directory: ''
/mnt/flash/rc.eos detected
```

Some extension installation errors are normal on first boot. The system continues normally.

### System Information Display

```
Model: DCS-7050TX-64
Serial Number: JPE14371630
System RAM: 3082512 kB
Flash Memory Size: 3.4G
```

### Login Prompts

You'll cycle through several login attempts as the system fully initializes:

```
localhost login: tupacibob
Password:
Login incorrect

localhost login: admin
Password:
Login incorrect
```

**Default Arista Credentials:**
- Username: `admin`
- Password: (blank - just press Enter)

---

## Step 7: First Login Success

Once the system fully stabilizes (give it another 30 seconds after first login prompt), try again:

```
localhost login: admin
Password: [press Enter]
localhost:~$
```

**You're in.**

### Initial Shell Environment

```
[admin@localhost ~]$ 
```

You're now in a bash shell with admin privileges. This is a full Linux environment.

**Try basic commands:**

```bash
# Check Linux kernel version
uname -a

# View running processes
ps aux

# Check system uptime
uptime

# View network interfaces (before configuration)
ip link show

# Enter privileged mode
sudo -i
```

---

## Step 8: Entering the Arista CLI

While the Linux shell is powerful, Arista provides a network-focused CLI:

```bash
# From the bash shell, enter the Arista CLI
cli

# You'll see:
Arista Networks EOS shell

# Enter privileged mode
enable

# Enter configuration mode
configure terminal
```

Now you're ready to configure the switch.

---

## Understanding What Just Happened

### The Boot Process Summary

1. **Hardware POST** (Power-On Self-Test) - Hardware initialization
2. **Bootloader (Aboot)** - Loads the EOS image from flash
3. **Linux Kernel Boot** - Standard Linux kernel initialization
4. **Filesystem Mount** - Flash storage mounted and checked
5. **Systemd Initialization** - Services start (SSH, Redis, etc.)
6. **Virtualization Layer** - QEMU processes for control plane isolation
7. **Network Services** - EOS-specific networking daemons
8. **Login Ready** - System fully operational

**Total boot time:** ~90-120 seconds from power-on to login

### Key Takeaways

**1. Network Switches Are Linux Computers**
Every command, every concept from RHCSA applies here. Understanding Linux makes you a better network engineer.

**2. Enterprise Hardware Is Transparent**
Unlike consumer gear, enterprise equipment shows you exactly what's happening. Those boot messages aren't errors - they're documentation.

**3. Modern Infrastructure Is Virtualized**
Even physical switches use virtualization for isolation and flexibility.

**4. Automation Is Built-In**
Notice that `ansible` user? Arista expects you to automate. Manual configuration is for bootstrapping only.

---

## Next Steps: Configuration Roadmap

Now that the switch is booted and accessible, here's the typical configuration path:

### Phase 1: Basic Connectivity
1. Set hostname
2. Configure management interface IP
3. Enable SSH access
4. Set timezone and NTP

### Phase 2: Network Configuration
1. Create VLANs for network segmentation
2. Configure trunk ports for inter-switch connectivity
3. Configure access ports for end devices
4. Set up spanning tree

### Phase 3: Advanced Features
1. Configure routing (if doing Layer 3)
2. Implement security policies
3. Set up monitoring (SNMP, syslog)
4. Configure high availability features

### Phase 4: Automation
1. Create Ansible inventory
2. Write playbooks for configuration
3. Version control in Git
4. Implement CI/CD for network changes

---

## Real-World Application: AARI Data Center

This switch will serve as the core networking infrastructure for the Atlanta AI & Robotics Initiative's physical data center:

### Network Architecture
```
[Internet]
    │
[Firewall/Router]
    │
[Arista DCS-7020TR-48] ← This switch
    │
    ├── VLAN 100: OpenShift Management Network
    ├── VLAN 200: OpenShift Data Network
    ├── VLAN 300: Penetration Testing Lab (Isolated)
    ├── VLAN 400: Student Development Environment
    └── VLAN 999: Out-of-Band Management
```

### Integration Points
- **OpenShift Cluster**: Dave's 3-node cluster connects here
- **Security Lab**: Isolated VLAN for "Build a Data Center, Then Hack It"
- **Student Access**: Safe, segmented environment for HBCU students
- **Management**: Separate OOB network for infrastructure access

---

## Why This Matters for Your Career

### Skills Demonstrated
✅ **Hardware expertise**: Physical infrastructure setup  
✅ **Linux systems**: Understanding boot processes, systemd, virtualization  
✅ **Networking**: Enterprise switching, VLANs, routing  
✅ **Documentation**: Creating repeatable processes  
✅ **Troubleshooting**: Reading boot logs, identifying errors  

### Certification Alignment
- **RHCSA**: Linux administration, systemd, networking
- **CCNA**: Network fundamentals, switch configuration
- **Security+**: Network segmentation, infrastructure security

### Enterprise Value
Companies need people who understand the full stack - from bare metal to application. This type of hands-on infrastructure work is rare and valuable.

---

## Troubleshooting Guide

### Common Issues and Solutions

**Issue: No console output when switch powers on**
- Check cable connections (both ends)
- Verify correct COM port in terminal software
- Try different baud rates (9600, 115200)
- Test with different terminal software

**Issue: Garbled text on console**
- Wrong baud rate (must be 9600 for Arista)
- Flow control settings incorrect (should be None)
- Cable quality issue (try different cable)

**Issue: Cannot login with admin/blank password**
- Switch may have been previously configured
- Try default password "admin"
- May need factory reset (requires physical access to reset button)

**Issue: System keeps rebooting**
- Corrupted EOS image - reflash from USB
- Hardware failure - check fan operation, temperature
- Power supply issue - verify stable power

**Issue: PCI memory errors prevent boot**
- These are usually non-fatal warnings
- If boot actually fails, may need firmware update
- Contact Arista support with serial number

---

## Resources and Further Reading

### Official Documentation
- [Arista EOS Documentation](https://www.arista.com/en/support/software-download)
- [Arista Configuration Guides](https://www.arista.com/en/support/product-documentation)

### Related Skills
- NetworkChuck's CCNA course (free on YouTube)
- Red Hat RHCSA official training
- Ansible for Network Automation (Arista has excellent docs)

### Community
- Arista Networks Community Forums
- Reddit: r/networking, r/homelab
- AARI Discord (for infrastructure learners)

---

## Conclusion

What looks like magic is actually engineering. That Arista switch isn't some mysterious appliance - it's a purpose-built Linux server with specialized hardware for packet switching.

By understanding the boot process, you've gained:
1. Confidence working with enterprise hardware
2. Deeper Linux systems knowledge
3. Foundation for network automation
4. Troubleshooting skills that transfer across infrastructure domains

**Most importantly:** You've proven that infrastructure isn't beyond your reach. It's just computers, all the way down.

The next step? Configure this switch, integrate it with your OpenShift cluster, segment your network properly, then systematically attack it to find weaknesses.

Build it. Break it. Learn.

---

## About the Author

Nolan is the founder and Executive Director of the Atlanta AI & Robotics Initiative (AARI), a 501(c)(3) nonprofit focused on hands-on AI infrastructure and robotics education for HBCU students, veterans, and professionals. With 8+ years of Data/AI experience as a Microsoft Solution Architect, he's pursuing RHCSA certification while building enterprise-grade infrastructure for educational purposes.

This documentation is part of the "Build a Data Center, Then Hack It" project - a comprehensive approach to learning infrastructure security through hands-on construction and penetration testing.

**Connect:**
- LinkedIn: [Your LinkedIn]
- AARI Website: [AARI site]
- GitHub: [Your repos]

---

## Appendix A: Quick Reference Commands

### Serial Console Connection
```bash
# Windows (PuTTY)
Serial line: COM3
Speed: 9600

# Mac/Linux (Screen)
screen /dev/ttyUSB0 9600

# Exit screen session
Ctrl+A, then K, then Y
```

### Initial Login
```bash
Username: admin
Password: [blank - press Enter]
```

### Basic Navigation
```bash
# Enter CLI from bash
cli

# Enter privileged mode
enable

# Enter configuration mode
configure terminal

# Save configuration
write memory

# Exit configuration mode
exit

# Return to bash
bash
```

### System Information
```bash
# Show version
show version

# Show running config
show running-config

# Show interfaces
show interfaces status

# Show VLANs
show vlan

# Show system resources
show processes top
```

---

## Appendix B: Network Diagram Template

```
┌─────────────────────────────────────────────────────────┐
│                    AARI Data Center                      │
│                Physical Network Layout                    │
└─────────────────────────────────────────────────────────┘

                        [Internet]
                            │
                     [Edge Router]
                            │
              ┌─────────────┴─────────────┐
              │                           │
        [Firewall/UTM]              [Out-of-Band]
              │                      [Management]
              │                           │
    ┌─────────┴──────────┐               │
    │                    │               │
[Core Switch]      [Management]──────────┘
Arista DCS-7020TR-48    Switch
    │
    ├── VLAN 100: OpenShift Management
    ├── VLAN 200: OpenShift Data  
    ├── VLAN 300: Pentest Lab (Isolated)
    ├── VLAN 400: Student Development
    └── VLAN 999: Infrastructure OOB
```

---

**End of Documentation**

*Last updated: December 31, 2024*  
*Version: 1.0*  
*License: CC BY-SA 4.0*
