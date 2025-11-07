# Chapter 2: Ansible Architecture

## 📖 Introduction

Understanding Ansible's architecture is crucial for effective automation. Unlike traditional configuration management tools, Ansible's agentless design makes it simple yet powerful. In this chapter, we'll explore how Ansible works under the hood.

## 🎯 Learning Objectives

By the end of this chapter, you will understand:
- The key components of Ansible architecture
- How control nodes and managed nodes interact
- The agentless architecture and its benefits
- Communication protocols (SSH and WinRM)
- How Ansible executes tasks

---

## 1. High-Level Architecture Overview

```
┌─────────────────────────────────────────────────────────────┐
│                      CONTROL NODE                            │
│  ┌──────────────┐  ┌──────────┐  ┌────────────────────┐    │
│  │   Ansible    │  │ Playbooks│  │    Inventory       │    │
│  │    Core      │  │   Roles  │  │  (hosts, groups)   │    │
│  └──────┬───────┘  └────┬─────┘  └─────────┬──────────┘    │
│         │               │                   │                │
│         └───────────────┴───────────────────┘                │
│                         │                                    │
└─────────────────────────┼────────────────────────────────────┘
                          │
                   SSH / WinRM
                          │
         ┌────────────────┼────────────────┐
         │                │                │
         ▼                ▼                ▼
┌─────────────┐  ┌─────────────┐  ┌─────────────┐
│   Managed   │  │   Managed   │  │   Managed   │
│    Node 1   │  │    Node 2   │  │    Node 3   │
│   (Linux)   │  │  (Windows)  │  │   (Linux)   │
└─────────────┘  └─────────────┘  └─────────────┘
```

---

## 2. Core Components

### 2.1 Control Node

**Definition**: The machine where Ansible is installed and from which you run commands and playbooks.

**Requirements**:
- Python 3.8+ (Python 3.11+ recommended)
- Linux, macOS, or WSL (Windows Subsystem for Linux)
- Cannot be a Windows machine (Windows can only be a managed node)

**Characteristics**:
```yaml
Location: Your laptop, CI/CD server, or dedicated automation server
Purpose: 
  - Execute playbooks
  - Store inventory files
  - Manage roles and variables
  - Connect to managed nodes
```

**What's Installed on Control Node**:
```bash
ansible/
├── Ansible Core (CLI tools)
├── Python libraries
├── Modules (3,000+ built-in)
├── Plugins
└── Collections
```

### 2.2 Managed Nodes

**Definition**: Target systems that Ansible manages (servers, network devices, cloud instances).

**Requirements**:
- **Linux/Unix**: Python 2.7+ or Python 3.5+
- **Windows**: PowerShell 3.0+ and .NET 4.0+
- **Network Devices**: No requirements (Ansible runs modules locally)

**No Agent Required**:
```
Traditional Tools          Ansible
─────────────────         ───────
Server → Agent            Server (nothing extra)
  ↓                         ↓
Agent checks in           SSH connection only
  ↓                         ↓
Pull configuration        Push configuration
```

**Characteristics**:
```yaml
What's NOT installed:
  - No Ansible agent
  - No background services
  - No database

What IS required:
  - SSH server (Linux) or WinRM (Windows)
  - Python (Linux) or PowerShell (Windows)
  - Proper user permissions
```

### 2.3 Inventory

**Definition**: A list of managed nodes organized into groups.

**Purpose**:
- Define which hosts to manage
- Organize hosts into logical groups
- Assign variables to hosts and groups

**Formats**:

**INI Format**:
```ini
# Simple inventory
[webservers]
web1.example.com
web2.example.com

[databases]
db1.example.com
db2.example.com

[production:children]
webservers
databases
```

**YAML Format**:
```yaml
all:
  children:
    webservers:
      hosts:
        web1.example.com:
        web2.example.com:
    databases:
      hosts:
        db1.example.com:
        db2.example.com:
```

**Dynamic Inventory**:
```python
# Query cloud providers (AWS, Azure, GCP)
# Generate inventory in real-time
# Scale automatically
```

### 2.4 Modules

**Definition**: Reusable, standalone scripts that Ansible executes on managed nodes.

