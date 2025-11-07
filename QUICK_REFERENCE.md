# Ansible Quick Reference Guide

## 📑 Common Commands

### Ad-Hoc Commands
```bash
# Ping all hosts
ansible all -m ping

# Run command
ansible all -m command -a "uptime"

# Copy file
ansible all -m copy -a "src=/local/file dest=/remote/file"

# Install package
ansible all -m apt -a "name=nginx state=present" --become

# Restart service
ansible all -m service -a "name=nginx state=restarted" --become

# Gather facts
ansible all -m setup
```

### Playbook Commands
```bash
# Run playbook
ansible-playbook playbook.yml

# Check syntax
ansible-playbook --syntax-check playbook.yml

# Dry run (check mode)
ansible-playbook --check playbook.yml

# Show differences
ansible-playbook --check --diff playbook.yml

# Limit to specific hosts
ansible-playbook playbook.yml --limit webservers

# Start at specific task
ansible-playbook playbook.yml --start-at-task="Install nginx"

# Use specific inventory
ansible-playbook -i inventory/production playbook.yml

# Verbose output
ansible-playbook playbook.yml -vvv

# Ask for sudo password
ansible-playbook playbook.yml --ask-become-pass

# Use vault password
ansible-playbook playbook.yml --ask-vault-pass
```

### Inventory Commands
```bash
# List hosts
ansible all --list-hosts

# Show inventory graph
ansible-inventory --graph

# View inventory as JSON
ansible-inventory --list

# Show host variables
ansible-inventory --host hostname
```

### Vault Commands
```bash
# Create encrypted file
ansible-vault create secret.yml

# Encrypt existing file
ansible-vault encrypt file.yml

# Decrypt file
ansible-vault decrypt file.yml

# Edit encrypted file
ansible-vault edit secret.yml

# View encrypted file
ansible-vault view secret.yml

# Rekey (change password)
ansible-vault rekey secret.yml

# Encrypt string
ansible-vault encrypt_string 'secret_value' --name 'variable_name'
```

### Galaxy Commands
```bash
# Install role
ansible-galaxy install username.rolename

# Install requirements
ansible-galaxy install -r requirements.yml

# List installed roles
ansible-galaxy list

# Remove role
ansible-galaxy remove username.rolename

# Initialize new role
ansible-galaxy init myrole

# Install collection
ansible-galaxy collection install community.general

# List collections
ansible-galaxy collection list
```

### Configuration Commands
```bash
# Show configuration
ansible-config view

# List all settings
ansible-config list

# Show current config
ansible-config dump

# Show only changed settings
ansible-config dump --only-changed
```

---

## 📝 Common Modules

### System Modules
```yaml
# Package management
apt:              # Debian/Ubuntu packages
yum:              # RHEL/CentOS packages
dnf:              # Fedora packages
package:          # Generic package manager

# Service management
service:          # Manage services
systemd:          # Systemd services

# User management
user:             # Manage users
group:            # Manage groups

# System
setup:            # Gather facts
reboot:           # Reboot system
hostname:         # Set hostname
timezone:         # Set timezone
```

### Files Modules
```yaml
copy:             # Copy files
template:         # Process Jinja2 templates
file:             # Set file attributes
lineinfile:       # Modify single line
blockinfile:      # Modify block of text
replace:          # Replace text with regex
fetch:            # Fetch files from remote
synchronize:      # rsync wrapper
```

### Commands Modules
```yaml
command:          # Execute commands (no shell)
shell:            # Execute commands (with shell)
raw:              # Execute raw commands (no Python)
script:           # Execute local script on remote
```

### Database Modules
```yaml
mysql_db:         # MySQL database
mysql_user:       # MySQL user
postgresql_db:    # PostgreSQL database
postgresql_user:  # PostgreSQL user
```

### Cloud Modules
```yaml
# AWS
ec2:              # EC2 instances
s3:               # S3 operations
rds:              # RDS databases

# Azure
azure_rm_virtualmachine:
azure_rm_storageaccount:

# GCP
gcp_compute_instance:
gcp_storage_bucket:
```

### Network Modules
```yaml
ios_command:      # Cisco IOS commands
ios_config:       # Cisco IOS configuration
junos_command:    # Juniper commands
junos_config:     # Juniper configuration
```

### Windows Modules
```yaml
win_copy:         # Copy files
win_service:      # Manage services
win_package:      # Install packages
win_feature:      # Windows features
win_user:         # Manage users
win_command:      # Execute commands
win_shell:        # Execute PowerShell
```

---

## 🎨 Common Patterns

### Basic Playbook Structure
```yaml
---
- name: Playbook name
  hosts: target_hosts
  become: yes
  vars:
    variable_name: value
  
  tasks:
    - name: Task name
      module_name:
        parameter: value
```

### Variables
```yaml
# Define variables
vars:
  http_port: 80
  app_name: myapp

# Use variables
{{ variable_name }}
{{ http_port }}

# Facts
{{ ansible_hostname }}
{{ ansible_distribution }}

# Magic variables
{{ inventory_hostname }}
{{ groups['webservers'] }}
{{ hostvars[inventory_hostname] }}
```

