# Lesson 08: Handlers and Error Handling

## Lesson Objectives
By the end of this lesson, you will be able to:
- Understand and use handlers effectively
- Implement error handling strategies
- Use blocks for error recovery
- Control task failure behavior
- Implement rescue and always blocks

## Prerequisites
- Completed Lesson 07: Conditionals and Loops
- Understanding of playbook basics

## Duration
75 minutes

## Lesson Content

### Handlers

Handlers are special tasks that run only when notified by other tasks, typically used for service management.

#### Basic Handlers

```yaml
# filepath: examples/handlers-basic.yml
---
- name: Basic handler usage
  hosts: webservers
  become: yes
  
  tasks:
    - name: Install nginx
      apt:
        name: nginx
        state: present
      notify: Start nginx
    
    - name: Copy nginx config
      copy:
        src: nginx.conf
        dest: /etc/nginx/nginx.conf
      notify: Reload nginx
    
    - name: Copy site config
      template:
        src: site.conf.j2
        dest: /etc/nginx/sites-available/mysite
      notify:
        - Validate nginx config
        - Reload nginx
  
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
      changed_when: false
```

#### Handler Behavior

```yaml
# filepath: examples/handlers-behavior.yml
---
- name: Understanding handler behavior
  hosts: webservers
  become: yes
  
  tasks:
    # Handlers run at the end of the play
    - name: Task 1 notifies handler A
      copy:
        content: "config1"
        dest: /tmp/config1.txt
      notify: Handler A
    
    - name: Task 2 notifies handler B
      copy:
        content: "config2"
        dest: /tmp/config2.txt
      notify: Handler B
    
    - name: Task 3 notifies handler A again
      copy:
        content: "config3"
        dest: /tmp/config3.txt
      notify: Handler A
    
    # Handlers run in the order they are defined, not notified
    # Handler A runs only once, even though notified twice
  
  handlers:
    - name: Handler A
      debug:
        msg: "Handler A executed (runs once)"
    
    - name: Handler B
      debug:
        msg: "Handler B executed"
```

#### Forcing Handler Execution

```yaml
# filepath: examples/handlers-flush.yml
---
- name: Force handlers to run immediately
  hosts: webservers
  become: yes
  
  tasks:
    - name: Update configuration
      copy:
        src: app.conf
        dest: /etc/app/app.conf
      notify: Restart app
    
    # Force handlers to run now
    - name: Flush handlers
      meta: flush_handlers
    
    # This task runs after handlers complete
    - name: Verify app is running
      uri:
        url: http://localhost:8080/health
        status_code: 200
      retries: 5
      delay: 2
  
  handlers:
    - name: Restart app
      service:
        name: myapp
        state: restarted
```

#### Listen to Multiple Handlers

```yaml
# filepath: examples/handlers-listen.yml
---
- name: Handlers with listen
  hosts: webservers
  become: yes
  
  tasks:
    - name: Update web server config
      template:
        src: nginx.conf.j2
        dest: /etc/nginx/nginx.conf
      notify: Restart web services
    
    - name: Update PHP config
      template:
        src: php.ini.j2
        dest: /etc/php/8.1/fpm/php.ini
      notify: Restart web services
  
  handlers:
    # Multiple handlers can listen to the same notification
    - name: Restart nginx
      service:
        name: nginx
        state: restarted
      listen: Restart web services
    
    - name: Restart php-fpm
      service:
        name: php8.1-fpm
        state: restarted
      listen: Restart web services
    
    - name: Clear cache
      file:
        path: /var/cache/nginx
        state: absent
      listen: Restart web services
```

#### Handler Conditions

```yaml
# filepath: examples/handlers-conditions.yml
---
- name: Conditional handlers
  hosts: webservers
  become: yes
  
  vars:
    enable_ssl: true
    reload_config: true
  
  tasks:
    - name: Update nginx config
      template:
        src: nginx.conf.j2
        dest: /etc/nginx/nginx.conf
      notify:
        - Validate nginx
        - Reload nginx
  
  handlers:
    - name: Validate nginx
      command: nginx -t
      changed_when: false
    
    - name: Reload nginx
      service:
        name: nginx
        state: reloaded
      when: reload_config
    
    - name: Restart nginx
      service:
        name: nginx
        state: restarted
      when: enable_ssl
```