**Categories**:
```yaml
System Modules:
  - user: Manage user accounts
  - service: Control services
  - yum/apt: Package management

Files Modules:
  - copy: Copy files
  - template: Process Jinja2 templates
  - file: Set file properties

Cloud Modules:
  - ec2: AWS EC2 instances
  - azure_rm: Azure resources
  - gcp_compute: GCP instances

Network Modules:
  - ios_command: Cisco IOS
  - junos_config: Juniper
  - eos_config: Arista
```

**How Modules Work**:
```
1. Ansible copies module to managed node
2. Module executes on target
3. Module returns JSON result
4. Ansible cleans up temporary files
```

**Example Module Usage**:
```yaml
- name: Install Apache
  yum:
    name: httpd
    state: present
```

### 2.5 Playbooks

**Definition**: YAML files containing ordered lists of tasks to execute.

**Structure**:
```yaml
---
- name: Configure Web Servers      # Play 1
  hosts: webservers
  become: yes
  tasks:
    - name: Install nginx           # Task 1
      apt:
        name: nginx
        state: present
    
    - name: Start nginx             # Task 2
      service:
        name: nginx
        state: started

- name: Configure Databases        # Play 2
  hosts: databases
  tasks:
    - name: Install PostgreSQL
      apt:
        name: postgresql
        state: present
```

**Key Concepts**:
- **Play**: Maps hosts to tasks
- **Task**: Calls a module with parameters
- **Handler**: Special task triggered by notifications

### 2.6 Plugins

**Definition**: Code that extends Ansible's core functionality.

**Types**:
```yaml
Connection Plugins: How Ansible connects (SSH, WinRM, Docker)
Callback Plugins: Control output and logging
Lookup Plugins: Read data from external sources
Filter Plugins: Transform data in templates
Inventory Plugins: Generate dynamic inventory
```

### 2.7 Collections

**Definition**: Distribution format for Ansible content (modules, plugins, roles).

**Structure**:
```
ansible_collections/
└── community/
    └── general/
        ├── plugins/
        ├── modules/
        ├── roles/
        └── playbooks/
```

**Example**:
```yaml
# Using collection module
- name: Manage Docker container
  community.docker.docker_container:
    name: myapp
    image: nginx:latest
    state: started
```

---

## 3. Agentless Architecture Explained

### 3.1 What Does "Agentless" Mean?

**Traditional Agent-Based Tools**:
```
┌──────────────────────────────────────┐
│ Server (Managed Node)                │
│  ┌────────────────────────────────┐  │
│  │ Agent Software                 │  │
│  │ - Runs continuously            │  │
│  │ - Uses memory/CPU              │  │
│  │ - Requires updates             │  │
│  │ - Security considerations      │  │
│  │ - Pulls configuration changes  │  │
│  └────────────────────────────────┘  │
└──────────────────────────────────────┘
```

**Ansible Agentless Approach**:
```
┌──────────────────────────────────────┐
│ Server (Managed Node)                │
│  ┌────────────────────────────────┐  │
│  │ SSH Server (already there)     │  │
│  │ Python (already installed)     │  │
│  │                                │  │
│  │ No extra software needed!      │  │
│  └────────────────────────────────┘  │
└──────────────────────────────────────┘
```

### 3.2 Benefits of Agentless Architecture

**✅ Simplified Management**
```yaml
No agents to:
  - Install
  - Configure
  - Update
  - Monitor
  - Troubleshoot
```

**✅ Reduced Resource Consumption**
```yaml
Managed nodes:
  - No background processes
  - No memory overhead
  - No CPU usage when idle
  - Only active during Ansible runs
```

**✅ Enhanced Security**
```yaml
Security benefits:
  - Smaller attack surface
  - No persistent connections
  - No agent vulnerabilities
  - Standard SSH/WinRM security
  - No firewall rule changes needed
```

**✅ Easy Adoption**
```yaml
Getting started:
  - No software rollout needed
  - Works with existing infrastructure
  - Immediate productivity
  - Low barrier to entry
```

**✅ Flexible Execution**
```yaml
Run from anywhere:
  - Your laptop
  - CI/CD pipeline
  - Jump host
  - Temporary container
```

### 3.3 How Agentless Works

**Execution Flow**:
```
1. User runs: ansible-playbook site.yml
         ↓
2. Ansible reads inventory and playbook
         ↓
3. For each host:
   a. Opens SSH connection
   b. Copies Python module to /tmp
   c. Executes module
   d. Returns JSON result
   e. Deletes temporary files
   f. Closes connection
         ↓
4. Ansible displays results
```

