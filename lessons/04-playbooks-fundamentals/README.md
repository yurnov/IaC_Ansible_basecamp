# Lesson 04: Playbooks Fundamentals

## Lesson Objectives
By the end of this lesson, you will be able to:
- Understand playbook structure and syntax
- Write basic playbooks
- Use tasks, plays, and handlers
- Control execution flow
- Debug playbooks effectively

## Prerequisites
- Completed Lesson 03: Inventory Management
- Understanding of YAML syntax
- Basic knowledge of Ansible modules

## Duration
90 minutes

## Lesson Content

### What is a Playbook?

A playbook is a YAML file containing one or more "plays" that define:
- Which hosts to target
- What tasks to execute
- How to execute them (order, privileges, etc.)

### Basic Playbook Structure

```yaml
# filepath: examples/basic-playbook.yml
---
- name: My First Playbook
  hosts: webservers
  become: yes
  
  tasks:
    - name: Install nginx
      apt:
        name: nginx
        state: present
    
    - name: Start nginx service
      service:
        name: nginx
        state: started
        enabled: yes
```

### Anatomy of a Play

```yaml
# filepath: examples/play-anatomy.yml
---
- name: Play name (descriptive)           # Play level
  hosts: target_hosts                     # Required: which hosts
  become: yes                             # Privilege escalation
  become_user: root                       # User to become
  gather_facts: yes                       # Collect system facts
  vars:                                   # Play variables
    http_port: 80
  
  tasks:                                  # List of tasks
    - name: Task name (descriptive)       # Task level
      module_name:                        # Module to use
        parameter1: value1
        parameter2: value2
```

### Multiple Plays in One Playbook

```yaml
# filepath: examples/multiple-plays.yml
---
- name: Configure web servers
  hosts: webservers
  become: yes
  
  tasks:
    - name: Install nginx
      apt:
        name: nginx
        state: present

- name: Configure database servers
  hosts: databases
  become: yes
  
  tasks:
    - name: Install postgresql
      apt:
        name: postgresql
        state: present

- name: Configure all servers
  hosts: all
  become: yes
  
  tasks:
    - name: Update system
      apt:
        update_cache: yes
        upgrade: dist
```

### Task Syntax Styles

#### Dictionary Style (Recommended)

```yaml
# filepath: examples/task-dictionary.yml
---
- name: Dictionary style tasks
  hosts: webservers
  
  tasks:
    - name: Install package
      apt:
        name: nginx
        state: present
        update_cache: yes
        cache_valid_time: 3600
```

#### Single Line Style

```yaml
# filepath: examples/task-single-line.yml
---
- name: Single line style
  hosts: webservers
  
  tasks:
    - name: Install package
      apt: name=nginx state=present
```

#### Key=Value Style (Not Recommended)

```yaml
# filepath: examples/task-keyvalue.yml
---
- name: Key-value style (avoid)
  hosts: webservers
  
  tasks:
    - name: Install package
      apt: name=nginx state=present update_cache=yes
```

### Common Playbook Directives

#### hosts

```yaml
# filepath: examples/hosts-directive.yml
---
# Single group
- hosts: webservers

# Multiple groups
- hosts: webservers,databases

# All hosts
- hosts: all

# Pattern matching
- hosts: web*

# Complex patterns
- hosts: webservers:&production:!maintenance
```

#### become

```yaml
# filepath: examples/become-directive.yml
---
- name: Privilege escalation
  hosts: webservers
  become: yes              # Enable privilege escalation
  become_method: sudo      # Method (sudo, su, pbrun, etc.)
  become_user: root        # User to become
  
  tasks:
    - name: Task with become
      apt:
        name: nginx
        state: present
    
    - name: Task without become
      command: whoami
      become: no           # Override play-level become
```

#### gather_facts

```yaml
# filepath: examples/gather-facts.yml
---
- name: With facts gathering
  hosts: webservers
  gather_facts: yes      # Default
  
  tasks:
    - name: Display OS
      debug:
        msg: "OS is {{ ansible_distribution }}"

- name: Without facts gathering
  hosts: webservers
  gather_facts: no       # Skip for speed
  
  tasks:
    - name: Simple task
      ping:
```

### Working with Tasks

#### Task Names