### Error Handling

#### Failed_when

```yaml
# filepath: examples/failed-when.yml
---
- name: Control task failure
  hosts: all
  
  tasks:
    - name: Check application status
      command: /usr/local/bin/check-app.sh
      register: app_status
      failed_when: "'ERROR' in app_status.stderr"
    
    - name: Verify disk space
      shell: df -h / | tail -1 | awk '{print $5}' | sed 's/%//'
      register: disk_usage
      failed_when: disk_usage.stdout | int > 90
      changed_when: false
    
    - name: Check service status
      systemd:
        name: nginx
      register: service_check
      failed_when:
        - service_check.status.ActiveState != "active"
        - service_check.status.SubState != "running"
    
    - name: Multiple failure conditions
      shell: /path/to/script.sh
      register: script_result
      failed_when:
        - script_result.rc != 0
        - script_result.rc != 2  # Allow rc=2 as acceptable
        - "'WARNING' not in script_result.stderr"
```

#### Ignore_errors

```yaml
# filepath: examples/ignore-errors.yml
---
- name: Ignore errors and continue
  hosts: all
  
  tasks:
    - name: Try to stop service (may not exist)
      service:
        name: optional-service
        state: stopped
      ignore_errors: yes
    
    - name: Check if file exists (don't fail if missing)
      stat:
        path: /etc/optional-config.conf
      register: config_file
      ignore_errors: yes
    
    - name: Create file if it doesn't exist
      copy:
        content: "default config"
        dest: /etc/optional-config.conf
      when: not config_file.stat.exists
    
    # This is better practice - use failed_when
    - name: Better approach - define what is failure
      command: optional-command
      register: result
      failed_when: 
        - result.rc != 0
        - result.rc != 127  # Command not found is OK
```

#### Any_errors_fatal

```yaml
# filepath: examples/any-errors-fatal.yml
---
- name: Stop entire play on any error
  hosts: webservers
  any_errors_fatal: true
  
  tasks:
    - name: Critical configuration update
      template:
        src: critical-config.j2
        dest: /etc/app/config.conf
    
    - name: Restart critical service
      service:
        name: critical-app
        state: restarted
    
    # If any host fails above tasks, play stops for all hosts

- name: Second play (won't run if first play failed)
  hosts: databases
  
  tasks:
    - name: Update database
      command: /usr/local/bin/db-migrate.sh
```

#### Max_fail_percentage

```yaml
# filepath: examples/max-fail-percentage.yml
---
- name: Allow some failures
  hosts: webservers
  max_fail_percentage: 25
  serial: 2
  
  tasks:
    - name: Deploy application
      git:
        repo: https://github.com/user/app.git
        dest: /var/www/app
        version: main
    
    - name: Restart service
      service:
        name: myapp
        state: restarted
    
    # Play continues as long as < 25% of hosts fail
```

### Blocks

Blocks group tasks and provide error handling capabilities.

#### Basic Blocks

```yaml
# filepath: examples/blocks-basic.yml
---
- name: Basic blocks
  hosts: webservers
  become: yes
  
  tasks:
    - name: Install and configure web server
      block:
        - name: Install nginx
          apt:
            name: nginx
            state: present
        
        - name: Copy configuration
          template:
            src: nginx.conf.j2
            dest: /etc/nginx/nginx.conf
        
        - name: Start nginx
          service:
            name: nginx
            state: started
      
      when: ansible_os_family == "Debian"
```

#### Blocks with Rescue

```yaml
# filepath: examples/blocks-rescue.yml
---
- name: Blocks with rescue
  hosts: webservers
  become: yes
  
  tasks:
    - name: Deploy application with error recovery
      block:
        - name: Stop application
          service:
            name: myapp
            state: stopped
        
        - name: Deploy new version
          git:
            repo: https://github.com/user/app.git
            dest: /var/www/app
            version: "{{ app_version }}"
        
        - name: Run migrations
          command: /var/www/app/bin/migrate.sh
        
        - name: Start application
          service:
            name: myapp
            state: started
      
      rescue:
        - name: Rollback on failure
          debug:
            msg: "Deployment failed, rolling back..."
        
        - name: Restore previous version
          git:
            repo: https://github.com/user/app.git
            dest: /var/www/app
            version: "{{ previous_version }}"
        
        - name: Start application
          service:
            name: myapp
            state: started
        
        - name: Send failure notification
          mail:
            to: ops@example.com
            subject: "Deployment failed on {{ inventory_hostname }}"
            body: "Application deployment failed and was rolled back"
```

