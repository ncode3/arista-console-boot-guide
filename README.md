# Arista Switch Console Boot Guide

> **A comprehensive, hands-on guide to booting and configuring Arista network switches from bare metal**

![License](https://img.shields.io/badge/license-MIT-blue.svg)
![Arista EOS](https://img.shields.io/badge/Arista_EOS-4.20.5.1M-orange.svg)
![Linux](https://img.shields.io/badge/Linux-Kernel_4.19-yellow.svg)
![Documentation](https://img.shields.io/badge/docs-complete-green.svg)

## 🎯 Overview

This repository documents the complete process of establishing console access to an Arista DCS-7020TR-48 enterprise network switch, understanding its boot sequence, and performing initial configuration. 

**What makes this different:** This isn't just a config guide. This is a deep dive into how enterprise network hardware actually works - from PCI device enumeration to Linux kernel initialization to the virtualization layer that separates control and data planes.

**Who this is for:**
- Infrastructure engineers studying for RHCSA, CCNA, or Security+
- Network engineers who want to understand the Linux foundation of modern switches
- Anyone building physical data center infrastructure
- Students and educators in infrastructure/networking programs
- DevOps/SRE professionals working on network automation

## 📋 Table of Contents

- [Hardware Overview](#hardware-overview)
- [Prerequisites](#prerequisites)
- [Quick Start](#quick-start)
- [Complete Documentation](#complete-documentation)
- [Boot Sequence Analysis](#boot-sequence-analysis)
- [Configuration Examples](#configuration-examples)
- [Automation with Ansible](#automation-with-ansible)
- [Troubleshooting](#troubleshooting)
- [Contributing](#contributing)
- [License](#license)
- [About AARI](#about-aari)

## 🖥️ Hardware Overview

**Device:** Arista DCS-7020TR-48

**Specifications:**
- 48x 10GBASE-T RJ45 ports
- 4x 40GbE QSFP+ uplink ports
- 3.4GB flash storage
- 3GB+ system RAM
- Arista EOS 4.20.5.1M (Linux-based)
- Built-in QEMU virtualization layer

**Use Case:** Core switching for the Atlanta AI & Robotics Initiative (AARI) data center, supporting OpenShift clusters, security labs, and hands-on infrastructure training.

## 🔧 Prerequisites

### Hardware Required

```
┌─────────────────────────────────────────┐
│  Your Laptop                            │
│  └─ USB-to-Serial Adapter               │
│     └─ DB9-to-RJ45 Console Cable        │
│        └─ Arista Switch Console Port    │
└─────────────────────────────────────────┘
```

- **Console cable**: DB9-to-RJ45 serial console cable OR USB-to-RJ45 direct console cable
- **USB-to-Serial adapter**: If using DB9 cable (not needed for USB-to-RJ45)
- **Laptop**: Windows, macOS, or Linux

### Software Required

**Terminal Software:**
- **Windows**: PuTTY, Tera Term, SecureCRT
- **macOS/Linux**: screen, minicom, or cu

**Optional (for automation):**
- Ansible 2.9+
- Python 3.8+
- Git

### Knowledge Prerequisites

- Basic Linux command line navigation
- Understanding of networking fundamentals (IP addressing, VLANs)
- Familiarity with terminal/shell environments (helpful but not required)

## 🚀 Quick Start

### 1. Connect Console Cable

```bash
# Identify your serial port
# Windows: Check Device Manager → Ports (COM & LPT)
# Mac: ls /dev/tty.usbserial*
# Linux: ls /dev/ttyUSB*
```

### 2. Configure Terminal Software

**Settings:**
- Baud rate: `9600`
- Data bits: `8`
- Parity: `None`
- Stop bits: `1`
- Flow control: `None`

**Connect:**

```bash
# Windows (PuTTY)
Connection Type: Serial
Serial line: COM3
Speed: 9600

# Mac/Linux (screen)
screen /dev/ttyUSB0 9600

# Linux (minicom)
minicom -D /dev/ttyUSB0 -b 9600
```

### 3. Power On and Watch Boot

Boot time: ~90-120 seconds

### 4. Login

```bash
localhost login: admin
Password: [press Enter - default is blank]

# You're now at the Linux bash shell
[admin@localhost ~]$

# Enter Arista CLI
cli

# Enter privileged mode
enable

# Enter configuration mode
configure terminal
```

### 5. Basic Configuration

```bash
# Set hostname
hostname aari-sw01

# Configure management interface
interface Management1
   ip address 192.168.1.10/24
   no shutdown

# Set default gateway
ip route 0.0.0.0/0 192.168.1.1

# Enable SSH
management ssh
   idle-timeout 30

# Create admin user with password
username admin privilege 15 secret <your-password>

# Save configuration
write memory
```

**Done.** You now have a functional, SSH-accessible switch.

## 📚 Complete Documentation

### Core Documentation

| Document | Description |
|----------|-------------|
| [Full Boot Guide](docs/arista_console_boot_guide.md) | Complete walkthrough with screenshots |
| [Quick Reference](docs/QUICK_REFERENCE.md) | Command cheat sheet |
| [Network Configuration](docs/NETWORK_CONFIG.md) | VLAN, trunking, and Layer 3 setup |
| [Security Hardening](docs/SECURITY.md) | Security best practices |
| [Ansible Automation](docs/ANSIBLE.md) | Network automation playbooks |
| [Troubleshooting](docs/TROUBLESHOOTING.md) | Common issues and solutions |

### Screenshots

All boot sequence screenshots are in [`screenshots/`](screenshots/) directory with detailed annotations.

### Configuration Examples

Pre-built configuration templates in [`configs/`](configs/):

- `base_config.cfg` - Minimal working configuration
- `datacenter_config.cfg` - Full data center setup
- `lab_config.cfg` - Lab/testing environment
- `security_hardened.cfg` - Production-ready secure config

### Ansible Playbooks

Automation playbooks in [`ansible/`](ansible/):

- `initial_setup.yml` - Bootstrap configuration
- `vlan_config.yml` - VLAN management
- `security_hardening.yml` - Security automation

## 🔍 Boot Sequence Analysis

### Boot Stages

```
┌─────────────────────────────────────────────────────┐
│ Stage 1: Hardware POST (0-5s)                       │
├─────────────────────────────────────────────────────┤
│ Stage 2: Bootloader - Aboot (5-15s)                 │
├─────────────────────────────────────────────────────┤
│ Stage 3: Linux Kernel Boot (15-30s)                 │
├─────────────────────────────────────────────────────┤
│ Stage 4: System Services - systemd (30-60s)         │
├─────────────────────────────────────────────────────┤
│ Stage 5: Virtualization Layer - QEMU (60-90s)       │
├─────────────────────────────────────────────────────┤
│ Stage 6: EOS Initialization (90-120s)               │
└─────────────────────────────────────────────────────┘
```

**Key Insight:** Arista switches are Linux computers with specialized ASICs. All your RHCSA/Linux skills apply to network infrastructure.

See [docs/BOOT_SEQUENCE.md](docs/BOOT_SEQUENCE.md) for detailed analysis.

## 📝 Configuration Examples

### Basic Management

```bash
configure terminal
hostname aari-sw01
interface Management1
   ip address 192.168.1.10/24
   no shutdown
ip route 0.0.0.0/0 192.168.1.1
write memory
```

### VLAN Configuration

```bash
vlan 100
   name MGMT
vlan 200
   name DATA

interface Ethernet1
   switchport mode trunk
   switchport trunk allowed vlan 100,200
```

More examples in [`configs/`](configs/) directory.

## 🤖 Automation with Ansible

### Quick Start

```bash
# Install Arista collection
ansible-galaxy collection install arista.eos

# Run initial setup playbook
ansible-playbook -i inventory/hosts.yml playbooks/initial_setup.yml
```

### Example Playbook

```yaml
- name: Configure Arista switch
  hosts: arista_switches
  tasks:
    - name: Set hostname
      arista.eos.eos_system:
        hostname: "{{ inventory_hostname }}"
    
    - name: Create VLANs
      arista.eos.eos_vlans:
        config:
          - vlan_id: 100
            name: "MGMT"
```

Full automation guide: [docs/ANSIBLE.md](docs/ANSIBLE.md)

## 🐛 Troubleshooting

### Common Issues

**No Console Output**
- Verify cable connections
- Check COM port number
- Confirm baud rate is 9600

**Cannot Login**
- Default: username `admin`, password blank (press Enter)
- Try password "admin" if previously configured

**PCI Errors**
- Usually non-fatal warnings
- Wait 2-3 minutes for full boot

See [docs/TROUBLESHOOTING.md](docs/TROUBLESHOOTING.md) for complete guide.

## 🤝 Contributing

Contributions welcome! See [CONTRIBUTING.md](CONTRIBUTING.md) for guidelines.

**Ideas:**
- Additional configuration examples
- Ansible playbooks for specific use cases
- Integration guides (Kubernetes, OpenShift, etc.)
- Translations

## 📄 License

MIT License - see [LICENSE](LICENSE) for details.

Use freely. Attribution appreciated but not required.

## 🎓 About AARI

This repository is maintained by the **Atlanta AI & Robotics Initiative (AARI)**, a 501(c)(3) nonprofit focused on hands-on AI infrastructure and robotics education for HBCU students, veterans, and professionals.

**Mission:** Build infrastructure sovereignty through education.

**Approach:** "Build a Data Center, Then Hack It"

1. Build enterprise infrastructure from bare metal
2. Configure with production-grade automation
3. Implement security hardening
4. Systematically test for weaknesses
5. Document everything openly

### Connect

- Website: [Coming Soon]
- GitHub: [@aari-initiative](https://github.com/aari-initiative)
- LinkedIn: [AARI LinkedIn]

---

## 🙏 Acknowledgments

- **Dave** (Red Hat mentor) - Infrastructure guidance and OpenShift access
- **Arista Networks** - For Linux-based, open network platforms
- **NetworkChuck** - Certification path inspiration
- **Open-source community** - Ansible, docs, shared knowledge

---

**Built with 💪 in Atlanta, GA**

*Infrastructure isn't magic. It's just engineering you can learn, document, and teach.*
