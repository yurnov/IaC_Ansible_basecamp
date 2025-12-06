# Lesson 07: Conditionals and Loops

## Lesson Objectives
By the end of this lesson, you will be able to:
- Use conditionals (when) effectively in playbooks
- Implement different types of loops
- Combine conditionals and loops
- Use loop control features
- Handle complex iteration scenarios

## Prerequisites
- Completed Lesson 06: Templates with Jinja2
- Understanding of Ansible variables and facts

## Duration
90 minutes

## Lesson Content

### Conditionals with `when`

The `when` statement allows you to run tasks conditionally based on facts, variables, or previous task results.

#### Basic When Statements

```yaml
# filepath: examples/when-basic.yml
---
- name: Basic conditional examples
  hosts: all
  
  vars:
    install_nginx: true
    environment: production
  
  tasks:
    - name: Install nginx (only if enabled)
      apt:
        name: nginx
        state: present
      when: install_nginx
      become: yes
    
    - name: Task for production only
      debug:
        msg: "Running in production"
      when: environment == "production"
    
    - name: Task for non-production
      debug:
        msg: "Running in {{ environment }}"
      when: environment != "production"
```

#### Comparing Values

```yaml
# filepath: examples/when-comparisons.yml
---
- name: Comparison operators
  hosts: all
  
  vars:
    http_port: 80
    app_version: "2.1.0"
    max_connections: 100
  
  tasks:
    - name: Equal to
      debug:
        msg: "Standard HTTP port"
      when: http_port == 80
    
    - name: Not equal to
      debug:
        msg: "Custom port: {{ http_port }}"
      when: http_port != 80
    
    - name: Greater than
      debug:
        msg: "High connection limit"
      when: max_connections > 50
    
    - name: Less than or equal
      debug:
        msg: "Normal connection limit"
      when: max_connections <= 100
    
    - name: Version comparison
      debug:
        msg: "Version is 2.x or higher"
      when: app_version is version('2.0', '>=')
```

#### Logical Operators

```yaml
# filepath: examples/when-logical.yml
---
- name: Logical operators
  hosts: all
  
  vars:
    environment: production
    ssl_enabled: true
    backup_enabled: false
  
  tasks:
    - name: AND operator
      debug:
        msg: "Production with SSL"
      when: environment == "production" and ssl_enabled
    
    - name: OR operator
      debug:
        msg: "Either production or SSL enabled"
      when: environment == "production" or ssl_enabled
    
    - name: NOT operator
      debug:
        msg: "Backup is disabled"
      when: not backup_enabled
    
    - name: Complex condition
      debug:
        msg: "Complex logic passed"
      when: >
        (environment == "production" and ssl_enabled) or
        (environment == "staging" and not backup_enabled)
```

#### Conditional with Facts

```yaml
# filepath: examples/when-facts.yml
---
- name: Conditionals based on facts
  hosts: all
  
  tasks:
    - name: Debian/Ubuntu specific task
      apt:
        update_cache: yes
      when: ansible_os_family == "Debian"
      become: yes
    
    - name: RedHat/CentOS specific task
      yum:
        name: '*'
        state: latest
      when: ansible_os_family == "RedHat"
      become: yes
    
    - name: Check if running on Ubuntu 20.04
      debug:
        msg: "Ubuntu 20.04 detected"
      when:
        - ansible_distribution == "Ubuntu"
        - ansible_distribution_version == "20.04"
    
    - name: Task for systems with more than 2GB RAM
      debug:
        msg: "System has {{ ansible_memtotal_mb }}MB of RAM"
      when: ansible_memtotal_mb > 2048
    
    - name: Task for x86_64 architecture
      debug:
        msg: "Running on 64-bit system"
      when: ansible_architecture == "x86_64"
```

#### Conditional with Registered Variables

