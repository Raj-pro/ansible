# Chapter 4: Inventory Management

## 📖 Introduction

The inventory is where you define which hosts Ansible will manage. This chapter covers everything from simple static inventories to complex dynamic setups.

## 🎯 Learning Objectives

- Create and manage static inventories
- Understand INI vs YAML formats
- Master host patterns and groups
- Implement inventory best practices
- Introduction to dynamic inventories

---

## 1. Understanding Inventory

### What is an Inventory?

An inventory is a list of managed nodes (hosts) that Ansible can connect to and manage. It can be:
- **Static**: Manually defined in a file
- **Dynamic**: Generated programmatically from external sources

### Inventory File Locations

```bash
# Default location
/etc/ansible/hosts

# Specify in ansible.cfg
[defaults]
inventory = ./inventory

# Command-line override
ansible -i /path/to/inventory all -m ping
```

---

## 2. Static Inventories

### 2.1 INI Format

**Basic Example**:
```ini
# Simple host list
server1.example.com
server2.example.com
192.168.1.100

# Hosts with connection details
web1 ansible_host=192.168.1.10 ansible_user=ubuntu
web2 ansible_host=192.168.1.11 ansible_user=ubuntu
```

**Host Groups**:
```ini
[webservers]
web1.example.com
web2.example.com
web3.example.com

[databases]
db1.example.com
db2.example.com

[loadbalancers]
lb1.example.com
```

**Nested Groups**:
```ini
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

# All servers in DC1
[dc1:children]
webservers
databases
loadbalancers
```

**Group Variables**:
```ini
[webservers]
web1.example.com
web2.example.com

[webservers:vars]
http_port=80
max_clients=200
ansible_user=webadmin
ansible_become=yes
```

**Host Variables**:
```ini
[databases]
db1.example.com db_role=master db_port=5432
db2.example.com db_role=slave db_port=5432
db3.example.com db_role=slave db_port=5432
```

**Range Patterns**:
```ini
# Numeric ranges
[webservers]
web[01:50].example.com

# Alphabetic ranges
[databases]
db-[a:f].example.com

# With leading zeros
[servers]
server[001:100].example.com
```

### 2.2 YAML Format

**Basic YAML Inventory**:
```yaml
all:
  hosts:
    server1.example.com:
    server2.example.com:
```

**With Groups**:
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

**With Variables**:
```yaml
all:
  children:
    webservers:
      hosts:
        web1.example.com:
          ansible_host: 192.168.1.10
          http_port: 8080
        web2.example.com:
          ansible_host: 192.168.1.11
          http_port: 8080
      vars:
        ansible_user: webadmin
        ansible_become: yes
        
    databases:
      hosts:
        db1.example.com:
          db_role: master
        db2.example.com:
          db_role: slave
      vars:
        ansible_user: dbadmin
        db_port: 5432
```

**Nested Groups in YAML**:
```yaml
all:
  children:
    production:
      children:
        webservers:
          hosts:
            web1.prod.example.com:
            web2.prod.example.com:
        databases:
          hosts:
            db1.prod.example.com:
    
    staging:
      children:
        webservers:
          hosts:
            web1.staging.example.com:
        databases:
          hosts:
            db1.staging.example.com:
```

---

## 3. Inventory Variables

### 3.1 Connection Variables

```yaml
# Connection type
ansible_connection: ssh        # ssh, winrm, local, docker, etc.

# SSH variables
ansible_host: 192.168.1.100   # IP or hostname to connect to
ansible_port: 22              # SSH port
ansible_user: ubuntu          # SSH username
ansible_ssh_private_key_file: ~/.ssh/id_rsa
ansible_ssh_common_args: '-o StrictHostKeyChecking=no'

# Become (privilege escalation)
ansible_become: yes
ansible_become_method: sudo   # sudo, su, pbrun, pfexec, etc.
ansible_become_user: root
ansible_become_password: "{{ vault_sudo_pass }}"

# Python interpreter
ansible_python_interpreter: /usr/bin/python3
```

### 3.2 Variable Precedence

Variables can be defined at multiple levels (lowest to highest precedence):