**Code Example**:
```bash
# What happens when you run:
ansible webservers -m ping

# Behind the scenes:
1. SSH to each host in 'webservers' group
2. Run: /usr/bin/python /tmp/ansible-tmp-xyz/ping.py
3. Receive: {"ping": "pong", "changed": false}
4. Clean up: rm -rf /tmp/ansible-tmp-xyz
5. Close SSH connection
```

### 3.4 Agentless Limitations

**⚠️ Challenges**:

1. **SSH Overhead**: Each task requires SSH connection
   ```yaml
   Impact: Slower than persistent agents for frequent updates
   Mitigation: Use pipelining, ControlPersist, multiplexing
   ```

2. **Network Dependency**: Requires network connectivity
   ```yaml
   Impact: Cannot update disconnected nodes
   Mitigation: Use pull mode (ansible-pull) for edge cases
   ```

3. **Python Requirement**: Managed nodes need Python
   ```yaml
   Impact: Minimal Linux/Unix systems might need Python installed
   Mitigation: Use raw module for bootstrapping Python
   ```

---

## 4. SSH and WinRM Communication

### 4.1 SSH (Secure Shell) - Linux/Unix

**How Ansible Uses SSH**:
```yaml
Connection Method: SSH (port 22 by default)
Authentication:
  - SSH keys (recommended)
  - Password (less secure)
  - Kerberos (enterprise)

Default SSH Client: OpenSSH
Alternative: Paramiko (pure Python implementation)
```

**SSH Connection Process**:
```
1. Control node initiates SSH connection
2. Authentication (key or password)
3. Ansible executes commands remotely
4. Results sent back over encrypted connection
5. Connection closed
```

**SSH Configuration Best Practices**:

```ini
# ~/.ssh/config
Host *.example.com
    User ansible
    IdentityFile ~/.ssh/ansible_key
    StrictHostKeyChecking no
    UserKnownHostsFile /dev/null
    ControlMaster auto
    ControlPath ~/.ssh/sockets/%r@%h-%p
    ControlPersist 600
```

**Ansible SSH Configuration**:
```ini
# ansible.cfg
[defaults]
host_key_checking = False
timeout = 30

[ssh_connection]
ssh_args = -o ControlMaster=auto -o ControlPersist=60s
pipelining = True
control_path = ~/.ansible/cp/%%h-%%p-%%r
```

**SSH Authentication Methods**:

**1. SSH Key (Recommended)**:
```bash
# Generate SSH key
ssh-keygen -t ed25519 -C "ansible@control"

# Copy to managed nodes
ssh-copy-id user@managed-node

# Test connection
ansible all -m ping
```

**2. Password Authentication**:
```bash
# Install sshpass
sudo apt install sshpass

# Run with password prompt
ansible all -m ping --ask-pass
```

**3. SSH Agent**:
```bash
# Start SSH agent
eval $(ssh-agent)

# Add key
ssh-add ~/.ssh/ansible_key

# Run playbooks (key used automatically)
ansible-playbook site.yml
```

### 4.2 WinRM (Windows Remote Management) - Windows

**How Ansible Uses WinRM**:
```yaml
Connection Method: WinRM (ports 5985/5986)
Protocol: HTTP (5985) or HTTPS (5986)
Authentication:
  - Basic
  - Certificate
  - Kerberos
  - NTLM
  - CredSSP
```

**WinRM Setup on Windows**:
```powershell
# Configure WinRM (run as Administrator)
# Download configuration script
Invoke-WebRequest -Uri https://raw.githubusercontent.com/ansible/ansible/devel/examples/scripts/ConfigureRemotingForAnsible.ps1 -OutFile ConfigureRemotingForAnsible.ps1

# Run script
.\ConfigureRemotingForAnsible.ps1 -EnableCredSSP

# Verify WinRM
winrm get winrm/config/service
```

**Ansible Configuration for Windows**:
```ini
# inventory
[windows]
win-server1.example.com

[windows:vars]
ansible_user=Administrator
ansible_password=SecurePassword123!
ansible_connection=winrm
ansible_winrm_server_cert_validation=ignore
ansible_winrm_transport=ntlm
ansible_port=5986
```

**Connection Types**:

