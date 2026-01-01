# Quick Reference Guide

> Cheat sheet for Arista EOS console operations

## Serial Console Connection

### Terminal Settings
```
Baud Rate: 9600
Data Bits: 8
Parity: None
Stop Bits: 1
Flow Control: None
```

### Connection Commands

**Windows (PuTTY):**
```
Connection Type: Serial
Serial Line: COM3 (check Device Manager)
Speed: 9600
```

**Mac/Linux (Screen):**
```bash
screen /dev/ttyUSB0 9600

# Exit screen: Ctrl+A, then K, then Y
```

**Mac/Linux (Minicom):**
```bash
minicom -D /dev/ttyUSB0 -b 9600
```

---

## Default Credentials

```
Username: admin
Password: [blank - just press Enter]
```

---

## Basic Navigation

### Shell to CLI
```bash
# From bash shell, enter Arista CLI
cli
```

### CLI Modes
```bash
# User mode (default after login)
>

# Enter privileged mode
enable
#

# Enter configuration mode
configure terminal
(config)#

# Exit configuration mode
exit
#

# Return to bash shell
bash
$
```

---

## Essential Commands

### System Information
```bash
# Show version and system info
show version

# Show running configuration
show running-config

# Show startup configuration  
show startup-config

# Show hardware inventory
show inventory

# Show system resources
show processes top

# Show flash filesystem
dir flash:
```

### Interface Commands
```bash
# Show all interfaces
show interfaces

# Show interface status
show interfaces status

# Show specific interface details
show interfaces Ethernet1

# Show interface brief
show ip interface brief
```

### Configuration Management
```bash
# Save running config to startup
write memory
# or
copy running-config startup-config

# View differences
show running-config diffs

# Clear current configuration
write erase

# Reload switch
reload
```

---

## Initial Configuration Template

```bash
# Enter CLI
cli

# Enter privileged mode
enable

# Enter configuration mode
configure terminal

# Set hostname
hostname AARI-SW01

# Configure management interface
interface Management1
   ip address 192.168.1.10/24
   no shutdown
   exit

# Set default gateway
ip route 0.0.0.0/0 192.168.1.1

# Enable SSH
management api http-commands
   no shutdown
   exit

# Create admin user with password
username admin privilege 15 role network-admin secret MySecurePassword

# Set timezone
clock timezone America/New_York

# Configure NTP
ntp server 0.pool.ntp.org
ntp server 1.pool.ntp.org

# Set DNS servers
ip name-server 8.8.8.8
ip name-server 8.8.4.4

# Set domain name
ip domain-name aari.local

# Enable LLDP
lldp run

# Save configuration
write memory

# Verify
show running-config
```

---

## VLAN Configuration

### Create VLANs
```bash
configure terminal

# Create VLAN
vlan 10
   name MANAGEMENT
   exit

vlan 20
   name DATA
   exit

vlan 30
   name GUEST
   exit

# Show VLANs
show vlan
```

### Configure Access Port
```bash
# Assign port to VLAN
interface Ethernet1
   description Server-01
   switchport mode access
   switchport access vlan 10
   spanning-tree portfast
   no shutdown
   exit
```

### Configure Trunk Port
```bash
# Configure trunk port
interface Ethernet48
   description Uplink-to-Core
   switchport mode trunk
   switchport trunk allowed vlan 10,20,30
   no shutdown
   exit
```

---

## Layer 3 Configuration

### Configure SVI (Switched Virtual Interface)
```bash
# Enable routing
ip routing

# Create VLAN interface
interface Vlan10
   description Management-Network
   ip address 10.10.10.1/24
   no shutdown
   exit

interface Vlan20
   description Data-Network  
   ip address 10.10.20.1/24
   no shutdown
   exit
```

### Static Routes
```bash
# Add static route
ip route 10.20.0.0/16 192.168.1.1

# Add default route
ip route 0.0.0.0/0 192.168.1.1

# Show routing table
show ip route
```

---

## Security Configuration

### SSH Configuration
```bash
# Generate SSH keys
security security configure
   ssh key rsa 4096
   exit

# Configure SSH
management ssh
   no shutdown
   exit

# Show SSH status
show management ssh
```

### Access Control Lists (ACLs)
```bash
# Create ACL
ip access-list MGMT-ACCESS
   10 permit tcp 192.168.1.0/24 any eq ssh
   20 permit tcp 192.168.1.0/24 any eq https
   30 deny ip any any log
   exit

# Apply to management interface
interface Management1
   ip access-group MGMT-ACCESS in
   exit
```