### Conditionals
```yaml
# Simple when
- name: Install on Ubuntu only
  apt:
    name: nginx
  when: ansible_distribution == 'Ubuntu'

# Multiple conditions (AND)
when:
  - ansible_distribution == 'Ubuntu'
  - ansible_distribution_version == '20.04'

# OR condition
when: ansible_distribution == 'Ubuntu' or ansible_distribution == 'Debian'

# Defined/undefined
when: variable_name is defined
when: variable_name is not defined
```

### Loops
```yaml
# Simple loop
- name: Install packages
  apt:
    name: "{{ item }}"
  loop:
    - nginx
    - git
    - vim

# Loop over dictionary
- name: Create users
  user:
    name: "{{ item.name }}"
    groups: "{{ item.groups }}"
  loop:
    - { name: 'alice', groups: 'admin' }
    - { name: 'bob', groups: 'users' }

# Loop with conditional
- name: Install on Ubuntu
  apt:
    name: "{{ item }}"
  loop: "{{ packages }}"
  when: ansible_distribution == 'Ubuntu'
```

### Handlers
```yaml
# Define handler
handlers:
  - name: restart nginx
    service:
      name: nginx
      state: restarted

# Notify handler
tasks:
  - name: Copy config
    template:
      src: nginx.conf.j2
      dest: /etc/nginx/nginx.conf
    notify: restart nginx
```

### Blocks
```yaml
# Error handling
- block:
    - name: Risky task
      command: /usr/bin/risky_command
  rescue:
    - name: Handle error
      debug:
        msg: "Task failed, handling error"
  always:
    - name: Always run
      debug:
        msg: "This always runs"
```

### Templates (Jinja2)
```jinja2
{# Comment #}

{{ variable }}                    # Variable
{{ variable | default('value') }} # Default value
{{ variable | upper }}            # Filter

{% if condition %}               # Conditional
  content
{% elif other_condition %}
  other content
{% else %}
  default content
{% endif %}

{% for item in list %}           # Loop
  {{ item }}
{% endfor %}
```

---

## 🔧 Configuration Examples

### ansible.cfg
```ini
[defaults]
inventory = ./inventory
host_key_checking = False
timeout = 30
forks = 10
log_path = ./ansible.log
roles_path = ./roles
collections_path = ./collections

[ssh_connection]
ssh_args = -o ControlMaster=auto -o ControlPersist=60s
pipelining = True

[privilege_escalation]
become = True
become_method = sudo
become_user = root
become_ask_pass = False
```

### Inventory (INI)
```ini
[webservers]
web[1:3].example.com

[databases]
db1.example.com

[production:children]
webservers
databases

[webservers:vars]
http_port=80
```

### Requirements (roles)
```yaml
---
# requirements.yml
- name: geerlingguy.nginx
  version: 3.1.4

- name: geerlingguy.mysql
  version: 4.0.0
```

### Requirements (collections)
```yaml
---
# requirements.yml
collections:
  - name: community.general
    version: 5.5.0
  
  - name: ansible.posix
    version: 1.4.0
```

---

## 🎯 Best Practices Checklist

### Playbook Organization
- ✅ Use meaningful names for plays and tasks
- ✅ One task, one action
- ✅ Use tags for selective execution
- ✅ Keep playbooks idempotent
- ✅ Use roles for reusability
- ✅ Store in version control (Git)

### Variables
- ✅ Use descriptive variable names
- ✅ Define variables in appropriate scope
- ✅ Use group_vars and host_vars
- ✅ Encrypt sensitive data with Vault
- ✅ Document variable purposes

### Inventory
- ✅ Separate environments
- ✅ Use groups logically
- ✅ Dynamic inventory for cloud
- ✅ Keep inventories in version control

### Security
- ✅ Use SSH keys, not passwords
- ✅ Enable host key checking in production
- ✅ Use Ansible Vault for secrets
- ✅ Implement least privilege
- ✅ Rotate credentials regularly

### Performance
- ✅ Increase forks for parallelism
- ✅ Enable SSH pipelining
- ✅ Use fact caching
- ✅ Disable gather_facts when not needed
- ✅ Use async for long-running tasks

---

## 📊 Exit Codes

```
0  - Success
1  - Error
2  - Usage error
3  - Host unreachable
4  - Connection failed
5  - Task failed
```

---

## 🔗 Useful Links

- [Official Documentation](https://docs.ansible.com/)
- [Ansible Galaxy](https://galaxy.ansible.com/)
- [Module Index](https://docs.ansible.com/ansible/latest/modules/modules_by_category.html)
- [Best Practices](https://docs.ansible.com/ansible/latest/user_guide/playbooks_best_practices.html)
- [GitHub Repository](https://github.com/ansible/ansible)

---

**Quick Tip**: Bookmark this page for fast reference while writing playbooks!
