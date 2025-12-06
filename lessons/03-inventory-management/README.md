# Lesson 03: Inventory Management

## Lesson Objectives
By the end of this lesson, you will be able to:
- Create and organize inventory files
- Use different inventory formats (INI, YAML)
- Work with groups and group variables
- Create dynamic inventories
- Use inventory patterns effectively

## Prerequisites
- Completed Lesson 02: Ansible Basics and Ad-Hoc Commands
- Understanding of YAML and INI formats

## Duration
60 minutes

## Lesson Content

### What is an Inventory?

An inventory defines the hosts and groups that Ansible manages. It's your infrastructure-as-code catalog.

### Inventory Locations

```bash
# Default locations (checked in order)
/etc/ansible/hosts
./inventory
./hosts

# Specify with -i flag
ansible all -i production.ini -m ping
ansible all -i inventory/ -m ping
```

### INI Format Inventory

#### Basic INI Inventory

```ini
# filepath: examples/inventory-basic.ini
# Individual hosts
web01.example.com
web02.example.com
db01.example.com

# Hosts with connection parameters
web03.example.com ansible_host=192.168.1.13 ansible_user=ubuntu
```

#### Groups in INI

```ini
# filepath: examples/inventory-groups.ini
# Web servers group
[webservers]
web01.example.com
web02.example.com
web03.example.com

# Database servers group
[databases]
db01.example.com
db02.example.com

# Application servers
[appservers]
app01.example.com
app02.example.com

# Group of groups
[production:children]
webservers
databases
appservers

# Development environment
[development]
dev01.example.com
dev02.example.com
```

#### Host Variables in INI

```ini
# filepath: examples/inventory-hostvars.ini
[webservers]
web01.example.com ansible_host=192.168.1.11 http_port=8080
web02.example.com ansible_host=192.168.1.12 http_port=8081

[databases]
db01.example.com ansible_host=192.168.1.21 mysql_port=3306
db02.example.com ansible_host=192.168.1.22 mysql_port=3307
```

#### Group Variables in INI

```ini
# filepath: examples/inventory-groupvars.ini
[webservers]
web01.example.com
web02.example.com

[webservers:vars]
ansible_user=ubuntu
ansible_become=yes
http_port=80
ssl_port=443

[databases]
db01.example.com
db02.example.com

[databases:vars]
ansible_user=dbadmin
ansible_python_interpreter=/usr/bin/python3
```

### YAML Format Inventory

#### Basic YAML Inventory

```yaml
# filepath: examples/inventory-basic.yml
---
all:
  hosts:
    web01.example.com:
    web02.example.com:
    db01.example.com:
```

#### Groups in YAML

```yaml
# filepath: examples/inventory-groups.yml
---
all:
  children:
    webservers:
      hosts:
        web01.example.com:
        web02.example.com:
        web03.example.com:
    
    databases:
      hosts:
        db01.example.com:
        db02.example.com:
    
    appservers:
      hosts:
        app01.example.com:
        app02.example.com:
    
    production:
      children:
        webservers:
        databases:
        appservers:
    
    development:
      hosts:
        dev01.example.com:
        dev02.example.com:
```

#### Host and Group Variables in YAML

```yaml
# filepath: examples/inventory-vars.yml
---
all:
  children:
    webservers:
      hosts:
        web01.example.com:
          ansible_host: 192.168.1.11
          http_port: 8080
          max_connections: 100
        
        web02.example.com:
          ansible_host: 192.168.1.12
          http_port: 8081
          max_connections: 150
      
      vars:
        ansible_user: ubuntu
        ansible_become: yes
        document_root: /var/www/html
    
    databases:
      hosts:
        db01.example.com:
          ansible_host: 192.168.1.21
          mysql_port: 3306
        
        db02.example.com:
          ansible_host: 192.168.1.22
          mysql_port: 3307
      
      vars:
        ansible_user: dbadmin
        ansible_python_interpreter: /usr/bin/python3
        max_connections: 500
```

### Advanced Inventory Features

#### Ranges and Patterns

```ini
# filepath: examples/inventory-ranges.ini
# Numeric ranges
[webservers]
web[01:05].example.com

# Alphabetic ranges
[databases]
db-[a:d].example.com

# Multiple patterns
[servers]
server-[a:c]-[1:3].example.com
# Expands to: server-a-1, server-a-2, server-a-3, server-b-1, etc.
```

```yaml
# filepath: examples/inventory-ranges.yml
---
all:
  children:
    webservers:
      hosts:
        web[01:05].example.com:
    
    databases:
      hosts:
        db-[a:d].example.com:
```

#### Connection Variables