```yaml
# filepath: examples/task-names.yml
---
- name: Task naming best practices
  hosts: webservers
  
  tasks:
    # ✅ Good: Descriptive and specific
    - name: Install nginx web server version 1.18
      apt:
        name: nginx=1.18.*
        state: present
    
    # ❌ Bad: Too vague
    - name: Install stuff
      apt:
        name: nginx
        state: present
    
    # ✅ Good: Action-oriented
    - name: Ensure nginx service is running and enabled at boot
      service:
        name: nginx
        state: started
        enabled: yes
```

#### Task Parameters

```yaml
# filepath: examples/task-parameters.yml
---
- name: Task parameters
  hosts: webservers
  
  tasks:
    - name: Copy configuration file
      copy:
        src: /local/path/nginx.conf
        dest: /etc/nginx/nginx.conf
        owner: root
        group: root
        mode: '0644'
        backup: yes
        validate: 'nginx -t -c %s'
```

### Return Values and Register

```yaml
# filepath: examples/register.yml
---
- name: Using register
  hosts: webservers
  
  tasks:
    - name: Check if nginx is installed
      command: which nginx
      register: nginx_check
      ignore_errors: yes
      changed_when: false
    
    - name: Display check result
      debug:
        var: nginx_check
    
    - name: Install nginx if not present
      apt:
        name: nginx
        state: present
      when: nginx_check.rc != 0
      become: yes
    
    - name: Get nginx version
      shell: nginx -v 2>&1
      register: nginx_version
      changed_when: false
    
    - name: Display nginx version
      debug:
        msg: "Nginx version: {{ nginx_version.stdout }}"
```

### Debug Module

```yaml
# filepath: examples/debug.yml
---
- name: Debugging playbooks
  hosts: localhost
  connection: local
  gather_facts: yes
  
  vars:
    my_var: "Hello World"
    my_list:
      - item1
      - item2
      - item3
  
  tasks:
    - name: Debug with msg
      debug:
        msg: "The variable value is {{ my_var }}"
    
    - name: Debug with var
      debug:
        var: my_var
    
    - name: Debug complex variable
      debug:
        var: my_list
    
    - name: Debug with verbosity
      debug:
        msg: "This only shows with -v or higher"
        verbosity: 1
    
    - name: Debug facts
      debug:
        msg: "OS: {{ ansible_distribution }} {{ ansible_distribution_version }}"
```

### Failed_when and Changed_when

```yaml
# filepath: examples/failed-changed-when.yml
---
- name: Control task status
  hosts: webservers
  
  tasks:
    - name: Check configuration
      command: /usr/local/bin/check-config.sh
      register: config_check
      failed_when: "'ERROR' in config_check.stderr"
      changed_when: false
    
    - name: Run backup script
      script: /scripts/backup.sh
      register: backup_result
      changed_when: "'backed up' in backup_result.stdout"
      failed_when: 
        - backup_result.rc != 0
        - "'WARNING' not in backup_result.stderr"
    
    - name: Check disk space
      shell: df -h / | tail -1 | awk '{print $5}' | sed 's/%//'
      register: disk_usage
      failed_when: disk_usage.stdout|int > 90
      changed_when: false
```

### Tags

```yaml
# filepath: examples/tags.yml
---
- name: Using tags
  hosts: webservers
  
  tasks:
    - name: Install nginx
      apt:
        name: nginx
        state: present
      tags:
        - packages
        - nginx
        - install
    
    - name: Configure nginx
      copy:
        src: nginx.conf
        dest: /etc/nginx/nginx.conf
      tags:
        - configuration
        - nginx
    
    - name: Start nginx
      service:
        name: nginx
        state: started
      tags:
        - services
        - nginx
    
    - name: Install monitoring tools
      apt:
        name: "{{ item }}"
        state: present
      loop:
        - htop
        - iotop
        - nethogs
      tags:
        - packages
        - monitoring
        - never  # Only run when explicitly specified
```

```bash
# Running with tags
ansible-playbook playbook.yml --tags "nginx"
ansible-playbook playbook.yml --tags "install,configuration"
ansible-playbook playbook.yml --skip-tags "packages"
ansible-playbook playbook.yml --tags "never"  # Run "never" tasks
ansible-playbook playbook.yml --list-tags     # List all tags
```

### Handlers

Handlers are tasks that run only when notified by other tasks.