```
1. Command line values (e.g., -e "user=my_user")
2. Role defaults
3. Inventory file or script group vars
4. Inventory group_vars/all
5. Playbook group_vars/all
6. Inventory group_vars/*
7. Playbook group_vars/*
8. Inventory file or script host vars
9. Inventory host_vars/*
10. Playbook host_vars/*
11. Host facts / cached set_facts
12. Play vars
13. Play vars_prompt
14. Play vars_files
15. Role vars (defined in role/vars/main.yml)
16. Block vars (only for tasks in block)
17. Task vars (only for the task)
18. Include vars
19. Set_facts / registered vars
20. Role (and include_role) params
21. Include params
22. Extra vars (-e in command line)
```

---

## 4. Host Patterns

### 4.1 Basic Patterns

```bash
# All hosts
ansible all -m ping

# All hosts in a group
ansible webservers -m ping

# Specific host
ansible web1.example.com -m ping

# Multiple groups (OR)
ansible webservers:databases -m ping

# Intersection (AND)
ansible webservers:&production -m ping

# Exclusion (NOT)
ansible webservers:!staging -m ping

# Complex patterns
ansible webservers:&production:!web1.example.com -m ping
```

### 4.2 Advanced Patterns

```bash
# Wildcard patterns
ansible 'web*.example.com' -m ping
ansible '*.example.com' -m ping
ansible 'web[1:3].example.com' -m ping

# Regular expressions
ansible '~web\d+\.example\.com' -m ping

# Multiple hosts
ansible web1.example.com,web2.example.com,db1.example.com -m ping

# First N hosts
ansible webservers[0] -m ping           # First host
ansible webservers[0:2] -m ping         # First 3 hosts
ansible webservers[-1] -m ping          # Last host
```

---

## 5. Organizing Inventory

### 5.1 Directory Structure

**Recommended Structure**:
```
inventory/
├── production/
│   ├── hosts                    # Production inventory
│   ├── group_vars/
│   │   ├── all.yml             # Variables for all hosts
│   │   ├── webservers.yml      # Web server variables
│   │   └── databases.yml       # Database variables
│   └── host_vars/
│       ├── web1.example.com.yml
│       └── db1.example.com.yml
├── staging/
│   ├── hosts
│   ├── group_vars/
│   │   └── all.yml
│   └── host_vars/
└── development/
    ├── hosts
    └── group_vars/
        └── all.yml
```

### 5.2 Group Variables

**group_vars/all.yml**:
```yaml
---
# Variables for all hosts
ntp_server: time.example.com
dns_servers:
  - 8.8.8.8
  - 8.8.4.4

# Common packages
common_packages:
  - vim
  - git
  - htop
  - curl
```

**group_vars/webservers.yml**:
```yaml
---
# Web server specific variables
http_port: 80
https_port: 443
max_clients: 200

nginx_user: www-data
nginx_worker_processes: auto

document_root: /var/www/html
```

### 5.3 Host Variables

**host_vars/web1.example.com.yml**:
```yaml
---
# Specific to web1
ansible_host: 192.168.1.10
server_id: 1
backup_enabled: true
monitoring_enabled: true

# Host-specific overrides
max_clients: 300  # Override group variable
```

---

## 6. Dynamic Inventories

### 6.1 What is Dynamic Inventory?

Dynamic inventory scripts query external systems to get host information:
- Cloud providers (AWS, Azure, GCP)
- Virtualization platforms (VMware, OpenStack)
- CMDB systems
- Container orchestrators (Kubernetes, Docker)

### 6.2 Using Dynamic Inventory

**AWS EC2 Example**:
```bash
# Install AWS collection
ansible-galaxy collection install amazon.aws

# Use EC2 plugin
ansible-inventory -i aws_ec2.yml --graph
```

**aws_ec2.yml** (Inventory Plugin Config):
```yaml
---
plugin: amazon.aws.aws_ec2
regions:
  - us-east-1
  - us-west-2

keyed_groups:
  # Group by instance state
  - key: instance_state
    prefix: aws_state
  
  # Group by tags
  - key: tags.Environment
    prefix: env
  
  # Group by instance type
  - key: instance_type
    prefix: type

hostnames:
  - tag:Name
  - private-ip-address

filters:
  instance-state-name: running
```

### 6.3 Custom Dynamic Inventory Script

