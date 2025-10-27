# Node.js and npm Manual Installation Log

This document provides a reference for the manual installation steps that the Ansible playbook automates. It serves as a guide for understanding what the playbook does and for troubleshooting purposes.

## System Information

**Tested On:**
- Operating System: Fedora 42 Workstation Edition
- Kernel: 6.11.x
- Architecture: x86_64
- Node.js Version: 20.x (LTS)
- npm Version: 10.x
- Shell: Bash/Zsh/Fish

## Manual Installation Steps

### Step 1: Install Node.js via DNF

```bash
# Update package cache
sudo dnf check-update

# Install Node.js (includes npm)
sudo dnf install -y nodejs

# Verify installation
node --version
npm --version
```

**Expected Output:**
```
v20.x.x
10.x.x
```

**What the Playbook Does:**
- Uses Ansible's `dnf` module to install Node.js
- Registers the installation result
- Verifies Node.js and npm versions
- Asserts minimum version requirements (Node.js 18+)

---

### Step 2: Create npm Global Directory

```bash
# Create directory for global npm packages
mkdir -p ~/.npm-global

# Set correct permissions
chmod 755 ~/.npm-global

# Verify creation
ls -la ~/.npm-global
```

**Expected Output:**
```
drwxr-xr-x. 2 username username 4096 Oct 27 10:30 .npm-global
```

