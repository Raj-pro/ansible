# Example Playbooks Collection

This directory contains practical, ready-to-use Ansible playbooks referenced throughout the book.

## 📁 Directory Structure

```
examples/
├── 01-basic/                    # Basic playbooks
├── 02-web-servers/              # Web server configurations
├── 03-databases/                # Database setups
├── 04-applications/             # Application deployments
├── 05-cloud/                    # Cloud provisioning
├── 06-security/                 # Security hardening
├── 07-monitoring/               # Monitoring setup
├── 08-ci-cd/                    # CI/CD examples
├── 09-containers/               # Container management
├── 10-network/                  # Network automation
└── 11-windows/                  # Windows automation
```

## 🚀 Quick Start

### Basic Examples

**Ping Test** (01-basic/ping.yml):
```yaml
---
- name: Test connectivity to all hosts
  hosts: all
  gather_facts: no
  
  tasks:
    - name: Ping all hosts
      ping:
    
    - name: Display message
      debug:
        msg: "Successfully connected to {{ inventory_hostname }}"
```

**System Update** (01-basic/update-system.yml):
```yaml
---
- name: Update system packages
  hosts: all
  become: yes
  
  tasks:
    - name: Update apt cache (Debian/Ubuntu)
      apt:
        update_cache: yes
        cache_valid_time: 3600
      when: ansible_os_family == "Debian"
    
    - name: Upgrade all packages (Debian/Ubuntu)
      apt:
        upgrade: dist
      when: ansible_os_family == "Debian"
    
    - name: Update yum packages (RHEL/CentOS)
      yum:
        name: "*"
        state: latest
      when: ansible_os_family == "RedHat"
```

### Web Server Setup

**Nginx Installation** (02-web-servers/nginx-basic.yml):
```yaml
---
- name: Install and configure Nginx
  hosts: webservers
  become: yes
  
  vars:
    http_port: 80
    server_name: example.com
  
  tasks:
    - name: Install Nginx
      apt:
        name: nginx
        state: present
        update_cache: yes
      when: ansible_os_family == "Debian"
    
    - name: Start and enable Nginx
      service:
        name: nginx
        state: started
        enabled: yes
    
    - name: Copy index.html
      copy:
        content: |
          <html>
          <head><title>Welcome</title></head>
          <body><h1>Hello from Ansible!</h1></body>
          </html>
        dest: /var/www/html/index.html
        mode: '0644'
    
    - name: Ensure Nginx is running
      service:
        name: nginx
        state: started
```

**Advanced Nginx with Template** (02-web-servers/nginx-advanced.yml):
```yaml
---
- name: Configure Nginx with custom settings
  hosts: webservers
  become: yes
  
  vars:
    nginx_user: www-data
    nginx_worker_processes: auto
    nginx_worker_connections: 1024
    server_names:
      - example.com
      - www.example.com
  
  tasks:
    - name: Install Nginx
      apt:
        name: nginx
        state: present
      
    - name: Configure Nginx
      template:
        src: nginx.conf.j2
        dest: /etc/nginx/nginx.conf
        validate: 'nginx -t -c %s'
      notify: Restart Nginx
    
    - name: Configure site
      template:
        src: site.conf.j2
        dest: /etc/nginx/sites-available/default
      notify: Reload Nginx
  
  handlers:
    - name: Restart Nginx
      service:
        name: nginx
        state: restarted
    
    - name: Reload Nginx
      service:
        name: nginx
        state: reloaded
```

### Database Setup

**MySQL Installation** (03-databases/mysql-setup.yml):
```yaml
---
- name: Install and configure MySQL
  hosts: databases
  become: yes
  
  vars:
    mysql_root_password: "{{ vault_mysql_root_password }}"
    mysql_databases:
      - name: appdb
        encoding: utf8mb4
        collation: utf8mb4_unicode_ci
    mysql_users:
      - name: appuser
        password: "{{ vault_mysql_appuser_password }}"
        priv: "appdb.*:ALL"
        host: "%"
  
  tasks:
    - name: Install MySQL
      apt:
        name:
          - mysql-server
          - mysql-client
          - python3-pymysql
        state: present
    
    - name: Start MySQL service
      service:
        name: mysql
        state: started
        enabled: yes
    
    - name: Set MySQL root password
      mysql_user:
        name: root
        password: "{{ mysql_root_password }}"
        login_unix_socket: /var/run/mysqld/mysqld.sock
        state: present
    
    - name: Create databases
      mysql_db:
        name: "{{ item.name }}"
        encoding: "{{ item.encoding }}"
        collation: "{{ item.collation }}"
        state: present
        login_user: root
        login_password: "{{ mysql_root_password }}"
      loop: "{{ mysql_databases }}"
    
    - name: Create MySQL users
      mysql_user:
        name: "{{ item.name }}"
        password: "{{ item.password }}"
        priv: "{{ item.priv }}"
        host: "{{ item.host }}"
        state: present
        login_user: root
        login_password: "{{ mysql_root_password }}"
      loop: "{{ mysql_users }}"
```