#### Blocks with Always

```yaml
# filepath: examples/blocks-always.yml
---
- name: Blocks with always
  hosts: webservers
  become: yes
  
  tasks:
    - name: Database maintenance
      block:
        - name: Put application in maintenance mode
          copy:
            content: "Under maintenance"
            dest: /var/www/html/maintenance.html
        
        - name: Stop application
          service:
            name: myapp
            state: stopped
        
        - name: Backup database
          command: /usr/local/bin/backup-db.sh
        
        - name: Run database maintenance
          command: /usr/local/bin/db-optimize.sh
        
        - name: Start application
          service:
            name: myapp
            state: started
      
      rescue:
        - name: Handle errors
          debug:
            msg: "Maintenance failed, attempting recovery"
        
        - name: Emergency start
          service:
            name: myapp
            state: started
          ignore_errors: yes
      
      always:
        - name: Remove maintenance mode (always runs)
          file:
            path: /var/www/html/maintenance.html
            state: absent
        
        - name: Log maintenance completion
          lineinfile:
            path: /var/log/maintenance.log
            line: "{{ ansible_date_time.iso8601 }} - Maintenance completed"
            create: yes
```

#### Complete Block Example

```yaml
# filepath: examples/blocks-complete.yml
---
- name: Complete block error handling
  hosts: webservers
  become: yes
  
  vars:
    app_version: "v2.0.0"
    backup_dir: "/backup"
  
  tasks:
    - name: Deploy with complete error handling
      block:
        # Main deployment tasks
        - name: Create backup directory
          file:
            path: "{{ backup_dir }}"
            state: directory
        
        - name: Backup current version
          archive:
            path: /var/www/app
            dest: "{{ backup_dir }}/app-{{ ansible_date_time.epoch }}.tar.gz"
        
        - name: Stop services
          service:
            name: "{{ item }}"
            state: stopped
          loop:
            - myapp
            - myapp-worker
        
        - name: Deploy new code
          git:
            repo: https://github.com/user/app.git
            dest: /var/www/app
            version: "{{ app_version }}"
        
        - name: Install dependencies
          command: /var/www/app/bin/install-deps.sh
          args:
            chdir: /var/www/app
        
        - name: Run database migrations
          command: /var/www/app/bin/migrate.sh
          args:
            chdir: /var/www/app
        
        - name: Start services
          service:
            name: "{{ item }}"
            state: started
          loop:
            - myapp
            - myapp-worker
        
        - name: Health check
          uri:
            url: http://localhost:8080/health
            status_code: 200
          retries: 5
          delay: 3
      
      rescue:
        # Run if any task in block fails
        - name: Log deployment failure
          debug:
            msg: "Deployment of {{ app_version }} failed on {{ inventory_hostname }}"
        
        - name: Find latest backup
          find:
            paths: "{{ backup_dir }}"
            patterns: "app-*.tar.gz"
          register: backups
        
        - name: Restore from backup
          unarchive:
            src: "{{ (backups.files | sort(attribute='mtime') | last).path }}"
            dest: /var/www/
            remote_src: yes
          when: backups.matched > 0
        
        - name: Start services after rollback
          service:
            name: "{{ item }}"
            state: started
          loop:
            - myapp
            - myapp-worker
          ignore_errors: yes
        
        - name: Send alert
          mail:
            to: ops@example.com
            subject: "ALERT: Deployment failed on {{ inventory_hostname }}"
            body: |
              Deployment of {{ app_version }} failed.
              System has been rolled back to previous version.
              Please investigate immediately.
      
      always:
        # Always run, regardless of success or failure
        - name: Ensure services are running
          service:
            name: "{{ item }}"
            state: started
          loop:
            - myapp
            - myapp-worker
          ignore_errors: yes
        
        - name: Log deployment attempt
          lineinfile:
            path: /var/log/deployments.log
            line: "{{ ansible_date_time.iso8601 }} - Attempted deployment of {{ app_version }} - Result: {{ ansible_failed_result | default('success') }}"
            create: yes
        
        - name: Clean up temp files
          file:
            path: /tmp/deployment-*
            state: absent
```