| Transport | Encryption | Authentication | Use Case |
|-----------|-----------|----------------|----------|
| **basic** | ⚠️ Optional | Username/Password | Testing only |
| **ntlm** | ✅ Yes | NTLM | Workgroup |
| **kerberos** | ✅ Yes | Kerberos | Domain |
| **credssp** | ✅ Yes | NTLM/Kerberos | Double-hop |
| **certificate** | ✅ Yes | Certificate | Secure |

**Testing Windows Connection**:
```bash
# Ping Windows hosts
ansible windows -m win_ping

# Get system info
ansible windows -m setup
```

### 4.3 Connection Plugin Architecture

**Available Connection Plugins**:
```yaml
ssh: Default for Linux/Unix
winrm: Windows systems
local: Execute on control node
docker: Docker containers
kubectl: Kubernetes pods
network_cli: Network devices (Cisco, Juniper)
httpapi: REST API-based devices
```

**Specifying Connection Type**:

```yaml
# In playbook
- name: Configure Linux servers
  hosts: linux_servers
  connection: ssh
  tasks:
    - name: Install package
      apt:
        name: nginx

- name: Configure Windows servers
  hosts: windows_servers
  connection: winrm
  tasks:
    - name: Install IIS
      win_feature:
        name: Web-Server
```

---

## 5. Ansible Execution Flow

### 5.1 Detailed Execution Steps

```
┌─────────────────────────────────────────────────────────────┐
│ Step 1: Parse Configuration                                 │
│  - Read ansible.cfg                                         │
│  - Load defaults                                            │
└────────────────────┬────────────────────────────────────────┘
                     ▼
┌─────────────────────────────────────────────────────────────┐
│ Step 2: Parse Inventory                                     │
│  - Read inventory file(s)                                   │
│  - Build host and group lists                               │
│  - Apply variable precedence                                │
└────────────────────┬────────────────────────────────────────┘
                     ▼
┌─────────────────────────────────────────────────────────────┐
│ Step 3: Parse Playbook                                      │
│  - Load YAML playbook                                       │
│  - Validate syntax                                          │
│  - Build task list                                          │
└────────────────────┬────────────────────────────────────────┘
                     ▼
┌─────────────────────────────────────────────────────────────┐
│ Step 4: Gather Facts (if enabled)                           │
│  - Connect to each host                                     │
│  - Run setup module                                         │
│  - Collect system information                               │
└────────────────────┬────────────────────────────────────────┘
                     ▼
┌─────────────────────────────────────────────────────────────┐
│ Step 5: Execute Tasks                                       │
│  For each task:                                             │
│    - Template module parameters                             │
│    - Open connection to hosts (parallel)                    │
│    - Copy module to temp directory                          │
│    - Execute module                                         │
│    - Receive results                                        │
│    - Clean up temp files                                    │
│    - Close connection                                       │
└────────────────────┬────────────────────────────────────────┘
                     ▼
┌─────────────────────────────────────────────────────────────┐
│ Step 6: Run Handlers (if notified)                          │
│  - Execute handler tasks                                    │
│  - Run once per handler                                     │
└────────────────────┬────────────────────────────────────────┘
                     ▼
┌─────────────────────────────────────────────────────────────┐
│ Step 7: Display Results                                     │
│  - Show task outcomes                                       │
│  - Report changed/failed/ok status                          │
│  - Generate summary                                         │
└─────────────────────────────────────────────────────────────┘
```

### 5.2 Task Execution Details

**Single Task Execution**:
```
Task: Install nginx package

Control Node:
  ├── Read task definition
  ├── Template variables: {{ package_name }} → nginx
  ├── Select module: apt
  └── Connect to hosts

For each host (parallel):
  ├── SSH connection established
  ├── Create temp directory: /tmp/ansible-tmp-12345/
  ├── Copy module: apt.py → /tmp/ansible-tmp-12345/
  ├── Execute: /usr/bin/python3 /tmp/ansible-tmp-12345/apt.py
  ├── Module checks: Is nginx installed?
  │   ├── If yes: Return {"changed": false}
  │   └── If no: Install nginx, return {"changed": true}
  ├── Remove temp directory
  └── Close SSH connection

Control Node:
  ├── Collect all results
  └── Display output
```

### 5.3 Parallel Execution

**Forks (Parallelism)**:
```ini
# ansible.cfg
[defaults]
forks = 5    # Run on 5 hosts simultaneously
```

**How It Works**:
```
100 hosts, forks = 5

Batch 1: Hosts 1-5   (execute task)
Batch 2: Hosts 6-10  (execute task)
Batch 3: Hosts 11-15 (execute task)
...
Batch 20: Hosts 96-100 (execute task)
```

