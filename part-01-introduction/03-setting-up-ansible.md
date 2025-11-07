# Chapter 3: Setting up Ansible

## 📖 Introduction

Getting Ansible up and running is straightforward. In this chapter, you'll learn how to install Ansible on different operating systems, configure it properly, and verify your installation with your first Ansible commands.

## 🎯 Learning Objectives

By the end of this chapter, you will:
- Install Ansible on Linux, macOS, and Windows (WSL)
- Understand Ansible configuration files
- Create your first inventory
- Execute your first Ansible commands
- Troubleshoot common installation issues

---

## 1. System Requirements

### 1.1 Control Node Requirements

**Operating System**:
- ✅ Linux (RHEL, CentOS, Fedora, Debian, Ubuntu, etc.)
- ✅ macOS
- ✅ WSL (Windows Subsystem for Linux)
- ❌ Windows (cannot be a control node, only managed node)

**Software Requirements**:
```yaml
Python: 3.8+ (3.11+ recommended)
SSH Client: OpenSSH or compatible
Additional: 
  - pip (Python package installer)
  - git (for version control)
  - Text editor (vim, nano, VS Code, etc.)
```

**Hardware Requirements**:
```yaml
Minimum:
  CPU: 1 core
  RAM: 512 MB
  Disk: 500 MB

Recommended:
  CPU: 2+ cores
  RAM: 2+ GB
  Disk: 5+ GB (for collections and roles)
```

### 1.2 Managed Node Requirements

**Linux/Unix Nodes**:
```yaml
Python: 2.7 or 3.5+
SSH: SSH server running
User: Account with sudo privileges
```

**Windows Nodes**:
```yaml
PowerShell: 3.0+
.NET Framework: 4.0+
WinRM: Configured and enabled
```

---

## 2. Installing Ansible

### 2.1 Installing on Ubuntu/Debian

**Method 1: Using APT (Recommended)**

```bash
# Update package index
sudo apt update

# Install software-properties-common (for add-apt-repository)
sudo apt install software-properties-common

# Add Ansible PPA
sudo add-apt-repository --yes --update ppa:ansible/ansible

# Install Ansible
sudo apt install ansible

# Verify installation
ansible --version
```

**Method 2: Using pip**

```bash
# Install pip if not already installed
sudo apt update
sudo apt install python3-pip

# Install Ansible via pip
python3 -m pip install --user ansible

# Add to PATH if needed
echo 'export PATH=$PATH:$HOME/.local/bin' >> ~/.bashrc
source ~/.bashrc

# Verify installation
ansible --version
```

**Expected Output**:
```
ansible [core 2.15.5]
  config file = None
  configured module search path = ['/home/user/.ansible/plugins/modules']
  ansible python module location = /usr/lib/python3/dist-packages/ansible
  ansible collection location = /home/user/.ansible/collections
  executable location = /usr/bin/ansible
  python version = 3.10.12
```

### 2.2 Installing on RHEL/CentOS/Fedora

**RHEL 9 / CentOS Stream 9**:

```bash
# Enable EPEL repository
sudo dnf install epel-release

# Install Ansible
sudo dnf install ansible

# Verify installation
ansible --version
```

**RHEL 8 / CentOS 8**:

```bash
# Enable Ansible repository
sudo subscription-manager repos --enable ansible-2-for-rhel-8-x86_64-rpms

# Or use EPEL
sudo dnf install epel-release

# Install Ansible
sudo dnf install ansible

# Verify installation
ansible --version
```

**Fedora**:

```bash
# Install Ansible (included in default repos)
sudo dnf install ansible

# Verify installation
ansible --version
```

### 2.3 Installing on macOS

**Method 1: Using Homebrew (Recommended)**

```bash
# Install Homebrew if not installed
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"

# Install Ansible
brew install ansible

# Verify installation
ansible --version
```

**Method 2: Using pip**

```bash
# Install Python 3 if not installed
brew install python3

# Install Ansible
python3 -m pip install --user ansible

# Add to PATH
echo 'export PATH=$PATH:$HOME/Library/Python/3.x/bin' >> ~/.zshrc
source ~/.zshrc

# Verify installation
ansible --version
```

