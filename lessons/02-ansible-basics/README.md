# Lesson 02: Ansible Basics and Ad-Hoc Commands

## Lesson Objectives
By the end of this lesson, you will be able to:
- Understand Ansible architecture and core concepts
- Use ad-hoc commands effectively
- Work with different Ansible modules
- Understand when to use ad-hoc commands vs playbooks

## Prerequisites
- Completed Lesson 01: YAML Fundamentals
- Working Ansible installation
- Access to test nodes

## Duration
60 minutes

## Lesson Content

### Ansible Architecture

```
┌─────────────────┐
│  Control Node   │
│   (Your PC)     │
│                 │
│  - ansible-core │
│  - Collections  │
│  - Inventory    │
│  - Playbooks    │
└────────┬────────┘
         │
         │ SSH/WinRM
         │
    ┌────┴─────┬────────┬────────┐
    │          │        │        │
┌───▼───┐  ┌───▼───┐  ┌─▼────┐  ┌─▼────┐
│ Node1 │  │ Node2 │  │Node3 │  │Node4 │
│       │  │       │  │      │  │      │
│Python │  │Python │  │Python│  │Python│
└───────┘  └───────┘  └──────┘  └──────┘
```

### Key Concepts

1. **Control Node**: Machine where Ansible is installed
2. **Managed Nodes**: Servers managed by Ansible
3. **Inventory**: List of managed nodes
4. **Modules**: Units of code Ansible executes
5. **Tasks**: Units of action in Ansible
6. **Playbooks**: Ordered lists of tasks
7. **Collections**: Distribution format for Ansible content

### Ad-Hoc Commands

Ad-hoc commands are one-liners for quick tasks without writing a playbook.

#### Basic Syntax

```bash
ansible <host-pattern> -m <module> -a "<module arguments>"
```

#### Simple Examples

```bash
# Ping all hosts
ansible all -m ping

# Check uptime on webservers
ansible webservers -m command -a "uptime"

# Get disk space
ansible all -m command -a "df -h"

# Reboot databases (with sudo)
ansible databases -m reboot --become
```

### Common Modules for Ad-Hoc Commands

#### 1. ping Module

```bash
# Test connectivity
ansible all -m ping

# Test specific host
ansible web01 -m ping

# Test with verbose output
ansible all -m ping -v
```

#### 2. command Module

```bash
# Run simple commands (default module)
ansible all -a "hostname"
ansible all -m command -a "hostname"

# Check disk space
ansible all -a "df -h"

# View processes
ansible webservers -a "ps aux | grep nginx"
```

**Note**: `command` module doesn't support shell features like pipes, redirects, or variables.

#### 3. shell Module

```bash
# Use when you need shell features
ansible all -m shell -a "ps aux | grep nginx | wc -l"

# Use environment variables
ansible all -m shell -a "echo $HOME"

# Run complex commands
ansible webservers -m shell -a "uptime && free -m"
```

#### 4. copy Module

```bash
# Copy file to remote hosts
ansible webservers -m copy -a "src=/tmp/test.txt dest=/tmp/test.txt"

# Copy with permissions
ansible all -m copy -a "src=/tmp/script.sh dest=/usr/local/bin/script.sh mode=0755"

# Copy content inline
ansible all -m copy -a "content='Hello World\n' dest=/tmp/hello.txt"
```

#### 5. file Module

```bash
# Create directory
ansible all -m file -a "path=/tmp/mydir state=directory mode=0755"

# Create file
ansible all -m file -a "path=/tmp/myfile state=touch"

# Delete file
ansible all -m file -a "path=/tmp/myfile state=absent"

# Create symlink
ansible all -m file -a "src=/tmp/file dest=/tmp/link state=link"

# Change permissions
ansible all -m file -a "path=/tmp/file mode=0644 owner=root group=root"
```

#### 6. apt/yum/dnf Modules

```bash
# Install package (Debian/Ubuntu)
ansible webservers -m apt -a "name=nginx state=present" --become

# Install multiple packages
ansible webservers -m apt -a "name=nginx,vim,curl state=present" --become

# Update package
ansible webservers -m apt -a "name=nginx state=latest" --become

# Remove package
ansible webservers -m apt -a "name=nginx state=absent" --become

# Update cache
ansible webservers -m apt -a "update_cache=yes" --become

# For RHEL/CentOS
ansible webservers -m yum -a "name=httpd state=present" --become
```

