# Fedora Security Tools - Lynis Installation

This Ansible project installs and configures Lynis, a security auditing tool for Unix-based systems.

## What is Lynis?

Lynis is an open-source security auditing tool for Linux, macOS, and Unix-based systems. It performs:
- Security scans and audits
- System hardening checks
- Compliance testing
- Vulnerability detection

## Project Structure

```
fedora_sec_tools/
├── site.yml                          # Main playbook
├── inventory.yml                     # Inventory file
├── roles/
│   └── lynis_install/
│       └── tasks/
│           └── main.yml              # Lynis installation tasks
└── README.md                         # This file
```

## Prerequisites

- Ansible 2.9 or higher
- Fedora, RHEL, CentOS, Rocky Linux, or AlmaLinux target systems
- Root or sudo privileges on target systems

## Usage

### Basic Installation

To install Lynis on localhost:

```bash
ansible-playbook -i inventory.yml site.yml
```

### Install with Security Audit

To install Lynis and immediately run a security audit:

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

## Running Lynis Manually

After installation, you can run Lynis manually:

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

## Logs and Reports

Lynis logs are stored in `/var/log/lynis/lynis.log`

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

## More Information

- Lynis Official Website: https://cisofy.com/lynis/
- Lynis GitHub: https://github.com/CISOfy/lynis