```yaml
# filepath: examples/handlers.yml
---
- name: Using handlers
  hosts: webservers
  become: yes
  
  tasks:
    - name: Install nginx
      apt:
        name: nginx
        state: present
      notify: Start nginx
    
    - name: Copy nginx configuration
      copy:
        src: nginx.conf
        dest: /etc/nginx/nginx.conf
      notify:
        - Validate nginx config
        - Reload nginx
    
    - name: Copy site configuration
      template:
        src: site.conf.j2
        dest: /etc/nginx/sites-available/mysite
      notify: Reload nginx
  
  handlers:
    - name: Start nginx
      service:
        name: nginx
        state: started
        enabled: yes
    
    - name: Reload nginx
      service:
        name: nginx
        state: reloaded
    
    - name: Restart nginx
      service:
        name: nginx
        state: restarted
    
    - name: Validate nginx config
      command: nginx -t
```

#### Handler Execution Order

```yaml
# filepath: examples/handler-order.yml
---
- name: Handler execution
  hosts: webservers
  become: yes
  
  tasks:
    - name: Task 1
      copy:
        content: "config1"
        dest: /tmp/file1
      notify: Handler B
    
    - name: Task 2
      copy:
        content: "config2"
        dest: /tmp/file2
      notify: Handler A
    
    - name: Task 3
      copy:
        content: "config3"
        dest: /tmp/file3
      notify: Handler B
    
    - name: Force handlers to run now
      meta: flush_handlers
    
    - name: Task 4 (runs after handlers)
      debug:
        msg: "Handlers have been executed"
  
  handlers:
    # Handlers run in the order defined, not the order notified
    - name: Handler A
      debug:
        msg: "Handler A executed"
    
    - name: Handler B
      debug:
        msg: "Handler B executed (runs once even if notified multiple times)"
```

### Playbook Execution Control

```yaml
# filepath: examples/execution-control.yml
---
- name: Execution control
  hosts: webservers
  serial: 2                    # Process 2 hosts at a time
  max_fail_percentage: 25      # Fail if more than 25% of hosts fail
  
  tasks:
    - name: Update application
      git:
        repo: https://github.com/user/app.git
        dest: /var/www/app
        version: main
    
    - name: Run deployment script
      command: /var/www/app/deploy.sh
      async: 300                # Run for up to 300 seconds
      poll: 10                  # Check every 10 seconds
    
    - name: Wait for service to be ready
      wait_for:
        port: 8080
        delay: 5
        timeout: 60
```

### Running Playbooks

```bash
# Basic execution
ansible-playbook playbook.yml

# With specific inventory
ansible-playbook -i inventory.yml playbook.yml

# Limit to specific hosts
ansible-playbook playbook.yml --limit webservers
ansible-playbook playbook.yml --limit web01,web02

# Check mode (dry run)
ansible-playbook playbook.yml --check

# Diff mode (show changes)
ansible-playbook playbook.yml --check --diff

# With extra variables
ansible-playbook playbook.yml -e "version=1.2.3"
ansible-playbook playbook.yml -e "@vars.yml"

# With tags
ansible-playbook playbook.yml --tags "configuration"
ansible-playbook playbook.yml --skip-tags "packages"

# Verbose output
ansible-playbook playbook.yml -v
ansible-playbook playbook.yml -vvv

# Syntax check
ansible-playbook playbook.yml --syntax-check

# List tasks
ansible-playbook playbook.yml --list-tasks

# List hosts
ansible-playbook playbook.yml --list-hosts

# Step through tasks
ansible-playbook playbook.yml --step

# Start at specific task
ansible-playbook playbook.yml --start-at-task="Install nginx"
```

## Hands-On Lab

### Lab 1: Basic Playbook

Create a playbook to set up a web server:

```yaml
# filepath: lab/webserver.yml
---
- name: Setup web server
  hosts: webservers
  become: yes
  
  tasks:
    - name: Update apt cache
      apt:
        update_cache: yes
        cache_valid_time: 3600
    
    - name: Install nginx
      apt:
        name: nginx
        state: present
    
    - name: Start and enable nginx
      service:
        name: nginx
        state: started
        enabled: yes
    
    - name: Create web root directory
      file:
        path: /var/www/mysite
        state: directory
        owner: www-data
        group: www-data
        mode: '0755'
    
    - name: Deploy index page
      copy:
        content: |
          <!DOCTYPE html>
          <html>
          <head><title>My Site</title></head>
          <body>
            <h1>Welcome to My Site</h1>
            <p>Deployed with Ansible</p>
          </body>
          </html>
        dest: /var/www/mysite/index.html
        owner: www-data
        group: www-data
        mode: '0644'
    
    - name: Verify nginx is responding
      uri:
        url: http://localhost
        status_code: 200
```

Run the playbook:
```bash
ansible-playbook -i inventory.ini webserver.yml
ansible-playbook -i inventory.ini webserver.yml --check --diff
```

