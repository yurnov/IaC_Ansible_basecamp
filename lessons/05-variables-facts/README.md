# Lesson 05: Variables and Facts

## Lesson Objectives
By the end of this lesson, you will be able to:
- Define and use variables in different scopes
- Work with facts and custom facts
- Use variable precedence effectively
- Implement variable files and vault
- Use magic variables and filters

## Prerequisites
- Completed Lesson 04: Playbooks Fundamentals
- Understanding of Jinja2 basics

## Duration
90 minutes

## Lesson Content

### What are Variables?

Variables store values that can be reused throughout your playbooks, making them flexible and maintainable.

### Variable Naming

```yaml
# filepath: examples/variable-naming.yml
---
# ✅ Good variable names
http_port: 80
app_name: myapp
db_user: admin
max_connections: 100
is_production: true

# ❌ Bad variable names
port: 80              # Too generic
n: myapp              # Not descriptive
DB-USER: admin        # Use underscores, not hyphens
MAX_CONNECTIONS: 100  # Lowercase preferred
prod: true            # Unclear abbreviation
```

### Defining Variables

#### 1. In Playbook (vars)

```yaml
# filepath: examples/vars-playbook.yml
---
- name: Variables in playbook
  hosts: webservers
  
  vars:
    http_port: 80
    ssl_port: 443
    server_name: www.example.com
    document_root: /var/www/html
  
  tasks:
    - name: Display configuration
      debug:
        msg: "Server {{ server_name }} on ports {{ http_port }}/{{ ssl_port }}"
```

#### 2. In Playbook (vars_files)

```yaml
# filepath: examples/vars-files.yml
---
- name: Variables from files
  hosts: webservers
  
  vars_files:
    - vars/common.yml
    - vars/webserver.yml
    - "vars/{{ ansible_distribution }}.yml"
  
  tasks:
    - name: Display app name
      debug:
        var: app_name
```

```yaml
# filepath: examples/vars/common.yml
---
app_name: myapp
app_version: 1.0.0
environment: production
```

```yaml
# filepath: examples/vars/webserver.yml
---
http_port: 80
ssl_port: 443
worker_processes: 4
```

#### 3. In Inventory

```yaml
# filepath: examples/inventory-vars.yml
---
all:
  children:
    webservers:
      hosts:
        web01:
          ansible_host: 192.168.1.11
          http_port: 8080
          worker_processes: 2
        web02:
          ansible_host: 192.168.1.12
          http_port: 8081
          worker_processes: 4
      
      vars:
        document_root: /var/www/html
        ssl_enabled: true
    
    databases:
      vars:
        db_port: 5432
        max_connections: 200
```

#### 4. In group_vars and host_vars

```yaml
# filepath: examples/group_vars/webservers.yml
---
http_port: 80
ssl_port: 443
document_root: /var/www/html
nginx_worker_processes: auto
nginx_worker_connections: 1024
```

```yaml
# filepath: examples/host_vars/web01.example.com.yml
---
ansible_host: 192.168.1.11
nginx_worker_processes: 4
custom_config: true
```

#### 5. Command Line (-e / --extra-vars)

```bash
# Single variable
ansible-playbook playbook.yml -e "version=1.2.3"

# Multiple variables
ansible-playbook playbook.yml -e "version=1.2.3 environment=production"

# JSON format
ansible-playbook playbook.yml -e '{"version":"1.2.3","env":"prod"}'

# From file
ansible-playbook playbook.yml -e "@vars.yml"
ansible-playbook playbook.yml -e "@vars.json"
```

### Variable Precedence

Variables can be defined in multiple places. Ansible uses this order (last one wins):

1. Command line values (-e)
2. Role defaults (lowest priority)
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
18. Include_vars
19. Set_facts / registered vars
20. Role (and include_role) params
21. Include params
22. Extra vars (-e) (highest priority)

```yaml
# filepath: examples/precedence.yml
---
- name: Variable precedence example
  hosts: webservers
  vars:
    my_var: "from_play"
  
  tasks:
    - name: Display variable from play
      debug:
        var: my_var
    
    - name: Override with task var
      debug:
        var: my_var
      vars:
        my_var: "from_task"
    
    - name: Set fact (high precedence)
      set_fact:
        my_var: "from_set_fact"
    
    - name: Display after set_fact
      debug:
        var: my_var
```

