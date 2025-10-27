# Node.js and npm Packages Installation with Ansible

A production-ready Ansible project for automated installation and configuration of Node.js and npm packages on RHEL-based systems, with full support for multiple shells (Bash, Zsh, Fish).

## Overview

This Ansible playbook automates the complete setup of Node.js development environment, including:

- Node.js installation via DNF package manager
- npm global directory configuration
- Multi-shell PATH configuration (Bash, Zsh, Fish)
- Automated installation of npm packages (Gemini CLI)
- System verification and validation
- User-friendly output and guidance

## Supported Systems

### Operating Systems
- Fedora 38 and later
- Red Hat Enterprise Linux (RHEL) 8 and later
- CentOS Stream 8 and later
- Rocky Linux 8 and later
- AlmaLinux 8 and later

### Shells
- Bash (via `~/.bashrc`)
- Zsh (via `~/.zshrc`)
- Fish (via `~/.config/fish/config.fish`)

### Requirements
- Minimum 2GB RAM
- DNF package manager
- sudo/root access for system packages
- User account for npm global configuration

## Quick Start

### 1. Clone or Navigate to Project

```bash
cd nodejs-npm-packages
```

### 2. Run Complete Installation

```bash
ansible-playbook site.yml
```

### 3. Reload Shell Configuration

**Bash:**
```bash
source ~/.bashrc
```

**Zsh:**
```bash
source ~/.zshrc
```

**Fish:**
```fish
source ~/.config/fish/config.fish
```

Or simply open a new terminal window.

### 4. Verify Installation

```bash
node --version
npm --version
gemini --version
which gemini
```

## Usage Examples

### Install Everything

```bash
ansible-playbook site.yml
```

### Install Only Node.js

```bash
ansible-playbook site.yml --tags nodejs
```

### Configure npm Only

```bash
ansible-playbook site.yml --tags npm_config
```

### Install Gemini CLI Only

```bash
ansible-playbook site.yml --tags gemini_cli
```

### Install All npm Packages

```bash
ansible-playbook site.yml --tags npm_packages
```

### Run Verification Only

```bash
ansible-playbook site.yml --tags verify
```

### Combine Multiple Tags

```bash
ansible-playbook site.yml --tags "nodejs,npm_config"
```

## Project Structure

```
nodejs-npm-packages/
├── README.md                          # This file
├── NODE_INSTALLATION_LOG.md           # Manual installation reference
├── ansible.cfg                        # Ansible configuration
├── inventory.yml                      # Host and variable definitions
├── site.yml                           # Main playbook
└── roles/                             # Ansible roles
    ├── nodejs_install/                # Node.js installation
    │   └── tasks/
    │       └── main.yml
    ├── npm_config/                    # npm configuration with multi-shell support
    │   └── tasks/
    │       └── main.yml
    └── npm_packages/                  # npm package installations
        └── gemini_cli/                # Gemini CLI package
            └── tasks/
                └── main.yml
```

## Configuration

### Variables (inventory.yml)

```yaml
nodejs_version: "20"                           # Node.js major version
npm_global_dir: "{{ ansible_env.HOME }}/.npm-global"  # Global npm directory
```

### Customization

To modify the Node.js version or npm global directory, edit `inventory.yml`:

```yaml
vars:
  nodejs_version: "22"                         # Change to desired version
  npm_global_dir: "{{ ansible_env.HOME }}/.my-npm"  # Custom directory
```

## Shell Configuration Details

The playbook automatically detects your shell and configures the appropriate file:

### Bash Configuration

**File:** `~/.bashrc`

**Added line:**
```bash
export PATH=~/.npm-global/bin:$PATH
```

### Zsh Configuration

**File:** `~/.zshrc`

**Added line:**
```zsh
export PATH=~/.npm-global/bin:$PATH
```

### Fish Configuration

**File:** `~/.config/fish/config.fish`

**Added line:**
```fish
set -gx PATH ~/.npm-global/bin $PATH
```

## Post-Installation Setup

### For Gemini CLI

1. Get your API key from: https://aistudio.google.com/app/apikey

2. Configure Gemini CLI:
   ```bash
   gemini config
   ```

3. Enter your API key when prompted

4. Test Gemini CLI:
   ```bash
   gemini ask "What is Ansible?"
   ```

### Common Commands

```bash
# Display help
gemini --help

# Check version
gemini --version

# Ask a question
gemini ask "your question here"

# Interactive mode
gemini
```

## Verification Commands

### Check Node.js

```bash
node --version          # Should show v20.x.x or later
node -e "console.log('Node.js is working')"
```

### Check npm

```bash
npm --version           # Should show npm version
npm config get prefix   # Should show ~/.npm-global
npm list -g --depth=0   # List global packages
```

### Check Gemini CLI

```bash
which gemini            # Should show ~/.npm-global/bin/gemini
gemini --version        # Should show Gemini CLI version
```

### Check PATH

**Bash/Zsh:**
```bash
echo $PATH | grep npm-global
```

**Fish:**
```fish
echo $PATH | grep npm-global
```

## Troubleshooting

### Issue: Command not found after installation

**Symptom:** `gemini: command not found`

**Solution:**
1. Reload your shell configuration:
   - Bash: `source ~/.bashrc`
   - Zsh: `source ~/.zshrc`
   - Fish: `source ~/.config/fish/config.fish`

2. Or open a new terminal window

3. Verify PATH:
   ```bash
   echo $PATH | grep npm-global
   ```

### Issue: npm prefix not set correctly

**Symptom:** Packages installing to system directory

**Solution:**
```bash
npm config set prefix "~/.npm-global"
```

### Issue: Permission denied during installation