### 2.4 Installing on Windows (WSL)

**Step 1: Install WSL**

```powershell
# Open PowerShell as Administrator
# Install WSL with Ubuntu
wsl --install

# Restart computer if prompted
```

**Step 2: Install Ansible in WSL**

```bash
# Launch Ubuntu from WSL
# Update package list
sudo apt update

# Install Ansible
sudo apt install software-properties-common
sudo add-apt-repository --yes --update ppa:ansible/ansible
sudo apt install ansible

# Verify installation
ansible --version
```

**Alternative: Using pip in WSL**

```bash
# Install pip
sudo apt update
sudo apt install python3-pip

# Install Ansible
python3 -m pip install --user ansible

# Add to PATH
echo 'export PATH=$PATH:$HOME/.local/bin' >> ~/.bashrc
source ~/.bashrc

# Verify
ansible --version
```

### 2.5 Installing from Source (Advanced)

```bash
# Install Git and Python dependencies
sudo apt install git python3-pip python3-dev

# Clone Ansible repository
git clone https://github.com/ansible/ansible.git
cd ansible

# Install from source
python3 -m pip install --user -e .

# Source the environment script
source hacking/env-setup

# Verify
ansible --version
```

### 2.6 Installing Specific Versions

**Using pip**:

```bash
# Install specific version
python3 -m pip install ansible-core==2.15.0

# Install latest 2.14.x
python3 -m pip install 'ansible-core>=2.14,<2.15'

# Upgrade to latest
python3 -m pip install --upgrade ansible
```

**Using Virtual Environment (Best Practice)**:

```bash
# Create virtual environment
python3 -m venv ~/ansible-venv

# Activate virtual environment
source ~/ansible-venv/bin/activate

# Install Ansible in venv
pip install ansible

# Verify
which ansible
ansible --version

# Deactivate when done
deactivate
```

---

## 3. Ansible Configuration Files

### 3.1 Configuration File Precedence

Ansible reads configuration in this order (first match wins):

```
1. ANSIBLE_CONFIG (environment variable)
2. ansible.cfg (in current directory)
3. ~/.ansible.cfg (in home directory)
4. /etc/ansible/ansible.cfg (system-wide)
```

**Check Current Configuration**:
```bash
# Show current config file
ansible --version

# Show all configuration settings
ansible-config dump

# Show only changed settings
ansible-config dump --only-changed

# List all config options
ansible-config list
```

### 3.2 Creating ansible.cfg

**Location Options**:

**Option 1: Project-specific (Recommended)**
```bash
# Create in your project directory
cd ~/ansible-projects/myproject
touch ansible.cfg
```

**Option 2: User-wide**
```bash
# Create in home directory
touch ~/.ansible.cfg
```

**Option 3: System-wide**
```bash
# Create system-wide (requires sudo)
sudo mkdir -p /etc/ansible
sudo touch /etc/ansible/ansible.cfg
```

### 3.3 Basic ansible.cfg Configuration

**Minimal Configuration**:
```ini
[defaults]
inventory = ./inventory
host_key_checking = False
```

**Recommended Configuration**:
```ini
[defaults]
# Inventory
inventory = ./inventory

# SSH Settings
host_key_checking = False
timeout = 30

# Output
stdout_callback = yaml
callbacks_enabled = timer, profile_tasks

# Logging
log_path = ./ansible.log

# Performance
forks = 10
gathering = smart
fact_caching = jsonfile
fact_caching_connection = /tmp/ansible_facts
fact_caching_timeout = 86400

# Privilege Escalation
become = False
become_method = sudo
become_user = root
become_ask_pass = False

[ssh_connection]
# SSH Multiplexing
ssh_args = -o ControlMaster=auto -o ControlPersist=60s
pipelining = True
control_path = %(directory)s/%%h-%%p-%%r

[privilege_escalation]
become = True
become_method = sudo
become_user = root
```