### Variable Types

#### Strings

```yaml
# filepath: examples/var-strings.yml
---
- name: String variables
  hosts: localhost
  connection: local
  
  vars:
    simple_string: hello
    quoted_string: "hello world"
    multiline_string: |
      This is a
      multiline string
    folded_string: >
      This is a long string
      that will be folded
      into a single line
  
  tasks:
    - debug: var=simple_string
    - debug: var=quoted_string
    - debug: var=multiline_string
    - debug: var=folded_string
```

#### Numbers

```yaml
# filepath: examples/var-numbers.yml
---
- name: Number variables
  hosts: localhost
  connection: local
  
  vars:
    port: 80
    timeout: 30
    pi: 3.14159
    percentage: 0.95
  
  tasks:
    - name: Math operations
      debug:
        msg: "{{ port + 100 }}"
    
    - name: Comparison
      debug:
        msg: "Port is standard"
      when: port == 80
```

#### Booleans

```yaml
# filepath: examples/var-booleans.yml
---
- name: Boolean variables
  hosts: localhost
  connection: local
  
  vars:
    is_production: true
    debug_mode: false
    ssl_enabled: yes
    monitoring_enabled: no
  
  tasks:
    - name: Conditional based on boolean
      debug:
        msg: "Running in production mode"
      when: is_production
    
    - name: Another conditional
      debug:
        msg: "SSL is enabled"
      when: ssl_enabled | bool
```

#### Lists

```yaml
# filepath: examples/var-lists.yml
---
- name: List variables
  hosts: localhost
  connection: local
  
  vars:
    packages:
      - nginx
      - vim
      - git
      - curl
    
    users:
      - name: alice
        shell: /bin/bash
      - name: bob
        shell: /bin/zsh
  
  tasks:
    - name: Display list
      debug:
        var: packages
    
    - name: Access list item by index
      debug:
        msg: "First package: {{ packages[0] }}"
    
    - name: Display list length
      debug:
        msg: "Number of packages: {{ packages | length }}"
```

#### Dictionaries

```yaml
# filepath: examples/var-dictionaries.yml
---
- name: Dictionary variables
  hosts: localhost
  connection: local
  
  vars:
    database:
      host: localhost
      port: 5432
      name: myapp
      user: dbuser
      password: secret123
    
    app_config:
      web:
        host: 0.0.0.0
        port: 8080
      database:
        pool_size: 10
        timeout: 30
  
  tasks:
    - name: Access dictionary values (dot notation)
      debug:
        msg: "Database: {{ database.name }} on {{ database.host }}"
    
    - name: Access dictionary values (bracket notation)
      debug:
        msg: "Port: {{ database['port'] }}"
    
    - name: Display nested dictionary
      debug:
        msg: "Web port: {{ app_config.web.port }}"
```

### Registered Variables

```yaml
# filepath: examples/registered-vars.yml
---
- name: Using registered variables
  hosts: webservers
  
  tasks:
    - name: Check if file exists
      stat:
        path: /etc/nginx/nginx.conf
      register: nginx_conf
    
    - name: Display registered variable
      debug:
        var: nginx_conf
    
    - name: Use registered variable in condition
      debug:
        msg: "Nginx config exists"
      when: nginx_conf.stat.exists
    
    - name: Run command and capture output
      command: hostname
      register: server_hostname
      changed_when: false
    
    - name: Display command output
      debug:
        msg: "Hostname is {{ server_hostname.stdout }}"
    
    - name: Register shell output
      shell: df -h / | tail -1 | awk '{print $5}'
      register: disk_usage
      changed_when: false
    
    - name: Check disk usage
      debug:
        msg: "Disk usage: {{ disk_usage.stdout }}"
      failed_when: disk_usage.stdout[:-1]|int > 90
```

### Facts

Facts are variables automatically discovered by Ansible about managed hosts.

#### Gathering Facts

```yaml
# filepath: examples/facts-gather.yml
---
- name: Gathering facts
  hosts: webservers
  gather_facts: yes  # Default
  
  tasks:
    - name: Display all facts
      debug:
        var: ansible_facts
    
    - name: Display specific facts
      debug:
        msg: |
          OS: {{ ansible_distribution }} {{ ansible_distribution_version }}
          Hostname: {{ ansible_hostname }}
          IP: {{ ansible_default_ipv4.address }}
          CPU cores: {{ ansible_processor_vcpus }}
          Memory: {{ ansible_memtotal_mb }} MB
```