### Assert Module

```yaml
# filepath: examples/assert.yml
---
- name: Using assert for validation
  hosts: all
  
  tasks:
    - name: Gather system facts
      setup:
    
    - name: Assert system requirements
      assert:
        that:
          - ansible_memtotal_mb >= 2048
          - ansible_processor_vcpus >= 2
          - ansible_distribution in ['Ubuntu', 'Debian', 'CentOS', 'RedHat']
        fail_msg: "System does not meet minimum requirements"
        success_msg: "System meets all requirements"
    
    - name: Assert disk space
      shell: df -h / | tail -1 | awk '{print $5}' | sed 's/%//'
      register: disk_usage
      changed_when: false
    
    - name: Verify disk space
      assert:
        that:
          - disk_usage.stdout | int < 80
        fail_msg: "Disk usage is {{ disk_usage.stdout }}%, must be < 80%"
        success_msg: "Disk usage is acceptable"
    
    - name: Assert configuration
      assert:
        that:
          - app_version is defined
          - app_version is version('1.0.0', '>=')
          - environment in ['production', 'staging', 'development']
        quiet: true  # Don't show assertion details
```

### Fail Module

```yaml
# filepath: examples/fail.yml
---
- name: Using fail module
  hosts: all
  
  vars:
    required_packages:
      - nginx
      - postgresql
  
  tasks:
    - name: Check if running as root
      fail:
        msg: "This playbook must not be run as root"
      when: ansible_user_id == "root"
    
    - name: Verify environment variable
      fail:
        msg: "APP_ENV environment variable must be set"
      when: lookup('env', 'APP_ENV') | length == 0
    
    - name: Check package installation
      package_facts:
        manager: auto
    
    - name: Fail if required packages missing
      fail:
        msg: "Required package {{ item }} is not installed"
      when: item not in ansible_facts.packages
      loop: "{{ required_packages }}"
    
    - name: Conditional failure
      block:
        - name: Check application health
          uri:
            url: http://localhost:8080/health
          register: health_check
          failed_when: false
        
        - name: Fail if unhealthy
          fail:
            msg: |
              Application health check failed!
              Status: {{ health_check.status | default('unknown') }}
              Response: {{ health_check.content | default('no response') }}
          when: health_check.status != 200
```

### Changed_when

```yaml
# filepath: examples/changed-when.yml
---
- name: Control changed status
  hosts: all
  
  tasks:
    - name: Check file exists (never changed)
      stat:
        path: /etc/nginx/nginx.conf
      register: nginx_conf
      changed_when: false
    
    - name: Get service status (never changed)
      command: systemctl status nginx
      register: nginx_status
      changed_when: false
      failed_when: false
    
    - name: Run idempotent script
      script: /scripts/configure.sh
      register: script_result
      changed_when: "'Configuration updated' in script_result.stdout"
    
    - name: Backup database
      shell: |
        backup_file="/backup/db-$(date +%Y%m%d).sql"
        if [ ! -f "$backup_file" ]; then
          mysqldump mydb > "$backup_file"
          echo "BACKED_UP"
        else
          echo "ALREADY_EXISTS"
        fi
      register: backup_result
      changed_when: "'BACKED_UP' in backup_result.stdout"
```

## Hands-On Lab

### Lab 1: Service Management with Handlers

```yaml
# filepath: lab/service-management.yml
---
- name: Manage web services with handlers
  hosts: webservers
  become: yes
  
  vars:
    nginx_sites:
      - name: site1.com
        port: 80
        root: /var/www/site1
      - name: site2.com
        port: 8080
        root: /var/www/site2
  
  tasks:
    - name: Install nginx
      apt:
        name: nginx
        state: present
      notify: Start nginx
    
    - name: Create site directories
      file:
        path: "{{ item.root }}"
        state: directory
        owner: www-data
        group: www-data
      loop: "{{ nginx_sites }}"
    
    - name: Deploy site configurations
      template:
        src: templates/site.conf.j2
        dest: "/etc/nginx/sites-available/{{ item.name }}"
      loop: "{{ nginx_sites }}"
      notify:
        - Validate nginx config
        - Reload nginx
    
    - name: Enable sites
      file:
        src: "/etc/nginx/sites-available/{{ item.name }}"
        dest: "/etc/nginx/sites-enabled/{{ item.name }}"
        state: link
      loop: "{{ nginx_sites }}"
      notify: Reload nginx
    
    - name: Deploy index pages
      copy:
        content: "<h1>{{ item.name }}</h1>"
        dest: "{{ item.root }}/index.html"
      loop: "{{ nginx_sites }}"
  
  handlers:
    - name: Start nginx
      service:
        name: nginx
        state: started
        enabled: yes
    
    - name: Validate nginx config
      command: nginx -t
      changed_when: false
    
    - name: Reload nginx
      service:
        name: nginx
        state: reloaded
```