```yaml
# filepath: examples/when-register.yml
---
- name: Conditionals with registered variables
  hosts: webservers
  
  tasks:
    - name: Check if nginx is installed
      command: which nginx
      register: nginx_check
      changed_when: false
      failed_when: false
    
    - name: Install nginx if not found
      apt:
        name: nginx
        state: present
      when: nginx_check.rc != 0
      become: yes
    
    - name: Get nginx version
      shell: nginx -v 2>&1 | awk '{print $3}'
      register: nginx_version
      changed_when: false
      when: nginx_check.rc == 0
    
    - name: Display nginx version
      debug:
        msg: "Nginx version: {{ nginx_version.stdout }}"
      when: nginx_version.stdout is defined
    
    - name: Check disk space
      shell: df -h / | tail -1 | awk '{print $5}' | sed 's/%//'
      register: disk_usage
      changed_when: false
    
    - name: Warn if disk is almost full
      debug:
        msg: "WARNING: Disk usage is at {{ disk_usage.stdout }}%"
      when: disk_usage.stdout | int > 80
```

#### Conditional with File Tests

```yaml
# filepath: examples/when-file-tests.yml
---
- name: File-based conditionals
  hosts: all
  
  tasks:
    - name: Check if config file exists
      stat:
        path: /etc/nginx/nginx.conf
      register: config_file
    
    - name: Backup config if exists
      copy:
        src: /etc/nginx/nginx.conf
        dest: /etc/nginx/nginx.conf.backup
        remote_src: yes
      when: config_file.stat.exists
      become: yes
    
    - name: Create config if doesn't exist
      copy:
        content: "# Default config\n"
        dest: /etc/nginx/nginx.conf
      when: not config_file.stat.exists
      become: yes
    
    - name: Check file permissions
      debug:
        msg: "Config file is writable by owner"
      when: config_file.stat.exists and config_file.stat.wusr
```

### Loops

Loops allow you to repeat tasks with different inputs.

#### Basic Loop

```yaml
# filepath: examples/loop-basic.yml
---
- name: Basic loops
  hosts: all
  
  tasks:
    - name: Install multiple packages
      apt:
        name: "{{ item }}"
        state: present
      loop:
        - nginx
        - vim
        - git
        - curl
        - htop
      become: yes
    
    - name: Create multiple users
      user:
        name: "{{ item }}"
        state: present
      loop:
        - alice
        - bob
        - charlie
      become: yes
    
    - name: Create multiple directories
      file:
        path: "/tmp/{{ item }}"
        state: directory
        mode: '0755'
      loop:
        - dir1
        - dir2
        - dir3
```

#### Loop with Dictionaries

```yaml
# filepath: examples/loop-dict.yml
---
- name: Loops with dictionaries
  hosts: all
  
  tasks:
    - name: Create users with specific properties
      user:
        name: "{{ item.name }}"
        comment: "{{ item.comment }}"
        shell: "{{ item.shell }}"
        groups: "{{ item.groups }}"
        state: present
      loop:
        - name: alice
          comment: "Alice Admin"
          shell: /bin/bash
          groups: sudo,docker
        - name: bob
          comment: "Bob Developer"
          shell: /bin/zsh
          groups: docker
        - name: charlie
          comment: "Charlie User"
          shell: /bin/bash
          groups: users
      become: yes
    
    - name: Deploy multiple config files
      template:
        src: "{{ item.src }}"
        dest: "{{ item.dest }}"
        mode: "{{ item.mode }}"
      loop:
        - src: nginx.conf.j2
          dest: /etc/nginx/nginx.conf
          mode: '0644'
        - src: site.conf.j2
          dest: /etc/nginx/sites-available/default
          mode: '0644'
      become: yes
```

#### Loop with Complex Data

```yaml
# filepath: examples/loop-complex.yml
---
- name: Complex loops
  hosts: all
  
  vars:
    web_servers:
      - hostname: web01
        ip: 192.168.1.11
        port: 80
        ssl: true
      - hostname: web02
        ip: 192.168.1.12
        port: 80
        ssl: false
  
  tasks:
    - name: Display server information
      debug:
        msg: >
          Server {{ item.hostname }} at {{ item.ip }}:{{ item.port }}
          (SSL: {{ 'enabled' if item.ssl else 'disabled' }})
      loop: "{{ web_servers }}"
    
    - name: Configure servers
      template:
        src: server-config.j2
        dest: "/tmp/{{ item.hostname }}-config.conf"
      loop: "{{ web_servers }}"
      delegate_to: localhost
```