#### Common Facts

```yaml
# filepath: examples/facts-common.yml
---
- name: Common facts
  hosts: webservers
  
  tasks:
    # System facts
    - debug: var=ansible_distribution         # Ubuntu, CentOS, etc.
    - debug: var=ansible_distribution_version # 20.04, 8, etc.
    - debug: var=ansible_os_family             # Debian, RedHat, etc.
    - debug: var=ansible_kernel                # Kernel version
    - debug: var=ansible_architecture          # x86_64, armv7l, etc.
    
    # Hardware facts
    - debug: var=ansible_processor_vcpus       # Number of vCPUs
    - debug: var=ansible_processor_cores       # Cores per socket
    - debug: var=ansible_memtotal_mb           # Total RAM in MB
    - debug: var=ansible_devices               # Disk devices
    
    # Network facts
    - debug: var=ansible_hostname              # Short hostname
    - debug: var=ansible_fqdn                  # Fully qualified domain name
    - debug: var=ansible_default_ipv4.address  # Primary IP address
    - debug: var=ansible_default_ipv4.gateway  # Default gateway
    - debug: var=ansible_all_ipv4_addresses    # All IPv4 addresses
    - debug: var=ansible_interfaces            # Network interfaces
    
    # Date/Time facts
    - debug: var=ansible_date_time.date        # Current date
    - debug: var=ansible_date_time.time        # Current time
    - debug: var=ansible_date_time.epoch       # Unix timestamp
```

#### Filtering Facts

```yaml
# filepath: examples/facts-filter.yml
---
- name: Filter facts during gathering
  hosts: webservers
  gather_facts: yes
  gather_subset:
    - '!all'           # Exclude all
    - '!min'           # Exclude minimal
    - network          # Include only network facts
    - hardware         # Include only hardware facts
  
  tasks:
    - name: Display filtered facts
      debug:
        var: ansible_facts
```

#### Custom Facts

```bash
# filepath: examples/setup-custom-facts.sh
# Create custom facts directory
sudo mkdir -p /etc/ansible/facts.d

# Create a custom fact file
cat <<EOF | sudo tee /etc/ansible/facts.d/app.fact
[general]
app_name=myapp
app_version=1.0.0
environment=production

[database]
host=db.example.com
port=5432
EOF

# Make it executable (for dynamic facts)
sudo chmod +x /etc/ansible/facts.d/app.fact
```

```yaml
# filepath: examples/facts-custom.yml
---
- name: Using custom facts
  hosts: webservers
  
  tasks:
    - name: Display custom facts
      debug:
        var: ansible_local
    
    - name: Use custom fact
      debug:
        msg: "App: {{ ansible_local.app.general.app_name }} v{{ ansible_local.app.general.app_version }}"
```

Dynamic custom fact (executable):

```python
#!/usr/bin/env python3
# filepath: /etc/ansible/facts.d/system_info.fact

import json
import subprocess

def get_system_info():
    return {
        "uptime": subprocess.getoutput("uptime"),
        "disk_usage": subprocess.getoutput("df -h /"),
        "load_average": subprocess.getoutput("cat /proc/loadavg").split()[0]
    }

if __name__ == '__main__':
    print(json.dumps(get_system_info()))
```

### Set Facts

```yaml
# filepath: examples/set-facts.yml
---
- name: Setting facts
  hosts: webservers
  
  tasks:
    - name: Set simple fact
      set_fact:
        deployment_time: "{{ ansible_date_time.iso8601 }}"
    
    - name: Set complex fact
      set_fact:
        app_info:
          name: myapp
          version: 1.0.0
          deployed_by: "{{ ansible_user_id }}"
          deployed_at: "{{ deployment_time }}"
    
    - name: Display facts
      debug:
        msg: "Deployed {{ app_info.name }} v{{ app_info.version }} at {{ app_info.deployed_at }}"
    
    - name: Set fact based on condition
      set_fact:
        environment_type: "{{ 'production' if inventory_hostname in groups['production'] else 'non-production' }}"
    
    - name: Facts persist across plays
      debug:
        var: app_info
```