#### 7. service/systemd Module

```bash
# Start service
ansible webservers -m service -a "name=nginx state=started" --become

# Stop service
ansible webservers -m service -a "name=nginx state=stopped" --become

# Restart service
ansible webservers -m service -a "name=nginx state=restarted" --become

# Enable service at boot
ansible webservers -m service -a "name=nginx enabled=yes" --become

# Using systemd module
ansible webservers -m systemd -a "name=nginx state=started daemon_reload=yes" --become
```

#### 8. user Module

```bash
# Create user
ansible all -m user -a "name=john state=present" --become

# Create user with home directory
ansible all -m user -a "name=john create_home=yes shell=/bin/bash" --become

# Add user to groups
ansible all -m user -a "name=john groups=sudo,docker append=yes" --become

# Remove user
ansible all -m user -a "name=john state=absent remove=yes" --become

# Set password (encrypted)
ansible all -m user -a "name=john password={{ 'mypassword' | password_hash('sha512') }}" --become
```

#### 9. setup Module

```bash
# Gather all facts
ansible all -m setup

# Filter specific facts
ansible all -m setup -a "filter=ansible_distribution*"

# Get network facts
ansible all -m setup -a "filter=ansible_default_ipv4"

# Get memory facts
ansible all -m setup -a "filter=ansible_memory_mb"
```

#### 10. git Module

```bash
# Clone repository
ansible webservers -m git -a "repo=https://github.com/user/repo.git dest=/var/www/app"

# Update repository
ansible webservers -m git -a "repo=https://github.com/user/repo.git dest=/var/www/app update=yes"

# Clone specific branch
ansible webservers -m git -a "repo=https://github.com/user/repo.git dest=/var/www/app version=develop"
```

### Command Options

#### Common Options

```bash
# Become (sudo)
ansible all -m apt -a "name=nginx state=present" --become
ansible all -m apt -a "name=nginx state=present" -b  # Short form

# Become user
ansible all -m command -a "whoami" --become --become-user=postgres

# Specify inventory
ansible all -i inventory.ini -m ping
ansible all -i inventory/ -m ping

# Limit to specific hosts
ansible all -m ping --limit webservers
ansible all -m ping --limit web01,web02

# Ask for sudo password
ansible all -m apt -a "name=nginx state=present" --become --ask-become-pass
ansible all -m apt -a "name=nginx state=present" -b -K  # Short form

# Check mode (dry run)
ansible all -m apt -a "name=nginx state=present" --check

# Verbose output
ansible all -m ping -v    # Verbose
ansible all -m ping -vv   # More verbose
ansible all -m ping -vvv  # Very verbose
ansible all -m ping -vvvv # Debug level

# Forks (parallel execution)
ansible all -m ping -f 10  # Run on 10 hosts at once
```

#### Connection Options

```bash
# Specify user
ansible all -m ping -u ubuntu

# Specify SSH key
ansible all -m ping --private-key ~/.ssh/my_key

# Specify SSH port
ansible all -m ping -e "ansible_port=2222"

# Use different connection type
ansible all -m ping -c local  # Local connection
```

### Practical Examples

#### System Administration

```bash
# Check system information
ansible all -m setup -a "filter=ansible_distribution*"

# Update all packages (Debian)
ansible all -m apt -a "upgrade=dist update_cache=yes" --become

# Check service status
ansible webservers -m service -a "name=nginx" --become

# Synchronize time
ansible all -m command -a "ntpdate pool.ntp.org" --become

# Clean package cache
ansible all -m apt -a "autoclean=yes" --become
```

#### File Management

```bash
# Backup configuration
ansible webservers -m fetch -a "src=/etc/nginx/nginx.conf dest=/tmp/backups/ flat=yes"

# Deploy configuration file
ansible webservers -m copy -a "src=nginx.conf dest=/etc/nginx/nginx.conf backup=yes" --become

# Set permissions recursively
ansible all -m file -a "path=/var/www state=directory recurse=yes owner=www-data group=www-data" --become
```