### Lab 2: Deployment with Rollback

```yaml
# filepath: lab/deployment-rollback.yml
---
- name: Application deployment with automatic rollback
  hosts: appservers
  become: yes
  
  vars:
    app_dir: /opt/myapp
    app_repo: https://github.com/user/myapp.git
    app_version: main
    health_check_url: http://localhost:8080/health
  
  tasks:
    - name: Deploy application
      block:
        - name: Get current version
          command: git -C {{ app_dir }} rev-parse HEAD
          register: current_version
          changed_when: false
          failed_when: false
        
        - name: Pull new code
          git:
            repo: "{{ app_repo }}"
            dest: "{{ app_dir }}"
            version: "{{ app_version }}"
          register: git_pull
        
        - name: Install dependencies
          command: npm install --production
          args:
            chdir: "{{ app_dir }}"
          when: git_pull.changed
        
        - name: Restart application
          service:
            name: myapp
            state: restarted
          when: git_pull.changed
        
        - name: Wait for application to start
          wait_for:
            port: 8080
            delay: 2
            timeout: 30
        
        - name: Health check
          uri:
            url: "{{ health_check_url }}"
            status_code: 200
            return_content: yes
          register: health
          retries: 5
          delay: 3
          until: health.status == 200
      
      rescue:
        - name: Rollback notification
          debug:
            msg: "Deployment failed, rolling back to {{ current_version.stdout }}"
        
        - name: Rollback to previous version
          command: git -C {{ app_dir }} reset --hard {{ current_version.stdout }}
          when: current_version.rc == 0
        
        - name: Restart after rollback
          service:
            name: myapp
            state: restarted
        
        - name: Verify rollback
          uri:
            url: "{{ health_check_url }}"
            status_code: 200
          retries: 3
          delay: 2
        
        - name: Fail playbook after rollback
          fail:
            msg: "Deployment failed and was rolled back"
      
      always:
        - name: Log deployment
          lineinfile:
            path: /var/log/deployments.log
            line: "{{ ansible_date_time.iso8601 }} - Deployment {{ 'failed' if ansible_failed_result is defined else 'succeeded' }}"
            create: yes
```

### Lab 3: Pre-flight Checks

```yaml
# filepath: lab/preflight-checks.yml
---
- name: Pre-flight system checks
  hosts: all
  gather_facts: yes
  
  tasks:
    - name: System requirements validation
      block:
        - name: Check OS version
          assert:
            that:
              - ansible_distribution in ['Ubuntu', 'Debian']
              - ansible_distribution_major_version | int >= 20
            fail_msg: "Unsupported OS: {{ ansible_distribution }} {{ ansible_distribution_version }}"
        
        - name: Check memory
          assert:
            that:
              - ansible_memtotal_mb >= 4096
            fail_msg: "Insufficient memory: {{ ansible_memtotal_mb }}MB (minimum 4096MB required)"
        
        - name: Check CPU cores
          assert:
            that:
              - ansible_processor_vcpus >= 2
            fail_msg: "Insufficient CPU cores: {{ ansible_processor_vcpus }} (minimum 2 required)"
        
        - name: Check disk space
          shell: df -BG / | tail -1 | awk '{print $4}' | sed 's/G//'
          register: available_space
          changed_when: false
        
        - name: Verify disk space
          assert:
            that:
              - available_space.stdout | int >= 10
            fail_msg: "Insufficient disk space: {{ available_space.stdout }}GB (minimum 10GB required)"
        
        - name: Check required ports
          wait_for:
            port: "{{ item }}"
            state: stopped
            timeout: 1
          loop:
            - 80
            - 443
            - 8080
          register: port_check
          failed_when: false
        
        - name: Verify ports are free
          assert:
            that:
              - item.failed
            fail_msg: "Port {{ item.item }} is already in use"
          loop: "{{ port_check.results }}"
          loop_control:
            label: "{{ item.item }}"
        
        - name: Check network connectivity
          uri:
            url: https://github.com
            timeout: 5
          register: network_check
          failed_when: false
        
        - name: Verify internet access
          assert:
            that:
              - network_check.status == 200
            fail_msg: "No internet connectivity"
      
      rescue:
        - name: Display failure summary
          debug:
            msg: |
              Pre-flight checks failed on {{ inventory_hostname }}
              Please fix the issues and try again
        
        - name: Fail playbook
          fail:
            msg: "Pre-flight checks failed"
```