### Variable Scoping

```yaml
# filepath: examples/var-scoping.yml
---
- name: Variable scoping
  hosts: webservers
  vars:
    play_var: "I'm at play scope"
  
  tasks:
    - name: Play variable
      debug:
        var: play_var
    
    - name: Task variable (task scope only)
      debug:
        msg: "Task var: {{ task_var }}"
      vars:
        task_var: "I'm at task scope"
    
    - name: Block variables
      block:
        - debug:
            msg: "Block var: {{ block_var }}"
        
        - debug:
            msg: "Still accessible: {{ block_var }}"
      
      vars:
        block_var: "I'm at block scope"
    
    - name: This will fail - block_var not accessible here
      debug:
        var: block_var
      ignore_errors: yes
```

### Magic Variables

```yaml
# filepath: examples/magic-vars.yml
---
- name: Magic variables
  hosts: webservers
  
  tasks:
    - name: Inventory information
      debug:
        msg: |
          Hostname: {{ inventory_hostname }}
          Short name: {{ inventory_hostname_short }}
          Groups: {{ group_names }}
          All groups: {{ groups }}
    
    - name: Playbook information
      debug:
        msg: |
          Playbook dir: {{ playbook_dir }}
          Role path: {{ role_path | default('N/A') }}
    
    - name: Ansible information
      debug:
        msg: |
          Ansible version: {{ ansible_version.full }}
          Python version: {{ ansible_python_version }}
    
    - name: Access other hosts' variables
      debug:
        msg: "web01 IP: {{ hostvars['web01']['ansible_default_ipv4']['address'] }}"
      run_once: true
    
    - name: Loop through all web servers
      debug:
        msg: "{{ item }} - {{ hostvars[item]['ansible_default_ipv4']['address'] }}"
      loop: "{{ groups['webservers'] }}"
      run_once: true
```

### Variable Prompts

```yaml
# filepath: examples/vars-prompt.yml
---
- name: Variable prompts
  hosts: webservers
  
  vars_prompt:
    - name: username
      prompt: "Enter username"
      private: no
    
    - name: password
      prompt: "Enter password"
      private: yes
      encrypt: sha512_crypt
      confirm: yes
    
    - name: environment
      prompt: "Enter environment (dev/staging/prod)"
      private: no
      default: "dev"
  
  tasks:
    - name: Display user input
      debug:
        msg: "User: {{ username }}, Env: {{ environment }}"
```

## Hands-On Lab

### Lab 1: Multi-Environment Configuration

Create a multi-environment setup with variables:

```bash
# Directory structure
mkdir -p lab/group_vars lab/host_vars
```

```yaml
# filepath: lab/inventory.yml
---
all:
  children:
    production:
      hosts:
        prod-web01:
        prod-web02:
    staging:
      hosts:
        stage-web01:
```

```yaml
# filepath: lab/group_vars/all.yml
---
app_name: myapp
app_user: appuser
log_level: info
```

```yaml
# filepath: lab/group_vars/production.yml
---
environment: production
app_port: 80
db_host: prod-db.example.com
workers: 4
log_level: warning
```

```yaml
# filepath: lab/group_vars/staging.yml
---
environment: staging
app_port: 8080
db_host: stage-db.example.com
workers: 2
log_level: debug
```

```yaml
# filepath: lab/deploy.yml
---
- name: Deploy application
  hosts: all
  
  tasks:
    - name: Display configuration
      debug:
        msg: |
          Environment: {{ environment }}
          App: {{ app_name }}
          Port: {{ app_port }}
          DB: {{ db_host }}
          Workers: {{ workers }}
          Log Level: {{ log_level }}
    
    - name: Create app directory
      file:
        path: /opt/{{ app_name }}
        state: directory
        owner: "{{ app_user }}"
        mode: '0755'
      become: yes
    
    - name: Deploy configuration
      copy:
        content: |
          [app]
          name={{ app_name }}
          port={{ app_port }}
          workers={{ workers }}
          
          [database]
          host={{ db_host }}
          
          [logging]
          level={{ log_level }}
        dest: /opt/{{ app_name }}/config.ini
      become: yes
```

Test with different environments:
```bash
ansible-playbook -i lab/inventory.yml lab/deploy.yml --limit production
ansible-playbook -i lab/inventory.yml lab/deploy.yml --limit staging
```