**What the Playbook Does:**
- Uses Ansible's `file` module to create directory
- Sets mode to 0755
- Sets owner to current user
- Ensures idempotent operation (doesn't fail if exists)

---

### Step 3: Configure npm Prefix

```bash
# Configure npm to use the global directory
npm config set prefix '~/.npm-global'

# Verify configuration
npm config get prefix
```

**Expected Output:**
```
/home/username/.npm-global
```

**What the Playbook Does:**
- Uses Ansible's `command` module to set npm prefix
- Registers configuration result
- Verifies the prefix setting
- Uses `changed_when` to track changes

---

### Step 4: Configure Shell PATH (Multi-Shell Support)

#### For Bash

```bash
# Add to ~/.bashrc
echo 'export PATH=~/.npm-global/bin:$PATH' >> ~/.bashrc

# Reload configuration
source ~/.bashrc

# Verify PATH
echo $PATH | grep npm-global
```

**Expected Output:**
```
/home/username/.npm-global/bin:/usr/local/bin:/usr/bin
```

#### For Zsh

```bash
# Add to ~/.zshrc
echo 'export PATH=~/.npm-global/bin:$PATH' >> ~/.zshrc

# Reload configuration
source ~/.zshrc

# Verify PATH
echo $PATH | grep npm-global
```

**Expected Output:**
```
/home/username/.npm-global/bin:/usr/local/bin:/usr/bin
```

#### For Fish

```bash
# Create Fish config directory if needed
mkdir -p ~/.config/fish

# Add to config.fish
echo 'set -gx PATH ~/.npm-global/bin $PATH' >> ~/.config/fish/config.fish

# Reload configuration
source ~/.config/fish/config.fish

# Verify PATH
echo $PATH | grep npm-global
```

**Expected Output:**
```
/home/username/.npm-global/bin /usr/local/bin /usr/bin
```

**What the Playbook Does:**
- Detects user's shell using `echo $SHELL`
- Uses Ansible's `lineinfile` module to add PATH
- Conditionally configures based on shell type:
  - Bash: `~/.bashrc` with `export PATH=...`
  - Zsh: `~/.zshrc` with `export PATH=...`
  - Fish: `~/.config/fish/config.fish` with `set -gx PATH ...`
- Creates config directory for Fish if needed
- Ensures idempotent operation (doesn't duplicate lines)

---

### Step 5: Install Gemini CLI

```bash
# Install Gemini CLI globally
npm install -g @google/gemini-cli

# Verify installation
which gemini
gemini --version
```

**Expected Output:**
```
/home/username/.npm-global/bin/gemini
1.x.x (or latest version)
```

**What the Playbook Does:**
- Uses Ansible's `npm` module with `global: yes`
- Sets environment variable: `npm_config_prefix`
- Registers installation result
- Verifies with `which gemini` command
- Checks version with `gemini --version`
- Displays usage instructions

---

### Step 6: Configure Gemini CLI

```bash
# Get API key from Google AI Studio
# Visit: https://aistudio.google.com/app/apikey

# Configure Gemini CLI
gemini config

# Enter API key when prompted
# Follow interactive prompts
```

**Expected Output:**
```
Welcome to Gemini CLI setup!
Please enter your API key: [enter key here]
Configuration saved successfully!
```

**What the Playbook Does:**
- The playbook installs Gemini CLI but doesn't configure API key
- Users must manually run `gemini config` after installation
- Playbook displays instructions for this step

---

## Verification Commands

### Check Node.js Installation

```bash
# Version check
node --version

# Test Node.js
node -e "console.log('Node.js is working!')"

# Check installation location
which node
```

**Expected Output:**
```
v20.x.x
Node.js is working!
/usr/bin/node
```

### Check npm Configuration

```bash
# Version check
npm --version

# Check prefix
npm config get prefix

# List global packages
npm list -g --depth=0

# Check installation location
which npm
```

**Expected Output:**
```
10.x.x
/home/username/.npm-global
├── @google/gemini-cli@x.x.x
/usr/bin/npm
```

### Check Gemini CLI

```bash
# Check location
which gemini

# Check version
gemini --version

# Display help
gemini --help

# Test with question (requires API key)
gemini ask "What is 2+2?"
```

**Expected Output:**
```
/home/username/.npm-global/bin/gemini
1.x.x
[help output]
The answer is 4.
```

### Check PATH Configuration

```bash
# Display PATH
echo $PATH

# Search for npm-global
echo $PATH | grep npm-global

# Verify PATH order
echo $PATH | tr ':' '\n' | head -5
```

**Expected Output:**
```
/home/username/.npm-global/bin:/usr/local/bin:/usr/bin:...
/home/username/.npm-global/bin:/usr/local/bin:/usr/bin:...
/home/username/.npm-global/bin
/usr/local/bin
/usr/bin
```

---

## Common Issues and Solutions

### Issue 1: npm Global Packages Not in PATH

**Problem:**
```bash
$ gemini --version
bash: gemini: command not found
```

**Solution:**
```bash
# Check if PATH is configured
echo $PATH | grep npm-global

# If not, reload shell config
source ~/.bashrc   # or ~/.zshrc or ~/.config/fish/config.fish

# Or add manually
export PATH=~/.npm-global/bin:$PATH  # Bash/Zsh
set -gx PATH ~/.npm-global/bin $PATH  # Fish
```

### Issue 2: npm Permission Errors

**Problem:**
```bash
$ npm install -g package
EACCES: permission denied
```

**Solution:**
```bash
# Reconfigure npm prefix
npm config set prefix '~/.npm-global'

# Verify
npm config get prefix

# Never use sudo with npm
# If you used sudo before, fix ownership:
sudo chown -R $USER:$USER ~/.npm-global
```

### Issue 3: Node.js Version Too Old

**Problem:**
```bash
$ node --version
v16.x.x
```

**Solution:**
```bash
# Update Node.js
sudo dnf update nodejs

# Or install specific version
sudo dnf install nodejs-20

# Verify
node --version
```

### Issue 4: Shell Config Not Applied

**Problem:**
Changes to shell config file don't take effect.

**Solution:**
```bash
# Reload configuration
source ~/.bashrc      # Bash
source ~/.zshrc       # Zsh
source ~/.config/fish/config.fish  # Fish

# Or open new terminal
# Or log out and log back in
```

### Issue 5: Gemini CLI API Key Issues

**Problem:**
```bash
$ gemini ask "test"
Error: API key not configured
```

**Solution:**
```bash
# Configure API key
gemini config

# Or set environment variable
export GEMINI_API_KEY="your-api-key-here"

# Or create config file manually
mkdir -p ~/.config/gemini
echo '{"apiKey":"your-key"}' > ~/.config/gemini/config.json
```

---

## What the Ansible Playbook Automates

The Ansible playbook (`site.yml`) automates all the above steps with:

### Pre-tasks
1. Display system information (OS, RAM, disk space)
2. Assert minimum requirements (2GB RAM, RHEL-based OS)
3. Detect user's shell
4. Create npm-global directory

### Roles
1. **nodejs_install**: Install Node.js via DNF and verify version
2. **npm_config**: Configure npm and shell PATH (multi-shell support)
3. **npm_packages/gemini_cli**: Install Gemini CLI globally

### Post-tasks
1. Verify all installations
2. Display comprehensive installation summary
3. Show shell-specific reload commands
4. Display verification commands
5. Show usage instructions

### Benefits of Automation
- **Idempotent**: Can run multiple times safely
- **Consistent**: Same result every time
- **Fast**: Completes in seconds
- **Reliable**: Error handling and verification
- **Documented**: Clear output and logging
- **Flexible**: Tag-based selective execution
- **Multi-shell**: Automatic shell detection and configuration

---

## Performance Metrics

**Manual Installation Time:** ~10-15 minutes
**Ansible Playbook Time:** ~2-3 minutes

**Steps Saved:**
- Manual commands: ~20
- Configuration file edits: ~3
- Verification commands: ~8
- Troubleshooting time: Variable

---

## Additional Notes

### Node.js Package Contents

The `nodejs` package from Fedora/RHEL repositories includes:
- Node.js runtime
- npm package manager
- Node.js documentation
- Development headers

### npm Global vs Local Packages

**Global Packages (`~/.npm-global`):**
- CLI tools (like Gemini CLI)
- Development tools (like TypeScript, ESLint)
- Utilities used across projects

**Local Packages (`./node_modules`):**
- Project-specific dependencies
- Libraries and frameworks
- Application packages

### Shell Configuration Files

**Bash:**
- `~/.bashrc` - Interactive non-login shells
- `~/.bash_profile` - Login shells
- `~/.profile` - Generic login shell

**Zsh:**
- `~/.zshrc` - Interactive shells
- `~/.zprofile` - Login shells
- `~/.zshenv` - All shells

**Fish:**
- `~/.config/fish/config.fish` - Main config
- `~/.config/fish/functions/` - Custom functions
- `~/.config/fish/conf.d/` - Auto-loaded configs

### Security Considerations

1. **No sudo for npm**: Global packages install to user directory
2. **Permissions**: User owns ~/.npm-global
3. **PATH order**: User bin directories before system
4. **API keys**: Never commit to version control

---

## Reference Links

- **Node.js Downloads**: https://nodejs.org/en/download/
- **npm Documentation**: https://docs.npmjs.com/
- **Gemini CLI**: https://www.npmjs.com/package/@google/gemini-cli
- **Google AI Studio**: https://aistudio.google.com/
- **Ansible DNF Module**: https://docs.ansible.com/ansible/latest/collections/ansible/builtin/dnf_module.html
- **Ansible npm Module**: https://docs.ansible.com/ansible/latest/collections/community/general/npm_module.html

---

**This manual installation log is maintained as a reference for what the Ansible playbook automates. For automated installation, use the playbook instead of following these manual steps.**

**Last Updated:** October 2024
**Tested On:** Fedora 42, Node.js 20.x