**Production Configuration**:
```ini
[defaults]
inventory = ./inventory/production
host_key_checking = True
retry_files_enabled = False
log_path = /var/log/ansible/ansible.log
roles_path = ./roles:~/.ansible/roles:/etc/ansible/roles
collections_path = ./collections:~/.ansible/collections
vault_password_file = ~/.ansible/vault_pass.txt

# Callbacks
stdout_callback = yaml
callbacks_enabled = timer, profile_tasks, profile_roles

# Performance
forks = 20
gathering = smart
fact_caching = redis
fact_caching_connection = localhost:6379:0
fact_caching_timeout = 3600

# Error Handling
retry_files_enabled = True
retry_files_save_path = ./retry

[ssh_connection]
ssh_args = -o ControlMaster=auto -o ControlPersist=1800s -o ServerAliveInterval=60
pipelining = True
control_path = /tmp/ansible-ssh-%%h-%%p-%%r

[privilege_escalation]
become = True
become_method = sudo
become_ask_pass = False
```

### 3.4 Configuration Sections Explained

**[defaults] Section**:
```ini
[defaults]
# Where to find inventory
inventory = ./inventory

# SSH timeout (seconds)
timeout = 30

# Disable SSH key checking (dev only!)
host_key_checking = False

# Number of parallel processes
forks = 5

# Roles search path
roles_path = ./roles:~/.ansible/roles

# Collections search path
collections_path = ./collections

# Log file location
log_path = ./ansible.log

# Disable retry files
retry_files_enabled = False
```

**[privilege_escalation] Section**:
```ini
[privilege_escalation]
# Enable sudo by default
become = True

# Sudo method
become_method = sudo

# Sudo to this user
become_user = root

# Prompt for sudo password
become_ask_pass = False
```

**[ssh_connection] Section**:
```ini
[ssh_connection]
# SSH arguments for optimization
ssh_args = -o ControlMaster=auto -o ControlPersist=60s

# Enable pipelining (faster)
pipelining = True

# Control socket path
control_path = %(directory)s/%%h-%%p-%%r
```

---

## 4. Creating Your First Inventory

### 4.1 Simple INI Inventory

**Create inventory file**:
```bash
# Create inventory file
touch inventory
```

**Edit inventory** (inventory):
```ini
# Single host
testserver ansible_host=192.168.1.100

# Multiple hosts
web1.example.com
web2.example.com
db1.example.com

# Host groups
[webservers]
web1.example.com
web2.example.com

[databases]
db1.example.com
db2.example.com

# Group of groups
[production:children]
webservers
databases

# Group variables
[webservers:vars]
ansible_user=ubuntu
ansible_port=22

[databases:vars]
ansible_user=dbadmin
```

### 4.2 YAML Inventory

**Create inventory.yml**:
```yaml
all:
  hosts:
    testserver:
      ansible_host: 192.168.1.100
  
  children:
    webservers:
      hosts:
        web1.example.com:
        web2.example.com:
      vars:
        ansible_user: ubuntu
        ansible_port: 22
    
    databases:
      hosts:
        db1.example.com:
        db2.example.com:
      vars:
        ansible_user: dbadmin
    
    production:
      children:
        webservers:
        databases:
```

### 4.3 Inventory Variables

**Common Inventory Variables**:
```ini
# Connection variables
ansible_host         # IP or hostname
ansible_port         # SSH port (default: 22)
ansible_user         # SSH user
ansible_password     # SSH password (use vault!)
ansible_connection   # Connection type (ssh, winrm, local)

# SSH specific
ansible_ssh_private_key_file  # SSH key path
ansible_ssh_common_args       # SSH arguments

# Privilege escalation
ansible_become              # Enable sudo
ansible_become_method       # sudo, su, pbrun, etc.
ansible_become_user         # User to become
ansible_become_password     # Sudo password (use vault!)

# Python interpreter
ansible_python_interpreter  # Python path
```

**Example with Variables**:
```ini
[webservers]
web1.example.com ansible_host=192.168.1.10 ansible_user=ubuntu
web2.example.com ansible_host=192.168.1.11 ansible_user=ubuntu

[webservers:vars]
ansible_port=22
ansible_ssh_private_key_file=~/.ssh/ansible_key
ansible_become=yes
ansible_become_method=sudo
```

---

## 5. Testing Your Installation