### Lab 2: Working with Facts

```yaml
# filepath: lab/facts-lab.yml
---
- name: Facts laboratory
  hosts: all
  
  tasks:
    - name: Gather minimal facts
      setup:
        gather_subset:
          - '!all'
          - '!min'
          - network
    
    - name: Display system information
      debug:
        msg: |
          OS: {{ ansible_distribution }} {{ ansible_distribution_version }}
          Kernel: {{ ansible_kernel }}
          Architecture: {{ ansible_architecture }}
          Hostname: {{ ansible_hostname }}
          FQDN: {{ ansible_fqdn }}
    
    - name: Display network information
      debug:
        msg: |
          Primary IP: {{ ansible_default_ipv4.address | default('N/A') }}
          Gateway: {{ ansible_default_ipv4.gateway | default('N/A') }}
          All IPs: {{ ansible_all_ipv4_addresses | join(', ') }}
    
    - name: Set custom facts
      set_fact:
        server_role: "{{ 'web' if 'web' in inventory_hostname else 'app' }}"
        deployment_id: "{{ ansible_date_time.epoch }}"
        cacheable: yes
    
    - name: Create dynamic configuration
      copy:
        content: |
          # Server Configuration
          # Generated: {{ ansible_date_time.iso8601 }}
          
          [server]
          hostname={{ ansible_hostname }}
          ip_address={{ ansible_default_ipv4.address | default('127.0.0.1') }}
          role={{ server_role }}
          
          [deployment]
          id={{ deployment_id }}
          os={{ ansible_distribution }} {{ ansible_distribution_version }}
          kernel={{ ansible_kernel }}
        dest: /tmp/server-config-{{ inventory_hostname }}.ini
      delegate_to: localhost
```

### Lab 3: Variable Precedence

```yaml
# filepath: lab/precedence-lab.yml
---
- name: Variable precedence demonstration
  hosts: localhost
  connection: local
  
  vars:
    my_var: "play_vars"
    level: "play"
  
  vars_files:
    - lab/extra_vars.yml
  
  tasks:
    - name: Display initial variable
      debug:
        msg: "my_var={{ my_var }}, level={{ level }}"
    
    - name: Override with task vars
      debug:
        msg: "my_var={{ my_var }}, level={{ level }}"
      vars:
        my_var: "task_vars"
        level: "task"
    
    - name: Back to play vars
      debug:
        msg: "my_var={{ my_var }}, level={{ level }}"
    
    - name: Set fact (high precedence)
      set_fact:
        my_var: "set_fact"
        level: "fact"
    
    - name: After set_fact
      debug:
        msg: "my_var={{ my_var }}, level={{ level }}"
    
    - name: Include vars (high precedence)
      include_vars:
        file: lab/include_vars.yml
    
    - name: After include_vars
      debug:
        msg: "my_var={{ my_var }}, level={{ level }}"
```

```yaml
# filepath: lab/extra_vars.yml
---
my_var: "vars_files"
from_file: true
```

```yaml
# filepath: lab/include_vars.yml
---
my_var: "include_vars"
level: "include"
```

Run with extra vars:
```bash
ansible-playbook lab/precedence-lab.yml
ansible-playbook lab/precedence-lab.yml -e "my_var=extra_vars level=cli"
```

## Summary

In this lesson, you learned:
- ✅ Defining variables in multiple locations
- ✅ Variable precedence and scoping
- ✅ Working with facts and custom facts
- ✅ Using registered variables
- ✅ Magic variables and their uses
- ✅ Set facts and variable prompts

## Additional Resources

- [Ansible Variables](https://docs.ansible.com/ansible/latest/user_guide/playbooks_variables.html)
- [Variable Precedence](https://docs.ansible.com/ansible/latest/user_guide/playbooks_variables.html#variable-precedence-where-should-i-put-a-variable)
- [Facts](https://docs.ansible.com/ansible/latest/user_guide/playbooks_vars_facts.html)
- [Special Variables](https://docs.ansible.com/ansible/latest/reference_appendices/special_variables.html)

## Next Steps

Proceed to [Lesson 06: Templates with Jinja2](../06-templates-jinja2/README.md) to learn how to create dynamic configuration files.