#### Security

```bash
# Change user password
ansible all -m user -a "name=admin password={{ 'newpass' | password_hash('sha512') }}" --become

# Add SSH key
ansible all -m authorized_key -a "user=ubuntu key='{{ lookup('file', '~/.ssh/id_rsa.pub') }}'" --become

# Update firewall rules (ufw)
ansible all -m ufw -a "rule=allow port=22 proto=tcp" --become
```

#### Application Management

```bash
# Restart application
ansible webservers -m systemd -a "name=myapp state=restarted" --become

# Clear application cache
ansible webservers -m file -a "path=/var/cache/myapp state=absent" --become
ansible webservers -m file -a "path=/var/cache/myapp state=directory" --become

# Pull latest code
ansible webservers -m git -a "repo=https://github.com/user/app.git dest=/var/www/app update=yes"
```

## Hands-On Lab

### Lab 1: Basic Ad-Hoc Commands

```bash
# 1. Test connectivity to all hosts
ansible all -m ping

# 2. Check hostname of all hosts
ansible all -a "hostname"

# 3. Get disk usage
ansible all -a "df -h"

# 4. Check memory
ansible all -m setup -a "filter=ansible_memory_mb"

# 5. List users
ansible all -a "cat /etc/passwd" | grep "/bin/bash"
```

### Lab 2: System Administration Tasks

```bash
# 1. Install vim on all hosts
ansible all -m apt -a "name=vim state=present" --become

# 2. Create a user
ansible all -m user -a "name=trainee shell=/bin/bash create_home=yes" --become

# 3. Create a directory
ansible all -m file -a "path=/opt/myapp state=directory mode=0755" --become

# 4. Copy a file
echo "Hello from Ansible" > /tmp/test.txt
ansible all -m copy -a "src=/tmp/test.txt dest=/tmp/test.txt" --become

# 5. Check service status
ansible all -m service -a "name=ssh" --become
```

### Lab 3: Real-World Scenario

Deploy a simple web server:

```bash
# 1. Install nginx
ansible webservers -m apt -a "name=nginx state=present" --become

# 2. Start and enable nginx
ansible webservers -m service -a "name=nginx state=started enabled=yes" --become

# 3. Create web directory
ansible webservers -m file -a "path=/var/www/mysite state=directory mode=0755" --become

# 4. Deploy index page
ansible webservers -m copy -a "content='<h1>Hello from Ansible</h1>' dest=/var/www/mysite/index.html" --become

# 5. Configure nginx (simplified)
ansible webservers -m copy -a "src=mysite.conf dest=/etc/nginx/sites-available/mysite" --become

# 6. Enable site
ansible webservers -m file -a "src=/etc/nginx/sites-available/mysite dest=/etc/nginx/sites-enabled/mysite state=link" --become

# 7. Reload nginx
ansible webservers -m service -a "name=nginx state=reloaded" --become

# 8. Verify
ansible webservers -m uri -a "url=http://localhost return_content=yes"
```

## When to Use Ad-Hoc Commands vs Playbooks

### Use Ad-Hoc Commands For:
- ✅ Quick one-time tasks
- ✅ Testing and debugging
- ✅ Urgent fixes
- ✅ Information gathering
- ✅ Simple repetitive tasks

### Use Playbooks For:
- ✅ Complex multi-step processes
- ✅ Repeatable deployments
- ✅ Configuration management
- ✅ Orchestration
- ✅ Documentation and version control

## Summary

In this lesson, you learned:
- ✅ Ansible architecture and core concepts
- ✅ How to use ad-hoc commands
- ✅ Common Ansible modules
- ✅ Command-line options
- ✅ Practical system administration tasks

## Additional Resources

- [Ansible Ad-Hoc Commands](https://docs.ansible.com/ansible/latest/user_guide/intro_adhoc.html)
- [Ansible Module Index](https://docs.ansible.com/ansible/latest/collections/index_module.html)
- [Ansible Command Line Tools](https://docs.ansible.com/ansible/latest/cli/ansible.html)

## Next Steps

Proceed to [Lesson 03: Inventory Management](../03-inventory-management/README.md) to learn how to organize your managed nodes.