### 5.1 Setup SSH Keys

**Generate SSH Key**:
```bash
# Generate new SSH key
ssh-keygen -t ed25519 -C "ansible-control" -f ~/.ssh/ansible_key

# Start SSH agent
eval $(ssh-agent)

# Add key to agent
ssh-add ~/.ssh/ansible_key
```

**Copy Key to Managed Nodes**:
```bash
# Copy to target hosts
ssh-copy-id -i ~/.ssh/ansible_key.pub user@target-host

# Or manually
cat ~/.ssh/ansible_key.pub | ssh user@target-host 'mkdir -p ~/.ssh && cat >> ~/.ssh/authorized_keys'
```

**Test SSH Connection**:
```bash
# Test manual SSH
ssh -i ~/.ssh/ansible_key user@target-host

# If successful, exit
exit
```

### 5.2 Your First Ansible Commands

**Ping Module** (Test Connectivity):
```bash
# Ping all hosts
ansible all -m ping

# Ping specific group
ansible webservers -m ping

# Ping specific host
ansible web1.example.com -m ping
```

**Expected Output**:
```
web1.example.com | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false,
    "ping": "pong"
}
```

**Gather Facts**:
```bash
# Gather system information
ansible all -m setup

# Specific facts
ansible all -m setup -a "filter=ansible_distribution*"

# Memory facts
ansible all -m setup -a "filter=ansible_memory*"
```

**Command Module**:
```bash
# Run command on all hosts
ansible all -m command -a "uptime"

# Check disk space
ansible all -m command -a "df -h"

# Check hostname
ansible all -m command -a "hostname"
```

**Shell Module**:
```bash
# Use shell for pipes and redirects
ansible all -m shell -a "ps aux | grep ssh"

# Environment variables
ansible all -m shell -a "echo $HOME"
```

### 5.3 Verbose Output

```bash
# Normal verbosity
ansible all -m ping

# Verbose (-v)
ansible all -m ping -v

# More verbose (-vv)
ansible all -m ping -vv

# Very verbose (-vvv) - shows SSH debugging
ansible all -m ping -vvv

# Extremely verbose (-vvvv) - shows everything
ansible all -m ping -vvvv
```

---

## 6. Troubleshooting Common Issues

### Issue 1: "No hosts matched"

**Error**:
```
[WARNING]: No hosts matched, nothing to do
```

**Solutions**:
```bash
# Check inventory file exists
ls -la inventory

# Verify inventory syntax
ansible-inventory --list

# Check group names
ansible-inventory --graph

# Specify inventory explicitly
ansible -i inventory all -m ping
```

### Issue 2: "Permission denied (publickey)"

**Error**:
```
UNREACHABLE! => {
    "msg": "Failed to connect to the host via ssh: Permission denied (publickey)."
}
```

**Solutions**:
```bash
# Verify SSH key is in agent
ssh-add -l

# Add key to agent
ssh-add ~/.ssh/ansible_key

# Test manual SSH
ssh -i ~/.ssh/ansible_key user@host

# Check key permissions
chmod 600 ~/.ssh/ansible_key
chmod 644 ~/.ssh/ansible_key.pub

# Use password authentication (temporarily)
ansible all -m ping --ask-pass
```

### Issue 3: "Failed to connect to the host via ssh: Host key verification failed"

**Error**:
```
UNREACHABLE! => {
    "msg": "Failed to connect to the host via ssh: Host key verification failed."
}
```

**Solutions**:
```bash
# Option 1: Add host to known_hosts
ssh-keyscan -H target-host >> ~/.ssh/known_hosts

# Option 2: Disable host key checking (dev only!)
# In ansible.cfg:
[defaults]
host_key_checking = False

# Option 3: Remove old key
ssh-keygen -R target-host
```

### Issue 4: "sudo: a password is required"

**Error**:
```
FAILED! => {
    "msg": "Missing sudo password"
}
```

**Solutions**:
```bash
# Prompt for sudo password
ansible all -m ping --become --ask-become-pass

# Configure passwordless sudo on managed nodes
# Edit /etc/sudoers:
username ALL=(ALL) NOPASSWD:ALL
```