### Lab 2: Playbook with Handlers

```yaml
# filepath: lab/nginx-config.yml
---
- name: Configure nginx with handlers
  hosts: webservers
  become: yes
  
  vars:
    nginx_worker_processes: 4
    nginx_worker_connections: 1024
  
  tasks:
    - name: Install nginx
      apt:
        name: nginx
        state: present
    
    - name: Copy main nginx configuration
      copy:
        content: |
          user www-data;
          worker_processes {{ nginx_worker_processes }};
          
          events {
            worker_connections {{ nginx_worker_connections }};
          }
          
          http {
            include /etc/nginx/mime.types;
            default_type application/octet-stream;
            
            sendfile on;
            keepalive_timeout 65;
            
            include /etc/nginx/conf.d/*.conf;
            include /etc/nginx/sites-enabled/*;
          }
        dest: /etc/nginx/nginx.conf
        backup: yes
      notify:
        - Validate nginx configuration
        - Reload nginx
    
    - name: Ensure nginx is started
      service:
        name: nginx
        state: started
        enabled: yes
  
  handlers:
    - name: Validate nginx configuration
      command: nginx -t
    
    - name: Reload nginx
      service:
        name: nginx
        state: reloaded
    
    - name: Restart nginx
      service:
        name: nginx
        state: restarted
```

### Lab 3: Multi-Play Playbook

```yaml
# filepath: lab/full-stack.yml
---
- name: Configure load balancer
  hosts: loadbalancers
  become: yes
  
  tasks:
    - name: Install HAProxy
      apt:
        name: haproxy
        state: present
    
    - name: Display message
      debug:
        msg: "Load balancer configured on {{ inventory_hostname }}"

- name: Configure web servers
  hosts: webservers
  become: yes
  serial: 2
  
  tasks:
    - name: Install nginx
      apt:
        name: nginx
        state: present
    
    - name: Deploy application
      git:
        repo: https://github.com/ansible/ansible-examples.git
        dest: /var/www/app
        version: master
    
    - name: Display message
      debug:
        msg: "Web server configured on {{ inventory_hostname }}"

- name: Configure database
  hosts: databases
  become: yes
  
  tasks:
    - name: Install PostgreSQL
      apt:
        name: postgresql
        state: present
    
    - name: Display message
      debug:
        msg: "Database configured on {{ inventory_hostname }}"

- name: Verify deployment
  hosts: all
  gather_facts: no
  
  tasks:
    - name: Check all services are running
      debug:
        msg: "Deployment complete for {{ inventory_hostname }}"
```

### Lab 4: Debugging Playbook

Create a playbook to practice debugging:

```yaml
# filepath: lab/debug-practice.yml
---
- name: Debug practice
  hosts: localhost
  connection: local
  gather_facts: yes
  
  vars:
    app_name: myapp
    app_version: 1.0.0
    app_port: 8080
  
  tasks:
    - name: Display variables
      debug:
        msg: "App: {{ app_name }} v{{ app_version }} on port {{ app_port }}"
    
    - name: Check system facts
      debug:
        var: ansible_distribution
    
    - name: Run command and register result
      command: date +%Y-%m-%d
      register: current_date
      changed_when: false
    
    - name: Display command result
      debug:
        var: current_date
    
    - name: Display specific output
      debug:
        msg: "Today is {{ current_date.stdout }}"
    
    - name: Conditional debug
      debug:
        msg: "This is a Debian-based system"
      when: ansible_os_family == "Debian"
```

Run with different verbosity levels:
```bash
ansible-playbook debug-practice.yml
ansible-playbook debug-practice.yml -v
ansible-playbook debug-practice.yml -vv
```

## Summary

In this lesson, you learned:
- ✅ Playbook structure and syntax
- ✅ Writing tasks and plays
- ✅ Using handlers for service management
- ✅ Controlling playbook execution
- ✅ Debugging techniques
- ✅ Best practices for playbook organization

## Additional Resources

- [Ansible Playbooks](https://docs.ansible.com/ansible/latest/user_guide/playbooks.html)
- [Playbook Keywords](https://docs.ansible.com/ansible/latest/reference_appendices/playbooks_keywords.html)
- [Best Practices](https://docs.ansible.com/ansible/latest/user_guide/playbooks_best_practices.html)

## Next Steps

Proceed to [Lesson 05: Variables and Facts](../05-variables-facts/README.md) to learn about managing data in Ansible.