```yaml
# filepath: examples/inventory-connection.yml
---
all:
  children:
    cloud_servers:
      hosts:
        aws-web01:
          ansible_host: ec2-54-123-45-67.compute.amazonaws.com
          ansible_user: ec2-user
          ansible_ssh_private_key_file: ~/.ssh/aws-key.pem
          ansible_port: 22
        
        azure-web01:
          ansible_host: 40.123.45.67
          ansible_user: azureuser
          ansible_ssh_private_key_file: ~/.ssh/azure-key.pem
    
    local_servers:
      hosts:
        localhost:
          ansible_connection: local
          ansible_python_interpreter: /usr/bin/python3
    
    windows_servers:
      hosts:
        win01:
          ansible_host: 192.168.1.100
          ansible_connection: winrm
          ansible_user: Administrator
          ansible_password: "{{ vault_win_password }}"
          ansible_winrm_transport: ntlm
```

### Inventory Directory Structure

Best practice is to organize inventory as a directory:
```
inventory/
├── production/
│   ├── hosts.yml
│   ├── group_vars/
│   │   ├── all.yml
│   │   ├── webservers.yml
│   │   └── databases.yml
│   └── host_vars/
│       ├── web01.example.com.yml
│       └── db01.example.com.yml
│
└── staging/
    ├── hosts.yml
    ├── group_vars/
    │   ├── all.yml
    │   └── webservers.yml
    └── host_vars/
        └── staging-web01.yml
```

#### Example Directory Structure

```yaml
# filepath: examples/inventory-dir/production/hosts.yml
---
all:
  children:
    webservers:
      hosts:
        web01.example.com:
        web02.example.com:
    
    databases:
      hosts:
        db01.example.com:
```

```yaml
# filepath: examples/inventory-dir/production/group_vars/all.yml
---
ansible_user: ubuntu
ansible_become: yes
ntp_server: pool.ntp.org
environment: production
```

```yaml
# filepath: examples/inventory-dir/production/group_vars/webservers.yml
---
http_port: 80
ssl_port: 443
document_root: /var/www/html
max_clients: 100
```

```yaml
# filepath: examples/inventory-dir/production/host_vars/web01.example.com.yml
---
ansible_host: 192.168.1.11
max_clients: 150
ssl_enabled: true
```

### Host and Group Patterns

```bash
# All hosts
ansible all -m ping

# Single host
ansible web01 -m ping

# Multiple hosts
ansible web01,web02,db01 -m ping

# Group
ansible webservers -m ping

# Multiple groups (union)
ansible webservers,databases -m ping

# Intersection (hosts in both groups)
ansible 'webservers:&production' -m ping

# Exclusion (in webservers but not in staging)
ansible 'webservers:!staging' -m ping

# Complex patterns
ansible 'webservers:&production:!maintenance' -m ping

# Wildcards
ansible 'web*.example.com' -m ping
ansible 'web*' -m ping

# Regex (prefix with ~)
ansible '~web[0-9]+' -m ping
```

### Dynamic Inventory

Dynamic inventory pulls data from external sources.

#### Custom Script Example

```python
#!/usr/bin/env python3
# filepath: examples/dynamic_inventory.py

import json
import sys

def get_inventory():
    inventory = {
        'webservers': {
            'hosts': ['web01', 'web02'],
            'vars': {
                'http_port': 80,
                'ansible_user': 'ubuntu'
            }
        },
        'databases': {
            'hosts': ['db01', 'db02'],
            'vars': {
                'mysql_port': 3306,
                'ansible_user': 'dbadmin'
            }
        },
        '_meta': {
            'hostvars': {
                'web01': {
                    'ansible_host': '192.168.1.11'
                },
                'web02': {
                    'ansible_host': '192.168.1.12'
                },
                'db01': {
                    'ansible_host': '192.168.1.21'
                },
                'db02': {
                    'ansible_host': '192.168.1.22'
                }
            }
        }
    }
    return inventory

def get_host(hostname):
    inventory = get_inventory()
    return inventory['_meta']['hostvars'].get(hostname, {})

if __name__ == '__main__':
    if len(sys.argv) == 2 and sys.argv[1] == '--list':
        print(json.dumps(get_inventory(), indent=2))
    elif len(sys.argv) == 3 and sys.argv[1] == '--host':
        print(json.dumps(get_host(sys.argv[2]), indent=2))
    else:
        print(json.dumps({}))
```

```bash
# Make executable
chmod +x dynamic_inventory.py

# Test it
./dynamic_inventory.py --list
ansible all -i dynamic_inventory.py -m ping
```

#### Using Cloud Provider Plugins

```yaml
# filepath: examples/aws_ec2.yml
---
plugin: amazon.aws.aws_ec2
regions:
  - us-east-1
  - us-west-2

filters:
  tag:Environment: production
  instance-state-name: running

keyed_groups:
  - key: tags.Role
    prefix: role
  - key: placement.region
    prefix: region

hostnames:
  - tag:Name
  - dns-name
  - private-ip-address
```