### Application Deployment

**Node.js Application** (04-applications/nodejs-app.yml):
```yaml
---
- name: Deploy Node.js application
  hosts: appservers
  become: yes
  
  vars:
    app_name: myapp
    app_user: nodejs
    app_directory: /opt/{{ app_name }}
    node_version: "18.x"
  
  tasks:
    - name: Install Node.js
      apt:
        name:
          - nodejs
          - npm
        state: present
    
    - name: Create application user
      user:
        name: "{{ app_user }}"
        system: yes
        create_home: no
    
    - name: Create application directory
      file:
        path: "{{ app_directory }}"
        state: directory
        owner: "{{ app_user }}"
        group: "{{ app_user }}"
    
    - name: Clone application repository
      git:
        repo: https://github.com/example/myapp.git
        dest: "{{ app_directory }}"
        version: main
      become_user: "{{ app_user }}"
    
    - name: Install dependencies
      npm:
        path: "{{ app_directory }}"
        state: present
      become_user: "{{ app_user }}"
    
    - name: Create systemd service
      template:
        src: nodejs-app.service.j2
        dest: /etc/systemd/system/{{ app_name }}.service
      notify: Restart app
    
    - name: Enable and start service
      systemd:
        name: "{{ app_name }}"
        enabled: yes
        state: started
        daemon_reload: yes
  
  handlers:
    - name: Restart app
      systemd:
        name: "{{ app_name }}"
        state: restarted
```

### Multi-Tier Application

**Complete LAMP Stack** (04-applications/lamp-stack.yml):
```yaml
---
- name: Configure Database Servers
  hosts: databases
  become: yes
  roles:
    - mysql
  tags: database

- name: Configure Web Servers
  hosts: webservers
  become: yes
  roles:
    - apache
    - php
  tags: webserver

- name: Deploy Application
  hosts: webservers
  become: yes
  
  tasks:
    - name: Clone application
      git:
        repo: https://github.com/example/webapp.git
        dest: /var/www/html
    
    - name: Set permissions
      file:
        path: /var/www/html
        owner: www-data
        group: www-data
        recurse: yes
    
    - name: Copy configuration
      template:
        src: config.php.j2
        dest: /var/www/html/config.php
      notify: Restart Apache
  
  handlers:
    - name: Restart Apache
      service:
        name: apache2
        state: restarted
  
  tags: deploy
```

### Cloud Provisioning

**AWS EC2 Instance** (05-cloud/aws-ec2.yml):
```yaml
---
- name: Provision AWS EC2 instances
  hosts: localhost
  connection: local
  gather_facts: no
  
  vars:
    region: us-east-1
    instance_type: t2.micro
    ami_id: ami-0c55b159cbfafe1f0  # Amazon Linux 2
    key_name: my-keypair
    security_group: web-sg
  
  tasks:
    - name: Create security group
      amazon.aws.ec2_group:
        name: "{{ security_group }}"
        description: Web server security group
        region: "{{ region }}"
        rules:
          - proto: tcp
            ports: 22
            cidr_ip: 0.0.0.0/0
          - proto: tcp
            ports: 80
            cidr_ip: 0.0.0.0/0
          - proto: tcp
            ports: 443
            cidr_ip: 0.0.0.0/0
    
    - name: Launch EC2 instances
      amazon.aws.ec2_instance:
        name: "webserver-{{ item }}"
        key_name: "{{ key_name }}"
        instance_type: "{{ instance_type }}"
        image_id: "{{ ami_id }}"
        region: "{{ region }}"
        security_group: "{{ security_group }}"
        wait: yes
        tags:
          Environment: Production
          Role: WebServer
      loop: "{{ range(1, 4) | list }}"
      register: ec2_instances
    
    - name: Add instances to inventory
      add_host:
        name: "{{ item.instances[0].public_ip_address }}"
        groups: webservers
        ansible_user: ec2-user
        ansible_ssh_private_key_file: ~/.ssh/{{ key_name }}.pem
      loop: "{{ ec2_instances.results }}"

- name: Configure EC2 instances
  hosts: webservers
  become: yes
  
  tasks:
    - name: Wait for SSH
      wait_for_connection:
        timeout: 300
    
    - name: Install web server
      yum:
        name: httpd
        state: present
    
    - name: Start web server
      service:
        name: httpd
        state: started
        enabled: yes
```