**Symptom:** EACCES errors during npm install

**Solution:**
The playbook installs to user directory (~/.npm-global) which doesn't require sudo.
Never use `sudo npm install -g`. Re-run the playbook to fix permissions.

### Issue: Node.js version too old

**Symptom:** Node.js version is below v18

**Solution:**
```bash
# Update Node.js
sudo dnf update nodejs

# Verify version
node --version
```

### Issue: Shell configuration not applied

**Symptom:** PATH not updated after running playbook

**Solution:**
1. Check which shell you're using:
   ```bash
   echo $SHELL
   ```

2. Manually verify the configuration file:
   - Bash: `cat ~/.bashrc | grep npm-global`
   - Zsh: `cat ~/.zshrc | grep npm-global`
   - Fish: `cat ~/.config/fish/config.fish | grep npm-global`

3. If missing, re-run with npm_config tag:
   ```bash
   ansible-playbook site.yml --tags npm_config
   ```

### Issue: Gemini CLI not working

**Symptom:** Gemini CLI installed but commands fail

**Solution:**
1. Verify installation:
   ```bash
   ls -la ~/.npm-global/bin/gemini
   ```

2. Check API key configuration:
   ```bash
   gemini config
   ```

3. Test with simple command:
   ```bash
   gemini --help
   ```

### Issue: Playbook fails on system checks

**Symptom:** Pre-task assertions fail

**Solution:**
1. Ensure minimum 2GB RAM available
2. Verify you're on a RHEL-based system:
   ```bash
   cat /etc/os-release
   ```
3. Check DNF is available:
   ```bash
   which dnf
   ```

## Security Considerations

### No sudo Required for npm

This playbook configures npm to install global packages in the user's home directory (`~/.npm-global`), eliminating the need for sudo during npm package installation.

**Benefits:**
- No system-wide changes for npm packages
- Better security (no root access needed)
- Easier cleanup (just delete ~/.npm-global)
- Per-user package management

### sudo Only for System Packages

sudo is only required for:
- Installing Node.js via DNF (system package)
- System-level operations (if any)

### API Keys and Credentials

- Never commit API keys to version control
- Store API keys securely using Gemini CLI's config
- Use environment variables for sensitive data when possible

## Adding More npm Packages

### Method 1: Add New Role

1. Create new role directory:
   ```bash
   mkdir -p roles/npm_packages/your_package/tasks
   ```

2. Create tasks file:
   ```yaml
   # roles/npm_packages/your_package/tasks/main.yml
   ---
   - name: Install your package globally
     npm:
       name: "your-package-name"
       global: yes
       state: present
     environment:
       npm_config_prefix: "{{ ansible_env.HOME }}/.npm-global"
   ```

3. Add role to site.yml:
   ```yaml
   - role: npm_packages/your_package
     tags: ['your_package', 'npm_packages']
   ```

### Method 2: Manual Installation

After running the playbook:
```bash
npm install -g package-name
```

The package will be installed to `~/.npm-global/bin` automatically.

## Tags Reference

| Tag | Description |
|-----|-------------|
| `nodejs` | Install Node.js only |
| `npm_config` | Configure npm and shell PATH only |
| `gemini_cli` | Install Gemini CLI only |
| `npm_packages` | Install all npm packages |
| `verify` | Run verification checks only |
| `always` | Always runs (pre-tasks and post-tasks) |

## Best Practices

### When to Re-run the Playbook

- After system updates that affect Node.js
- When switching shells (Bash → Zsh → Fish)
- To verify/repair npm configuration
- To reinstall npm packages

### Keeping Node.js Updated

```bash
# Check for updates
sudo dnf check-update nodejs

# Update Node.js
sudo dnf update nodejs

# Verify new version
node --version
```

### Managing Global Packages

```bash
# List installed global packages
npm list -g --depth=0

# Update all global packages
npm update -g

# Update specific package
npm update -g @google/gemini-cli

# Uninstall package
npm uninstall -g package-name
```

### Clean Installation

To start fresh:

```bash
# Remove npm global directory
rm -rf ~/.npm-global

# Re-run playbook
ansible-playbook site.yml
```

## Development and Testing

### Syntax Check

```bash
ansible-playbook site.yml --syntax-check
```

### List Tasks

```bash
ansible-playbook site.yml --list-tasks
```

### List Tags

```bash
ansible-playbook site.yml --list-tags
```

### Dry Run

```bash
ansible-playbook site.yml --check
```

### Verbose Output

```bash
ansible-playbook site.yml -v    # Verbose
ansible-playbook site.yml -vv   # More verbose
ansible-playbook site.yml -vvv  # Debug level
```

## Additional Resources

### Documentation
- [Node.js Official Documentation](https://nodejs.org/docs/)
- [npm Documentation](https://docs.npmjs.com/)
- [Ansible Documentation](https://docs.ansible.com/)
- [Gemini API Documentation](https://ai.google.dev/docs)

### Related Files
- `NODE_INSTALLATION_LOG.md` - Manual installation reference
- `ansible.cfg` - Ansible configuration details
- `inventory.yml` - Variable definitions

## Contributing

To add new features or npm packages:

1. Follow the existing role structure
2. Add appropriate tags
3. Update README.md with new usage examples
4. Test on supported systems
5. Update NODE_INSTALLATION_LOG.md if adding manual steps

## License

This project is provided as-is for educational and production use.

## Support

For issues or questions:
1. Check the Troubleshooting section above
2. Review NODE_INSTALLATION_LOG.md for manual steps
3. Verify system meets requirements
4. Check Ansible and Node.js documentation

---

**Built with Ansible for automated, repeatable, production-ready deployments**