#### Loop with Ranges

```yaml
# filepath: examples/loop-range.yml
---
- name: Loop with ranges
  hosts: localhost
  connection: local
  
  tasks:
    - name: Create numbered files
      file:
        path: "/tmp/file{{ item }}.txt"
        state: touch
      loop: "{{ range(1, 6) | list }}"
    
    - name: Create directories with padding
      file:
        path: "/tmp/backup_{{ '%02d' | format(item) }}"
        state: directory
      loop: "{{ range(1, 13) | list }}"
    
    - name: Create config files for ports
      copy:
        content: "port: {{ item }}\n"
        dest: "/tmp/config_{{ item }}.conf"
      loop: "{{ range(8080, 8086) | list }}"
```

#### Loop with `with_*` (Legacy)

```yaml
# filepath: examples/loop-with-legacy.yml
---
- name: Legacy loop syntax (still supported)
  hosts: localhost
  connection: local
  
  tasks:
    - name: with_items (use loop instead)
      debug:
        msg: "{{ item }}"
      with_items:
        - apple
        - banana
        - orange
    
    - name: with_dict
      debug:
        msg: "{{ item.key }} = {{ item.value }}"
      with_dict:
        name: myapp
        version: 1.0.0
        port: 8080
    
    - name: with_fileglob
      debug:
        msg: "Found file: {{ item }}"
      with_fileglob:
        - "/tmp/*.txt"
    
    - name: with_sequence
      debug:
        msg: "Number: {{ item }}"
      with_sequence: start=0 end=5 stride=2
```

#### Modern Loop Equivalents

```yaml
# filepath: examples/loop-modern.yml
---
- name: Modern loop syntax
  hosts: localhost
  connection: local
  
  tasks:
    - name: loop (replaces with_items)
      debug:
        msg: "{{ item }}"
      loop:
        - apple
        - banana
        - orange
    
    - name: loop with dict2items
      debug:
        msg: "{{ item.key }} = {{ item.value }}"
      loop: "{{ {'name': 'myapp', 'version': '1.0.0', 'port': 8080} | dict2items }}"
    
    - name: loop with fileglob lookup
      debug:
        msg: "Found file: {{ item }}"
      loop: "{{ lookup('fileglob', '/tmp/*.txt', wantlist=True) }}"
    
    - name: loop with sequence
      debug:
        msg: "Number: {{ item }}"
      loop: "{{ range(0, 6, 2) | list }}"
```

### Loop Control

#### loop_var

```yaml
# filepath: examples/loop-control-var.yml
---
- name: Loop control - custom loop variable
  hosts: localhost
  connection: local
  
  tasks:
    - name: Nested loop with custom variable names
      debug:
        msg: "Outer: {{ outer_item }}, Inner: {{ item }}"
      loop: "{{ range(1, 4) | list }}"
      loop_control:
        loop_var: outer_item
      with_items:
        - a
        - b
        - c
```

#### index_var

```yaml
# filepath: examples/loop-control-index.yml
---
- name: Loop control - index variable
  hosts: localhost
  connection: local
  
  vars:
    packages:
      - nginx
      - vim
      - git
  
  tasks:
    - name: Display packages with index
      debug:
        msg: "{{ idx + 1 }}. {{ item }}"
      loop: "{{ packages }}"
      loop_control:
        index_var: idx
```

#### label

```yaml
# filepath: examples/loop-control-label.yml
---
- name: Loop control - labels for cleaner output
  hosts: all
  
  vars:
    users:
      - name: alice
        uid: 1001
        groups: sudo,docker
        shell: /bin/bash
      - name: bob
        uid: 1002
        groups: docker
        shell: /bin/zsh
  
  tasks:
    - name: Create users
      user:
        name: "{{ item.name }}"
        uid: "{{ item.uid }}"
        groups: "{{ item.groups }}"
        shell: "{{ item.shell }}"
        state: present
      loop: "{{ users }}"
      loop_control:
        label: "{{ item.name }}"
      become: yes
```

#### pause