**Increasing Parallelism**:
```bash
# Command line override
ansible-playbook -f 20 site.yml    # 20 parallel forks

# For large environments
forks = 50  # Can handle hundreds of hosts faster
```

### 5.4 Connection Persistence

**ControlMaster (SSH Multiplexing)**:
```ini
# ansible.cfg
[ssh_connection]
ssh_args = -o ControlMaster=auto -o ControlPersist=60s
control_path = %(directory)s/%%h-%%p-%%r
```

**Benefits**:
- Single SSH connection for multiple tasks
- Faster execution
- Reduced authentication overhead

**Pipelining**:
```ini
[ssh_connection]
pipelining = True
```

**Benefits**:
- Reduces file transfers
- Faster execution
- Requires: `requiretty` disabled in sudoers

---

## 6. Architecture Best Practices

### ✅ Control Node Best Practices

```yaml
1. Use dedicated control node for production
2. Install latest Ansible version
3. Use Python virtual environments
4. Store playbooks in version control (Git)
5. Secure ansible.cfg and inventory files
6. Use Ansible Vault for secrets
7. Enable SSH key-based authentication
8. Configure SSH multiplexing
9. Set appropriate forks value
10. Monitor control node resources
```

### ✅ Managed Node Best Practices

```yaml
1. Keep Python up to date
2. Use consistent Python versions
3. Configure proper user permissions
4. Disable requiretty for sudo
5. Harden SSH configuration
6. Monitor disk space in /tmp
7. Set up logging
8. Use non-root ansible user with sudo
9. Implement SSH key rotation
10. Keep OS packages updated
```

### ✅ Network Best Practices

```yaml
1. Use jump hosts for isolated networks
2. Configure SSH bastion hosts
3. Implement VPN for secure access
4. Use connection caching
5. Set appropriate timeouts
6. Monitor network latency
7. Use local package mirrors
8. Implement rate limiting
9. Configure firewall rules
10. Use encrypted connections (SSH/HTTPS)
```

---

## 📝 Chapter Summary

In this chapter, you learned:
- ✅ Ansible uses a control node to manage multiple managed nodes
- ✅ Agentless architecture eliminates the need for installed agents
- ✅ Inventory defines and organizes managed hosts
- ✅ Modules are the units of work Ansible executes
- ✅ Playbooks orchestrate multiple tasks
- ✅ SSH (Linux) and WinRM (Windows) enable remote execution
- ✅ Ansible executes tasks in parallel for efficiency
- ✅ Connection persistence optimizes performance

---

## 🧪 Hands-On Exercise

**Exercise 1: Architecture Diagram**

Draw an architecture diagram for this scenario:
- 1 Control node (your laptop)
- 3 Web servers (Linux)
- 2 Database servers (Linux)
- 1 Windows application server

Include:
- Connection types
- Inventory groups
- Communication protocols

**Exercise 2: Connection Testing**

```bash
# Test different connection methods
# (Requires access to test systems)

# 1. Standard SSH connection
ansible testhost -m ping

# 2. Verbose SSH debugging
ansible testhost -m ping -vvv

# 3. Different user
ansible testhost -m ping -u admin

# 4. With password prompt
ansible testhost -m ping --ask-pass
```

---

## 🤔 Review Questions

1. What are the main components of Ansible architecture?
2. Why is Ansible called "agentless"?
3. What are the advantages and disadvantages of agentless architecture?
4. How does Ansible communicate with Linux vs Windows systems?
5. What happens when Ansible executes a task on a remote host?
6. How does parallel execution work in Ansible?
7. What is SSH multiplexing and why is it useful?
8. What is the role of the inventory in Ansible architecture?

---

## 📚 Additional Resources

- [Ansible Architecture Documentation](https://docs.ansible.com/ansible/latest/dev_guide/overview_architecture.html)
- [SSH Best Practices for Ansible](https://docs.ansible.com/ansible/latest/user_guide/connection_details.html)
- [Windows Remote Management](https://docs.ansible.com/ansible/latest/user_guide/windows_winrm.html)
- [Connection Plugins](https://docs.ansible.com/ansible/latest/plugins/connection.html)

---

[← Previous: What is Ansible?](./01-what-is-ansible.md) | [Next: Setting up Ansible →](./03-setting-up-ansible.md)