### Issue 5: Python not found

**Error**:
```
FAILED! => {
    "msg": "/bin/sh: /usr/bin/python: No such file or directory"
}
```

**Solutions**:
```bash
# Specify Python interpreter in inventory
[all:vars]
ansible_python_interpreter=/usr/bin/python3

# Or auto-detect
ansible_python_interpreter=auto
```

---

## 7. Best Practices for Setup

### ✅ Directory Structure

**Recommended Project Structure**:
```
ansible-project/
├── ansible.cfg              # Project configuration
├── inventory/
│   ├── production          # Production inventory
│   ├── staging             # Staging inventory
│   └── group_vars/
│       ├── all.yml         # Variables for all hosts
│       ├── webservers.yml  # Variables for web servers
│       └── databases.yml   # Variables for databases
├── playbooks/
│   ├── site.yml            # Main playbook
│   ├── webservers.yml
│   └── databases.yml
├── roles/                  # Custom roles
├── collections/            # Collections
└── files/                  # Static files
```

### ✅ Security Best Practices

```yaml
1. Use SSH keys (not passwords)
2. Enable host key checking in production
3. Use Ansible Vault for secrets
4. Configure proper file permissions (600 for keys)
5. Use separate inventories for different environments
6. Enable logging for audit trails
7. Use jump hosts for isolated networks
8. Regularly rotate SSH keys
9. Use least privilege principle
10. Keep Ansible updated
```

### ✅ Performance Optimization

```ini
# ansible.cfg optimizations
[defaults]
forks = 20                    # Increase parallelism
gathering = smart             # Smart fact gathering
fact_caching = jsonfile       # Cache facts
fact_caching_timeout = 86400  # 24 hours

[ssh_connection]
pipelining = True             # Reduce SSH operations
ssh_args = -o ControlMaster=auto -o ControlPersist=1800s
```

---

## 📝 Chapter Summary

In this chapter, you learned:
- ✅ How to install Ansible on various operating systems
- ✅ Configuration file locations and precedence
- ✅ How to create and customize ansible.cfg
- ✅ How to create inventory files in INI and YAML formats
- ✅ How to test your Ansible installation
- ✅ How to troubleshoot common issues
- ✅ Best practices for Ansible setup

---

## 🧪 Hands-On Exercise

**Complete Setup Lab**:

```bash
# 1. Create project directory
mkdir -p ~/ansible-lab
cd ~/ansible-lab

# 2. Create ansible.cfg
cat > ansible.cfg <<EOF
[defaults]
inventory = ./inventory
host_key_checking = False
log_path = ./ansible.log
EOF

# 3. Create inventory (replace with your hosts)
cat > inventory <<EOF
[local]
localhost ansible_connection=local

[test]
# Add your test hosts here
# testhost ansible_host=192.168.1.100 ansible_user=ubuntu
EOF

# 4. Test localhost
ansible local -m ping

# 5. Gather facts from localhost
ansible local -m setup

# 6. Run a command
ansible local -m command -a "date"

# 7. Check configuration
ansible-config dump --only-changed

# 8. View inventory
ansible-inventory --graph
```

---

## 🤔 Review Questions

1. What are the minimum requirements for an Ansible control node?
2. In what order does Ansible read configuration files?
3. What is the difference between the `command` and `shell` modules?
4. How do you test connectivity to managed nodes?
5. What is the purpose of SSH key-based authentication?
6. How can you increase Ansible's parallel execution?
7. What is the `ping` module used for?
8. How do you troubleshoot SSH connection issues?

---

## 📚 Additional Resources

- [Ansible Installation Guide](https://docs.ansible.com/ansible/latest/installation_guide/index.html)
- [Ansible Configuration Settings](https://docs.ansible.com/ansible/latest/reference_appendices/config.html)
- [Inventory Guide](https://docs.ansible.com/ansible/latest/user_guide/intro_inventory.html)
- [Connection Methods](https://docs.ansible.com/ansible/latest/user_guide/connection_details.html)

---

[← Previous: Ansible Architecture](./02-ansible-architecture.md) | [Next: Part II - Core Concepts →](../part-02-core-concepts/README.md)
