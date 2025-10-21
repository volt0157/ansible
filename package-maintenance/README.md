# Ansible Multi-Distro Package Maintenance

Automated package maintenance for multiple Linux distributions using Ansible.

## Supported Package Managers

### Native Package Managers
- **APT** - Debian, Ubuntu, Linux Mint
- **DNF** - Fedora, RHEL 8+, CentOS Stream
- **YUM** - RHEL/CentOS 7
- **Zypper** - openSUSE, SLES
- **Pacman** - Arch Linux, Manjaro

### Universal Package Managers
- **Flatpak** - Cross-distro app platform
- **Snap** - Ubuntu's universal package system

## Usage

```bash
# Update everything on localhost
ansible-playbook site.yml

# Update only specific package manager
ansible-playbook site.yml --tags apt
ansible-playbook site.yml --tags dnf
ansible-playbook site.yml --tags flatpak

# Update only native package managers
ansible-playbook site.yml --tags native

# Update only universal package managers
ansible-playbook site.yml --tags universal
```

## Project Structure

```
ansible-package-maintenance/
├── ansible.cfg
├── inventory.yml
├── site.yml
├── README.md
└── roles/
    ├── apt_maintenance/
    ├── dnf_maintenance/
    ├── yum_maintenance/
    ├── zypper_maintenance/
    ├── pacman_maintenance/
    ├── flatpak_maintenance/
    └── snap_maintenance/
```

## Features

- ✅ Automatic distribution detection
- ✅ Updates package cache
- ✅ Upgrades all packages
- ✅ Removes unnecessary packages
- ✅ Cleans package cache
- ✅ Works on any supported Linux distribution
- ✅ Tag-based filtering
- ✅ Localhost and remote host support

## Requirements

- Ansible 2.9+
- Python 3.6+
- sudo privileges on target hosts

## License

MIT