### Lab 4: Database Maintenance

```yaml
# filepath: lab/database-maintenance.yml
---
- name: Database maintenance with error handling
  hosts: databases
  become: yes
  
  vars:
    db_name: myapp
    backup_dir: /backup/mysql
    maintenance_window: 3600  # 1 hour in seconds
  
  tasks:
    - name: Database maintenance
      block:
        - name: Create backup directory
          file:
            path: "{{ backup_dir }}"
            state: directory
            mode: '0700'
        
        - name: Check database size
          shell: |
            mysql -e "SELECT ROUND(SUM(data_length + index_length) / 1024 / 1024, 2) as size_mb
            FROM information_schema.tables
            WHERE table_schema = '{{ db_name }}';"
          register: db_size
          changed_when: false
        
        - name: Display database size
          debug:
            msg: "Database size: {{ db_size.stdout_lines[1] }} MB"
        
        - name: Put application in maintenance mode
          command: /usr/local/bin/maintenance-mode.sh on
          delegate_to: "{{ groups['webservers'][0] }}"
        
        - name: Backup database
          shell: |
            mysqldump {{ db_name }} | gzip > {{ backup_dir }}/{{ db_name }}-{{ ansible_date_time.epoch }}.sql.gz
          async: "{{ maintenance_window }}"
          poll: 10
        
        - name: Optimize tables
          shell: mysqlcheck -o {{ db_name }}
          async: "{{ maintenance_window }}"
          poll: 10
        
        - name: Analyze tables
          shell: mysqlcheck -a {{ db_name }}
          async: "{{ maintenance_window }}"
          poll: 10
      
      rescue:
        - name: Log maintenance error
          debug:
            msg: "Database maintenance failed: {{ ansible_failed_result }}"
        
        - name: Emergency backup check
          stat:
            path: "{{ backup_dir }}/{{ db_name }}-{{ ansible_date_time.epoch }}.sql.gz"
          register: emergency_backup
        
        - name: Alert if no backup
          fail:
            msg: "Critical: Maintenance failed and no backup was created!"
          when: not emergency_backup.stat.exists
      
      always:
        - name: Remove maintenance mode
          command: /usr/local/bin/maintenance-mode.sh off
          delegate_to: "{{ groups['webservers'][0] }}"
          ignore_errors: yes
        
        - name: Clean old backups (keep last 7 days)
          find:
            paths: "{{ backup_dir }}"
            patterns: "*.sql.gz"
            age: 7d
          register: old_backups
        
        - name: Delete old backups
          file:
            path: "{{ item.path }}"
            state: absent
          loop: "{{ old_backups.files }}"
          loop_control:
            label: "{{ item.path }}"
```

## Summary

In this lesson, you learned:
- ✅ Using handlers for service management
- ✅ Handler execution behavior and control
- ✅ Error handling with failed_when and ignore_errors
- ✅ Using blocks for error recovery
- ✅ Implementing rescue and always blocks
- ✅ Assertions and validations
- ✅ Complete deployment with rollback strategies

## Additional Resources

- [Ansible Handlers](https://docs.ansible.com/ansible/latest/user_guide/playbooks_handlers.html)
- [Error Handling](https://docs.ansible.com/ansible/latest/user_guide/playbooks_error_handling.html)
- [Blocks](https://docs.ansible.com/ansible/latest/user_guide/playbooks_blocks.html)

## Next Steps

Proceed to [Lesson 09: Ansible Roles](../09-ansible-roles/README.md) to learn how to organize playbooks into reusable roles.