### User Management
```bash
# Create user with privilege 15 (admin)
username netadmin privilege 15 role network-admin secret SecurePass123

# Create read-only user
username readonly privilege 1 role network-operator secret ReadPass456

# Show users
show user-account
```

---

## Monitoring & Troubleshooting

### Interface Troubleshooting
```bash
# Show interface errors
show interfaces counters errors

# Show interface rates
show interfaces counters rates

# Clear interface counters
clear counters

# Show interface transceivers
show interfaces transceiver
```

### System Logs
```bash
# Show recent logs
show logging

# Show logs filtered by severity
show logging level errors

# Configure remote syslog
logging host 192.168.1.100

# Set logging level
logging level informational
```

### Connectivity Testing
```bash
# Ping from switch
ping 8.8.8.8

# Ping with source interface
ping 8.8.8.8 source Management1

# Traceroute
traceroute 8.8.8.8

# Test DNS
nslookup google.com
```

---

## Backup & Restore

### Backup Configuration
```bash
# Copy running config to TFTP server
copy running-config tftp://192.168.1.100/switch-config.txt

# Copy to USB (if available)
copy running-config usb:/config-backup.txt

# Copy startup config
copy startup-config tftp://192.168.1.100/startup-config.txt
```

### Restore Configuration
```bash
# Restore from TFTP
copy tftp://192.168.1.100/switch-config.txt running-config

# Restore from USB
copy usb:/config-backup.txt running-config
```

---

## Advanced Features

### MLAG (Multi-Chassis Link Aggregation)
```bash
# Configure MLAG domain
mlag configuration
   domain-id MLAG-DOMAIN-1
   local-interface Vlan4094
   peer-address 10.255.255.2
   peer-link Port-Channel10
   exit
```

### BGP Configuration
```bash
# Enable BGP
router bgp 65001
   router-id 10.10.10.1
   neighbor 10.10.20.1 remote-as 65002
   network 10.10.10.0/24
   exit
```

### EVPN/VXLAN
```bash
# Configure VXLAN interface
interface Vxlan1
   vxlan source-interface Loopback0
   vxlan udp-port 4789
   vxlan vlan 10 vni 10010
   exit
```

---

## Ansible Quick Start

### Install Arista Collection
```bash
ansible-galaxy collection install arista.eos
```

### Simple Playbook
```yaml
---
- name: Configure Arista Switch
  hosts: arista_switches
  gather_facts: no
  
  tasks:
    - name: Set hostname
      arista.eos.eos_system:
        hostname: AARI-SW01
    
    - name: Configure Management Interface
      arista.eos.eos_l3_interfaces:
        config:
          - name: Management1
            ipv4:
              - address: 192.168.1.10/24
    
    - name: Save configuration
      arista.eos.eos_config:
        save_when: modified
```

---

## Emergency Recovery

### Factory Reset
```bash
# WARNING: This erases all configuration

# Method 1: From CLI
write erase
reload

# Method 2: From Aboot (during boot)
# Press Ctrl+C during boot
# At Aboot# prompt:
zerotouch cancel
reboot
```

### Boot from USB
```bash
# During boot, press Ctrl+C
# At Aboot# prompt:
boot usb:/EOS.swi
```

### Password Recovery
```bash
# Boot to Aboot shell (Ctrl+C during boot)
# Mount flash
mount /dev/sda1 /mnt

# Edit startup config
vi /mnt/startup-config

# Remove username/password lines
# Reboot
reboot
```

---

## Useful Keyboard Shortcuts

### CLI Navigation
```
Tab             - Auto-complete
?               - Context-sensitive help
Up/Down Arrow   - Command history
Ctrl+A          - Beginning of line
Ctrl+E          - End of line
Ctrl+W          - Delete previous word
Ctrl+U          - Delete entire line
Ctrl+Z          - Exit to privileged mode
```

### Terminal (Screen)
```
Ctrl+A, D       - Detach session
Ctrl+A, K       - Kill session
Ctrl+A, ?       - Help
```

---

## Common Error Messages

### "Login incorrect"
- Check username (default: `admin`)
- Password might be set (try `admin` or blank)
- Wait for system to fully boot

### "Incomplete command"
- Use `?` to see available options
- Check syntax with `show running-config`

### "Invalid input"
- Command may not be available in current mode
- Check EOS version for feature support

### "% Ambiguous command"
- Multiple commands match - be more specific
- Use Tab completion

---

## Additional Resources

- Official Docs: https://www.arista.com/en/support/product-documentation
- EOS API: https://www.arista.com/en/um-eos/eos-section-42-1-programmability
- Ansible Collection: https://docs.ansible.com/ansible/latest/collections/arista/eos/

---

*This quick reference is part of the [Arista Console Boot Guide](README.md) project.*