**Simple Python Script** (inventory.py):
```python
#!/usr/bin/env python3
import json

def get_inventory():
    return {
        "webservers": {
            "hosts": ["web1.example.com", "web2.example.com"],
            "vars": {
                "http_port": 80
            }
        },
        "databases": {
            "hosts": ["db1.example.com"],
            "vars": {
                "db_port": 5432
            }
        },
        "_meta": {
            "hostvars": {
                "web1.example.com": {
                    "ansible_host": "192.168.1.10"
                }
            }
        }
    }

if __name__ == "__main__":
    print(json.dumps(get_inventory(), indent=2))
```

**Usage**:
```bash
# Make executable
chmod +x inventory.py

# Test output
./inventory.py

# Use with ansible
ansible -i inventory.py all -m ping
```

---

## 7. Inventory Management Commands

### 7.1 Listing Hosts

```bash
# List all hosts
ansible all --list-hosts

# List hosts in a group
ansible webservers --list-hosts

# List with pattern
ansible 'web*' --list-hosts

# Count hosts
ansible all --list-hosts | wc -l
```

### 7.2 Viewing Inventory

```bash
# Show inventory as JSON
ansible-inventory --list

# Show inventory as YAML
ansible-inventory --list -y

# Show inventory graph
ansible-inventory --graph

# Graph for specific group
ansible-inventory --graph webservers

# Show host details
ansible-inventory --host web1.example.com
```

### 7.3 Testing Variables

```bash
# Debug variables for a host
ansible web1.example.com -m debug -a "var=hostvars[inventory_hostname]"

# Check specific variable
ansible webservers -m debug -a "var=http_port"

# List all variables
ansible-inventory --host web1.example.com
```

---

## 8. Best Practices

### ✅ Inventory Organization

```yaml
1. Separate environments (production, staging, development)
2. Use descriptive group names
3. Keep inventories in version control
4. Use group_vars and host_vars for variables
5. Don't hardcode secrets (use Ansible Vault)
6. Document your inventory structure
7. Use YAML for complex inventories
8. Implement naming conventions
9. Keep dynamic and static inventories separate
10. Regularly audit and clean up inventory
```

### ✅ Security Best Practices

```yaml
1. Never store passwords in plain text
2. Use Ansible Vault for sensitive data
3. Implement SSH key authentication
4. Rotate credentials regularly
5. Use separate credentials per environment
6. Limit access to inventory files
7. Encrypt inventory repositories
8. Audit inventory changes
9. Use jump hosts for isolated networks
10. Implement least privilege access
```

### ✅ Variable Management

```yaml
1. Follow variable precedence rules
2. Use clear variable names
3. Group related variables
4. Document variable purposes
5. Avoid variable collisions
6. Use defaults appropriately
7. Validate variables in playbooks
8. Keep variables DRY (Don't Repeat Yourself)
9. Use facts where possible
10. Version control all variable files
```

---

## 📝 Chapter Summary

✅ Static inventories use INI or YAML format  
✅ Groups organize hosts logically  
✅ Variables can be set at multiple levels  
✅ Host patterns enable flexible targeting  
✅ Dynamic inventories query external sources  
✅ Proper organization improves maintainability

---

## 🧪 Hands-On Exercise

**Create a Complete Inventory Structure**:

```bash
# 1. Create directory structure
mkdir -p inventory/production/{group_vars,host_vars}

# 2. Create production inventory
cat > inventory/production/hosts <<EOF
[webservers]
web[1:3].example.com

[databases]
db1.example.com

[production:children]
webservers
databases
EOF

# 3. Create group variables
cat > inventory/production/group_vars/all.yml <<EOF
---
ntp_server: time.example.com
EOF

cat > inventory/production/group_vars/webservers.yml <<EOF
---
http_port: 80
EOF

# 4. Test inventory
ansible-inventory -i inventory/production --graph
ansible-inventory -i inventory/production --list
```

---

## 🤔 Review Questions

1. What's the difference between static and dynamic inventory?
2. How do you target multiple groups with one command?
3. What is variable precedence?
4. How do you exclude hosts from a pattern?
5. Where should sensitive variables be stored?
6. What's the difference between group_vars and host_vars?
7. How do you test dynamic inventory scripts?

---

[← Previous: Setting up Ansible](../part-01-introduction/03-setting-up-ansible.md) | [Next: Ad-Hoc Commands →](./05-ad-hoc-commands.md)