```yaml
# filepath: examples/loop-control-pause.yml
---
- name: Loop control - pause between iterations
  hosts: webservers
  
  tasks:
    - name: Restart services with delay
      service:
        name: "{{ item }}"
        state: restarted
      loop:
        - nginx
        - php-fpm
        - redis
      loop_control:
        pause: 10
      become: yes
```

### Combining Conditionals and Loops

```yaml
# filepath: examples/conditional-loops.yml
---
- name: Combining conditionals and loops
  hosts: all
  
  vars:
    packages:
      - name: nginx
        state: present
        enabled: true
      - name: apache2
        state: absent
        enabled: false
      - name: vim
        state: present
        enabled: true
      - name: emacs
        state: present
        enabled: false
  
  tasks:
    - name: Install only enabled packages
      apt:
        name: "{{ item.name }}"
        state: "{{ item.state }}"
      loop: "{{ packages }}"
      when: item.enabled
      become: yes
    
    - name: Create users only on Ubuntu
      user:
        name: "{{ item }}"
        state: present
      loop:
        - user1
        - user2
        - user3
      when: ansible_distribution == "Ubuntu"
      become: yes
    
    - name: Install packages based on OS family
      package:
        name: "{{ item }}"
        state: present
      loop:
        - vim
        - git
        - curl
      when: ansible_os_family in ["Debian", "RedHat"]
      become: yes
```

### Loop Filters

```yaml
# filepath: examples/loop-filters.yml
---
- name: Loop with filters
  hosts: localhost
  connection: local
  
  vars:
    all_packages:
      - nginx
      - apache2
      - vim
      - emacs
      - git
      - svn
    
    numbers:
      - 1
      - 5
      - 3
      - 9
      - 2
      - 7
  
  tasks:
    - name: Loop with select filter
      debug:
        msg: "Package: {{ item }}"
      loop: "{{ all_packages | select('match', '^[ng].*') | list }}"
    
    - name: Loop with reject filter
      debug:
        msg: "Package: {{ item }}"
      loop: "{{ all_packages | reject('match', 'apache.*') | list }}"
    
    - name: Loop with sorted items
      debug:
        msg: "Package: {{ item }}"
      loop: "{{ all_packages | sort }}"
    
    - name: Loop with unique items
      debug:
        msg: "Number: {{ item }}"
      loop: "{{ ([1, 2, 2, 3, 3, 3, 4] | unique) }}"
    
    - name: Loop with filtered numbers
      debug:
        msg: "Number: {{ item }}"
      loop: "{{ numbers | select('>', 5) | list }}"
```

### until Loops (Retry Logic)

```yaml
# filepath: examples/loop-until.yml
---
- name: Loop until condition is met
  hosts: webservers
  
  tasks:
    - name: Wait for service to be ready
      uri:
        url: "http://localhost:8080/health"
        status_code: 200
      register: result
      until: result.status == 200
      retries: 10
      delay: 5
    
    - name: Wait for file to appear
      stat:
        path: /tmp/deployment.lock
      register: lock_file
      until: lock_file.stat.exists
      retries: 30
      delay: 2
    
    - name: Wait for command to succeed
      command: /usr/local/bin/check-status.sh
      register: status_check
      until: status_check.rc == 0
      retries: 5
      delay: 10
      changed_when: false
```

### Nested Loops

```yaml
# filepath: examples/loop-nested.yml
---
- name: Nested loops
  hosts: localhost
  connection: local
  
  vars:
    users:
      - alice
      - bob
    
    groups:
      - developers
      - operators
  
  tasks:
    - name: Add users to groups (Cartesian product)
      debug:
        msg: "Adding {{ item.0 }} to {{ item.1 }}"
      loop: "{{ users | product(groups) | list }}"
    
    - name: Create files in multiple directories
      file:
        path: "{{ item.0 }}/{{ item.1 }}"
        state: touch
      loop: "{{ ['/tmp/dir1', '/tmp/dir2'] | product(['file1.txt', 'file2.txt', 'file3.txt']) | list }}"
```

### Loop with Subelements