### Security Hardening

**System Hardening** (06-security/system-hardening.yml):
```yaml
---
- name: Harden Linux systems
  hosts: all
  become: yes
  
  tasks:
    - name: Update all packages
      apt:
        upgrade: dist
        update_cache: yes
      when: ansible_os_family == "Debian"
    
    - name: Disable root login
      lineinfile:
        path: /etc/ssh/sshd_config
        regexp: '^PermitRootLogin'
        line: 'PermitRootLogin no'
      notify: Restart SSH
    
    - name: Disable password authentication
      lineinfile:
        path: /etc/ssh/sshd_config
        regexp: '^PasswordAuthentication'
        line: 'PasswordAuthentication no'
      notify: Restart SSH
    
    - name: Configure firewall
      ufw:
        rule: "{{ item.rule }}"
        port: "{{ item.port }}"
        proto: "{{ item.proto }}"
      loop:
        - { rule: 'allow', port: '22', proto: 'tcp' }
        - { rule: 'allow', port: '80', proto: 'tcp' }
        - { rule: 'allow', port: '443', proto: 'tcp' }
    
    - name: Enable firewall
      ufw:
        state: enabled
    
    - name: Install fail2ban
      apt:
        name: fail2ban
        state: present
    
    - name: Start fail2ban
      service:
        name: fail2ban
        state: started
        enabled: yes
  
  handlers:
    - name: Restart SSH
      service:
        name: sshd
        state: restarted
```

### Docker Container Management

**Docker Setup** (09-containers/docker-setup.yml):
```yaml
---
- name: Install and configure Docker
  hosts: docker_hosts
  become: yes
  
  tasks:
    - name: Install prerequisites
      apt:
        name:
          - apt-transport-https
          - ca-certificates
          - curl
          - gnupg
          - lsb-release
        state: present
    
    - name: Add Docker GPG key
      apt_key:
        url: https://download.docker.com/linux/ubuntu/gpg
        state: present
    
    - name: Add Docker repository
      apt_repository:
        repo: "deb [arch=amd64] https://download.docker.com/linux/ubuntu {{ ansible_distribution_release }} stable"
        state: present
    
    - name: Install Docker
      apt:
        name:
          - docker-ce
          - docker-ce-cli
          - containerd.io
        state: present
        update_cache: yes
    
    - name: Start Docker service
      service:
        name: docker
        state: started
        enabled: yes
    
    - name: Add user to docker group
      user:
        name: "{{ ansible_user }}"
        groups: docker
        append: yes
    
    - name: Deploy container
      community.docker.docker_container:
        name: nginx
        image: nginx:latest
        state: started
        ports:
          - "80:80"
        restart_policy: always
```

## 📝 Usage Instructions

1. **Clone or copy examples**:
   ```bash
   cd ~/ansible-projects
   mkdir examples
   cd examples
   ```

2. **Update inventory**:
   Edit `inventory` file with your hosts

3. **Run playbook**:
   ```bash
   ansible-playbook 01-basic/ping.yml
   ```

4. **Use with variables**:
   ```bash
   ansible-playbook playbook.yml -e "http_port=8080"
   ```

## 🔒 Security Notes

- Always use Ansible Vault for sensitive data
- Never commit passwords or keys to version control
- Review and test playbooks in non-production first
- Use `--check` mode for dry runs

## 📚 Related Chapters

Each example directory corresponds to chapters in the book:
- 01-basic → Chapters 4-6
- 02-web-servers → Chapter 17
- 03-databases → Chapter 17
- 04-applications → Chapter 18
- 05-cloud → Chapter 16
- 06-security → Chapter 24
- 09-containers → Chapter 20

---

**Note**: These are working examples. Customize variables and configurations for your environment.
