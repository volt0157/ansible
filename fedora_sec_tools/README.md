# Fedora Security Tools

This Ansible project automates the installation and configuration of essential security tools for Fedora and RHEL-based systems.

## Security Tools Included

### 1. Lynis
Open-source security auditing tool that performs:
- Security scans and audits
- System hardening checks
- Compliance testing
- Vulnerability detection

### 2. Firewall Tools
Firewall management tools for network security:
- **firewalld**: Dynamic firewall daemon with D-Bus interface
- **firewall-cmd**: Command-line interface for firewalld
- **firewall-config**: Graphical user interface for firewall management

### 3. ClamAV Antivirus Suite
Comprehensive antivirus solution including:
- **clamav**: Core antivirus engine
- **clamav-update**: Virus definition updater
- **clamtk**: GUI interface for ClamAV
- Automatic virus definition updates via clamav-freshclam service

## Project Structure

```
fedora_sec_tools/
├── site.yml                          # Main playbook
├── inventory.yml                     # Inventory file
├── roles/
│   ├── lynis_install/
│   │   └── tasks/
│   │       └── main.yml              # Lynis installation tasks
│   ├── firewall_tools/
│   │   └── tasks/
│   │       └── main.yml              # Firewall tools installation
│   └── clamav_install/
│       └── tasks/
│           └── main.yml              # ClamAV installation and configuration
└── README.md                         # This file
```

## Prerequisites

- Ansible 2.9 or higher
- Fedora, RHEL, CentOS, Rocky Linux, or AlmaLinux target systems
- Root or sudo privileges on target systems

## Usage

### Basic Installation

To install all security tools on localhost:

```bash
cd fedora_sec_tools
ansible-playbook -i inventory.yml site.yml
```

### Install with Lynis Security Audit

To install all tools and immediately run a Lynis security audit:

```bash
ansible-playbook -i inventory.yml site.yml -e "run_lynis_audit=true"
```

### Install on Remote Servers

1. Edit `inventory.yml` and add your target servers:

```yaml
fedora_servers:
  hosts:
    server1:
      ansible_host: 192.168.1.100
      ansible_user: admin
    server2:
      ansible_host: 192.168.1.101
      ansible_user: admin
```

2. Run the playbook:

```bash
ansible-playbook -i inventory.yml site.yml
```

## Using the Security Tools

### Lynis Security Auditing

```bash
# Quick system audit
sudo lynis audit system --quick

# Full system audit
sudo lynis audit system

# View Lynis version
lynis show version

# View audit report
lynis show report
```

Lynis logs are stored in `/var/log/lynis/lynis.log`

### Firewall Management

```bash
# Check firewall status
sudo firewall-cmd --state

# List active zones
sudo firewall-cmd --get-active-zones

# List all services in default zone
sudo firewall-cmd --list-all

# Add a service (e.g., HTTP)
sudo firewall-cmd --add-service=http --permanent
sudo firewall-cmd --reload

# Launch GUI tool
firewall-config
```

### ClamAV Antivirus

```bash
# Scan a directory
sudo clamscan -r /home

# Scan and remove infected files
sudo clamscan -r --remove /path/to/scan

# Update virus definitions manually
sudo freshclam

# Check ClamAV version
clamscan --version

# Launch GUI tool
clamtk
```

ClamAV automatically updates virus definitions via the clamav-freshclam service

## Variables

You can customize the installation with these variables:

- `run_lynis_audit`: Run security audit after installation (default: `false`)

Example:
```bash
ansible-playbook -i inventory.yml site.yml -e "run_lynis_audit=true"
```

## Supported Distributions

- Fedora (all recent versions)
- RHEL 8+
- CentOS 8+
- Rocky Linux 8+
- AlmaLinux 8+

## License

This project is open source.

## Features

- **Automated Installation**: One-command setup of all security tools
- **Smart ClamAV Updates**: Runs freshclam only on new installations to avoid conflicts
- **Automatic Virus Definition Updates**: Enables clamav-freshclam service for scheduled updates
- **Firewall Management**: Installs both CLI and GUI tools for firewall configuration
- **Idempotent**: Safe to run multiple times without side effects
- **EPEL Support**: Automatically configures EPEL repository when needed

## More Information

### Lynis
- Official Website: https://cisofy.com/lynis/
- GitHub: https://github.com/CISOfy/lynis

### Firewall
- Documentation: https://firewalld.org/documentation/
- RHEL Guide: https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/8/html/configuring_and_managing_networking/using-and-configuring-firewalld_configuring-and-managing-networking

### ClamAV
- Official Website: https://www.clamav.net/
- Documentation: https://docs.clamav.net/