```yaml
# filepath: examples/loop-subelements.yml
---
- name: Loop with subelements
  hosts: localhost
  connection: local
  
  vars:
    users:
      - name: alice
        authorized_keys:
          - ssh-rsa AAAA...
          - ssh-ed25519 BBBB...
      - name: bob
        authorized_keys:
          - ssh-rsa CCCC...
  
  tasks:
    - name: Add SSH keys for users
      debug:
        msg: "Adding key for {{ item.0.name }}: {{ item.1 }}"
      loop: "{{ users | subelements('authorized_keys') }}"
```

## Hands-On Lab

### Lab 1: OS-Specific Package Installation

```yaml
# filepath: lab/os-specific.yml
---
- name: Install packages based on OS
  hosts: all
  become: yes
  
  vars:
    common_packages:
      - git
      - curl
      - wget
      - vim
  
  tasks:
    - name: Install packages on Debian/Ubuntu
      apt:
        name: "{{ item }}"
        state: present
        update_cache: yes
      loop: "{{ common_packages }}"
      when: ansible_os_family == "Debian"
    
    - name: Install packages on RedHat/CentOS
      yum:
        name: "{{ item }}"
        state: present
      loop: "{{ common_packages }}"
      when: ansible_os_family == "RedHat"
    
    - name: Install web server (Debian)
      apt:
        name: nginx
        state: present
      when: ansible_os_family == "Debian"
    
    - name: Install web server (RedHat)
      yum:
        name: httpd
        state: present
      when: ansible_os_family == "RedHat"
```

### Lab 2: User and Group Management

```yaml
# filepath: lab/users-groups.yml
---
- name: Manage users and groups
  hosts: all
  become: yes
  
  vars:
    app_users:
      - name: webapp
        comment: "Web Application User"
        shell: /bin/bash
        groups: www-data
        create_home: yes
        enabled: true
      - name: dbadmin
        comment: "Database Administrator"
        shell: /bin/bash
        groups: sudo
        create_home: yes
        enabled: true
      - name: tempuser
        comment: "Temporary User"
        shell: /bin/bash
        groups: users
        create_home: no
        enabled: false
  
  tasks:
    - name: Create groups
      group:
        name: "{{ item }}"
        state: present
      loop:
        - www-data
        - dbusers
    
    - name: Create enabled users
      user:
        name: "{{ item.name }}"
        comment: "{{ item.comment }}"
        shell: "{{ item.shell }}"
        groups: "{{ item.groups }}"
        create_home: "{{ item.create_home }}"
        state: present
      loop: "{{ app_users }}"
      when: item.enabled
      loop_control:
        label: "{{ item.name }}"
    
    - name: Set password for users with sudo access
      user:
        name: "{{ item.name }}"
        password: "{{ 'changeme' | password_hash('sha512') }}"
        update_password: on_create
      loop: "{{ app_users }}"
      when: 
        - item.enabled
        - "'sudo' in item.groups"
      loop_control:
        label: "{{ item.name }}"
```

### Lab 3: Service Health Check and Restart

```yaml
# filepath: lab/service-health.yml
---
- name: Check and manage services
  hosts: webservers
  become: yes
  
  vars:
    services:
      - name: nginx
        port: 80
        enabled: true
      - name: php-fpm
        port: 9000
        enabled: true
      - name: redis
        port: 6379
        enabled: false
  
  tasks:
    - name: Check if enabled services are running
      service_facts:
    
    - name: Display service status
      debug:
        msg: "{{ item.name }} is {{ ansible_facts.services[item.name + '.service'].state | default('not found') }}"
      loop: "{{ services }}"
      when: item.enabled
      loop_control:
        label: "{{ item.name }}"
    
    - name: Ensure enabled services are started
      service:
        name: "{{ item.name }}"
        state: started
        enabled: yes
      loop: "{{ services }}"
      when: item.enabled
      loop_control:
        label: "{{ item.name }}"
    
    - name: Check service ports
      wait_for:
        host: localhost
        port: "{{ item.port }}"
        timeout: 5
        state: started
      loop: "{{ services }}"
      when: item.enabled
      register: port_check
      failed_when: false
      loop_control:
        label: "{{ item.name }}:{{ item.port }}"
    
    - name: Restart failed services
      service:
        name: "{{ item.item.name }}"
        state: restarted
      loop: "{{ port_check.results }}"
      when:
        - item.item.enabled
        - item.failed is defined
        - item.failed
      loop_control:
        label: "{{ item.item.name }}"
```