```bash
# Use AWS dynamic inventory
ansible-inventory -i aws_ec2.yml --graph
ansible all -i aws_ec2.yml -m ping
```

### Inventory Commands

```bash
# List all hosts
ansible-inventory --list

# List hosts in JSON format
ansible-inventory --list -i inventory.yml --output json

# Show inventory graph
ansible-inventory --graph

# Show specific host variables
ansible-inventory --host web01 -i inventory.yml

# List hosts in a group
ansible-inventory --graph webservers
```

## Hands-On Lab

### Lab 1: Create a Multi-Environment Inventory

Create the following structure:

```bash
mkdir -p inventory/{production,staging}/{group_vars,host_vars}
```

1. **Production Inventory**

```yaml
# filepath: lab/inventory/production/hosts.yml
---
all:
  children:
    webservers:
      hosts:
        prod-web01:
          ansible_host: 192.168.1.11
        prod-web02:
          ansible_host: 192.168.1.12
    
    databases:
      hosts:
        prod-db01:
          ansible_host: 192.168.1.21
```

```yaml
# filepath: lab/inventory/production/group_vars/all.yml
---
environment: production
ansible_user: ubuntu
ntp_server: pool.ntp.org
log_level: warning
```

```yaml
# filepath: lab/inventory/production/group_vars/webservers.yml
---
http_port: 80
ssl_port: 443
max_connections: 200
```

2. **Staging Inventory**

```yaml
# filepath: lab/inventory/staging/hosts.yml
---
all:
  children:
    webservers:
      hosts:
        staging-web01:
          ansible_host: 192.168.2.11
```

```yaml
# filepath: lab/inventory/staging/group_vars/all.yml
---
environment: staging
ansible_user: ubuntu
ntp_server: pool.ntp.org
log_level: debug
```

3. Test your inventories:

```bash
# List production hosts
ansible-inventory -i inventory/production --graph

# List staging hosts
ansible-inventory -i inventory/staging --graph

# Check variables for specific host
ansible-inventory -i inventory/production --host prod-web01

# Ping all production servers
ansible all -i inventory/production -m ping
```

### Lab 2: Use Inventory Patterns

```bash
# Target specific patterns
ansible 'prod-web*' -i inventory/production -m ping
ansible 'webservers:&production' -i inventory/production -m ping
ansible 'all:!databases' -i inventory/production -m ping

# Limit execution
ansible all -i inventory/production -m ping --limit webservers
ansible all -i inventory/production -m ping --limit prod-web01
```

### Lab 3: Create a Simple Dynamic Inventory

Create a Python script that generates inventory from a CSV file:

```csv
# filepath: lab/servers.csv
hostname,ip,group,role
web01,192.168.1.11,webservers,frontend
web02,192.168.1.12,webservers,frontend
db01,192.168.1.21,databases,backend
```

```python
#!/usr/bin/env python3
# filepath: lab/csv_inventory.py

import csv
import json
import sys

def read_csv(filename='servers.csv'):
    inventory = {'_meta': {'hostvars': {}}}
    
    with open(filename, 'r') as f:
        reader = csv.DictReader(f)
        for row in reader:
            hostname = row['hostname']
            group = row['group']
            
            # Add host to group
            if group not in inventory:
                inventory[group] = {'hosts': []}
            inventory[group]['hosts'].append(hostname)
            
            # Add host variables
            inventory['_meta']['hostvars'][hostname] = {
                'ansible_host': row['ip'],
                'role': row['role']
            }
    
    return inventory

if __name__ == '__main__':
    if len(sys.argv) == 2 and sys.argv[1] == '--list':
        print(json.dumps(read_csv(), indent=2))
    elif len(sys.argv) == 3 and sys.argv[1] == '--host':
        inventory = read_csv()
        print(json.dumps(inventory['_meta']['hostvars'].get(sys.argv[2], {})))
    else:
        print(json.dumps({}))
```

Test it:
```bash
chmod +x csv_inventory.py
./csv_inventory.py --list
ansible all -i csv_inventory.py -m ping
```

## Summary

In this lesson, you learned:
- ✅ Creating inventory files in INI and YAML formats
- ✅ Organizing hosts into groups
- ✅ Using host and group variables
- ✅ Inventory directory structures
- ✅ Host patterns and targeting
- ✅ Dynamic inventory concepts

## Additional Resources

- [Ansible Inventory Documentation](https://docs.ansible.com/ansible/latest/user_guide/intro_inventory.html)
- [Inventory Plugins](https://docs.ansible.com/ansible/latest/plugins/inventory.html)
- [Patterns Documentation](https://docs.ansible.com/ansible/latest/user_guide/intro_patterns.html)

## Next Steps

Proceed to [Lesson 04: Playbooks Fundamentals](../04-playbooks-fundamentals/README.md) to learn how to write Ansible playbooks.