### Lab 4: Dynamic Configuration Deployment

```yaml
# filepath: lab/dynamic-config.yml
---
- name: Deploy dynamic configurations
  hosts: all
  
  vars:
    environments:
      production:
        log_level: warning
        debug: false
        workers: 4
        db_pool: 20
      staging:
        log_level: info
        debug: false
        workers: 2
        db_pool: 10
      development:
        log_level: debug
        debug: true
        workers: 1
        db_pool: 5
    
    current_env: "{{ environment | default('development') }}"
  
  tasks:
    - name: Display environment configuration
      debug:
        msg: |
          Environment: {{ current_env }}
          Log Level: {{ environments[current_env].log_level }}
          Debug: {{ environments[current_env].debug }}
          Workers: {{ environments[current_env].workers }}
          DB Pool: {{ environments[current_env].db_pool }}
    
    - name: Create config file
      copy:
        content: |
          [app]
          environment={{ current_env }}
          log_level={{ environments[current_env].log_level }}
          debug={{ environments[current_env].debug | lower }}
          workers={{ environments[current_env].workers }}
          
          [database]
          pool_size={{ environments[current_env].db_pool }}
        dest: "/tmp/{{ inventory_hostname }}-{{ current_env }}.conf"
      delegate_to: localhost
```

Run with different environments:
```bash
ansible-playbook lab/dynamic-config.yml -e "environment=production"
ansible-playbook lab/dynamic-config.yml -e "environment=staging"
ansible-playbook lab/dynamic-config.yml -e "environment=development"
```

### Lab 5: File Backup and Cleanup

```yaml
# filepath: lab/backup-cleanup.yml
---
- name: Backup and cleanup old files
  hosts: all
  
  vars:
    backup_dirs:
      - /etc/nginx
      - /etc/apache2
      - /var/www
    
    backup_dest: /backup
    max_age_days: 30
  
  tasks:
    - name: Check if directories exist
      stat:
        path: "{{ item }}"
      register: dir_check
      loop: "{{ backup_dirs }}"
      loop_control:
        label: "{{ item }}"
    
    - name: Create backup directory
      file:
        path: "{{ backup_dest }}"
        state: directory
        mode: '0755'
      become: yes
    
    - name: Backup existing directories
      archive:
        path: "{{ item.item }}"
        dest: "{{ backup_dest }}/{{ item.item | basename }}-{{ ansible_date_time.epoch }}.tar.gz"
        format: gz
      loop: "{{ dir_check.results }}"
      when: item.stat.exists
      become: yes
      loop_control:
        label: "{{ item.item }}"
    
    - name: Find old backups
      find:
        paths: "{{ backup_dest }}"
        patterns: "*.tar.gz"
        age: "{{ max_age_days }}d"
      register: old_backups
      become: yes
    
    - name: Remove old backups
      file:
        path: "{{ item.path }}"
        state: absent
      loop: "{{ old_backups.files }}"
      when: old_backups.matched > 0
      become: yes
      loop_control:
        label: "{{ item.path }}"
```

## Summary

In this lesson, you learned:
- ✅ Using conditionals with `when` statements
- ✅ Implementing various types of loops
- ✅ Loop control features (labels, pause, index)
- ✅ Combining conditionals and loops
- ✅ Retry logic with `until`
- ✅ Nested loops and complex iterations

## Additional Resources

- [Ansible Conditionals](https://docs.ansible.com/ansible/latest/user_guide/playbooks_conditionals.html)
- [Ansible Loops](https://docs.ansible.com/ansible/latest/user_guide/playbooks_loops.html)
- [Loop Control](https://docs.ansible.com/ansible/latest/user_guide/playbooks_loops.html#adding-controls-to-loops)
- [Tests](https://docs.ansible.com/ansible/latest/user_guide/playbooks_tests.html)

## Next Steps

Proceed to [Lesson 08: Handlers and Error Handling](../08-handlers-errors/README.md) to learn about managing service states and handling errors effectively.
