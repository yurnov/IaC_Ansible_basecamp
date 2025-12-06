# Lesson 14: Best Practices and Advanced Topics

## Lesson Objectives
By the end of this lesson, you will be able to:
- Apply Ansible best practices
- Optimize playbook performance
- Implement advanced patterns
- Troubleshoot common issues
- Scale Ansible for large infrastructures

## Prerequisites
- Completed Lesson 13: CI/CD with Ansible
- Strong understanding of all previous lessons

## Duration
90 minutes

## Lesson Content

### Project Organization

#### Directory Structure

```
ansible-project/
├── ansible.cfg                 # Project configuration
├── requirements.yml            # Collections and roles
├── .gitignore                 # Git ignore patterns
├── .ansible-lint              # Linting rules
├── .yamllint                  # YAML linting rules
│
├── inventories/
│   ├── production/
│   │   ├── hosts.yml
│   │   ├── group_vars/
│   │   │   ├── all.yml
│   │   │   ├── webservers.yml
│   │   │   └── databases.yml
│   │   └── host_vars/
│   │       └── web01.yml
│   └── staging/
│       ├── hosts.yml
│       └── group_vars/
│           └── all.yml
│
├── playbooks/
│   ├── site.yml               # Main playbook
│   ├── webservers.yml
│   ├── databases.yml
│   └── maintenance/
│       ├── backup.yml
│       └── update.yml
│
├── roles/
│   ├── common/
│   ├── webserver/
│   ├── database/
│   └── monitoring/
│
├── collections/
│   └── ansible_collections/
│       └── mycompany/
│           └── infrastructure/
│
├── group_vars/
│   └── all.yml                # Global variables
│
├── host_vars/
│
├── filter_plugins/            # Custom filters
├── library/                   # Custom modules
├── module_utils/              # Shared module code
│
├── files/                     # Static files
├── templates/                 # Jinja2 templates
│
├── tests/
│   ├── integration/
│   └── unit/
│
└── docs/
    ├── README.md
    └── deployment.md
```

#### Configuration File

```ini
# filepath: examples/ansible.cfg
[defaults]
# Inventory
inventory = ./inventories/production
host_key_checking = False

# Performance
forks = 20
gathering = smart
fact_caching = jsonfile
fact_caching_connection = /tmp/ansible_facts
fact_caching_timeout = 86400

# Output
stdout_callback = yaml
bin_ansible_callbacks = True
callbacks_enabled = profile_tasks, timer

# Roles
roles_path = ./roles:~/.ansible/roles:/usr/share/ansible/roles

# Collections
collections_paths = ./collections:~/.ansible/collections:/usr/share/ansible/collections

# SSH
ssh_args = -o ControlMaster=auto -o ControlPersist=60s -o UserKnownHostsFile=/dev/null
pipelining = True
control_path = /tmp/ansible-%%h-%%p-%%r

# Logging
log_path = ./ansible.log

# Vault
vault_password_file = ./.vault_pass

[privilege_escalation]
become = False
become_method = sudo
become_user = root
become_ask_pass = False

[ssh_connection]
ssh_args = -C -o ControlMaster=auto -o ControlPersist=60s
retries = 3
```

### Performance Optimization

#### Strategy: Free

```yaml
# filepath: examples/performance-free-strategy.yml
---
- name: Fast parallel execution
  hosts: all
  strategy: free  # Don't wait for slowest host
  gather_facts: no
  
  tasks:
    - name: Quick task
      command: echo "{{ inventory_hostname }}"
      changed_when: false
```

#### Strategy: Linear with Forks

```yaml
# filepath: examples/performance-forks.yml
---
- name: Controlled parallel execution
  hosts: all
  strategy: linear
  gather_facts: yes
  
  # Run on 10 hosts at a time
  serial: 10
  
  tasks:
    - name: Deploy application
      include_role:
        name: deploy_app
```

#### Async Tasks

```yaml
# filepath: examples/performance-async.yml
---
- name: Async task execution
  hosts: all
  
  tasks:
    - name: Long running task
      command: /usr/local/bin/long-process.sh
      async: 3600  # Run for up to 1 hour
      poll: 0      # Fire and forget
      register: long_task
    
    - name: Continue with other tasks
      debug:
        msg: "Continuing while long task runs"
    
    - name: Check on long running task
      async_status:
        jid: "{{ long_task.ansible_job_id }}"
      register: job_result
      until: job_result.finished
      retries: 120
      delay: 30
```

#### Fact Caching

```yaml
# filepath: examples/performance-fact-caching.yml
---
- name: Use cached facts
  hosts: all
  gather_facts: no  # Skip gathering
  
  tasks:
    - name: Setup (only if needed)
      setup:
      when: ansible_facts == {}
    
    - name: Use facts
      debug:
        msg: "{{ ansible_distribution }}"
```

#### Mitogen Strategy

```ini
# filepath: examples/ansible-mitogen.cfg
[defaults]
strategy_plugins = /path/to/mitogen/ansible_mitogen/plugins/strategy
strategy = mitogen_linear

[ssh_connection]
# Mitogen is 1.25-7x faster than standard SSH
```

### Advanced Patterns

#### Dynamic Groups

```yaml
# filepath: examples/dynamic-groups.yml
---
- name: Create dynamic groups
  hosts: all
  gather_facts: yes
  
  tasks:
    - name: Group by OS family
      group_by:
        key: "os_{{ ansible_os_family }}"
    
    - name: Group by environment
      group_by:
        key: "env_{{ environment | default('unknown') }}"
    
    - name: Group by memory size
      group_by:
        key: "{{ 'memory_large' if ansible_memtotal_mb > 16384 else 'memory_small' }}"

- name: Configure Debian systems
  hosts: os_Debian
  tasks:
    - name: Debian-specific tasks
      debug:
        msg: "Configuring Debian system"

- name: Configure production systems
  hosts: env_production
  tasks:
    - name: Production-specific tasks
      debug:
        msg: "Configuring production system"
```

#### Delegation Patterns

```yaml
# filepath: examples/delegation-patterns.yml
---
- name: Delegation patterns
  hosts: webservers
  
  tasks:
    - name: Update load balancer
      community.general.haproxy:
        backend: app_backend
        host: "{{ inventory_hostname }}"
        state: disabled
      delegate_to: loadbalancer
      run_once: true
    
    - name: Deploy application
      include_role:
        name: deploy_app
    
    - name: Update monitoring
      uri:
        url: "http://monitoring/api/deploy"
        method: POST
        body_format: json
        body:
          host: "{{ inventory_hostname }}"
          version: "{{ app_version }}"
      delegate_to: localhost
    
    - name: Re-enable in load balancer
      community.general.haproxy:
        backend: app_backend
        host: "{{ inventory_hostname }}"
        state: enabled
      delegate_to: loadbalancer
```

#### Include vs Import

```yaml
# filepath: examples/include-vs-import.yml
---
- name: Understanding include vs import
  hosts: all
  
  tasks:
    # import_tasks: Static, processed at parse time
    - name: Import tasks (static)
      import_tasks: common-tasks.yml
      tags:
        - always
    
    # include_tasks: Dynamic, processed at runtime
    - name: Include tasks conditionally
      include_tasks: "{{ ansible_os_family }}.yml"
      when: ansible_os_family in ['Debian', 'RedHat']
    
    # import_role: Static
    - name: Import role
      import_role:
        name: common
    
    # include_role: Dynamic
    - name: Include role conditionally
      include_role:
        name: webserver
      when: "'webservers' in group_names"
```

#### Task Blocks for Error Recovery

```yaml
# filepath: examples/advanced-blocks.yml
---
- name: Advanced block usage
  hosts: all
  
  tasks:
    - name: Complex deployment with recovery
      block:
        - name: Create backup
          archive:
            path: /opt/app
            dest: /backup/app-{{ ansible_date_time.epoch }}.tar.gz
        
        - name: Stop application
          service:
            name: myapp
            state: stopped
        
        - name: Deploy new version
          unarchive:
            src: "{{ app_package }}"
            dest: /opt/app
        
        - name: Run migrations
          command: /opt/app/bin/migrate
          register: migration
        
        - name: Start application
          service:
            name: myapp
            state: started
      
      rescue:
        - name: Restore from backup
          unarchive:
            src: "/backup/app-{{ ansible_date_time.epoch }}.tar.gz"
            dest: /opt
            remote_src: yes
        
        - name: Start old version
          service:
            name: myapp
            state: started
        
        - name: Send alert
          community.general.mail:
            to: ops@example.com
            subject: "Deployment failed on {{ inventory_hostname }}"
            body: "{{ migration.stderr | default('Unknown error') }}"
      
      always:
        - name: Clean up temp files
          file:
            path: /tmp/deploy-*
            state: absent
        
        - name: Log deployment attempt
          lineinfile:
            path: /var/log/deployments.log
            line: "{{ ansible_date_time.iso8601 }} - {{ ansible_failed_result | default('success') }}"
            create: yes
```

### Security Best Practices

#### Least Privilege

```yaml
# filepath: examples/security-least-privilege.yml
---
- name: Least privilege principle
  hosts: all
  
  tasks:
    # Don't use become unless necessary
    - name: User-level task
      copy:
        src: user_file.txt
        dest: "~/file.txt"
    
    # Use become only when needed
    - name: System-level task
      apt:
        name: nginx
        state: present
      become: yes
      become_user: root  # Be explicit
    
    # Use specific user for application tasks
    - name: Application task
      command: /opt/app/bin/task
      become: yes
      become_user: appuser  # Not root
```

#### No Logging Sensitive Data

```yaml
# filepath: examples/security-no-log.yml
---
- name: Secure sensitive data
  hosts: all
  
  vars:
    db_password: "{{ vault_db_password }}"
  
  tasks:
    - name: Configure database
      template:
        src: db_config.j2
        dest: /etc/app/db.conf
        mode: '0600'
      no_log: true  # Don't log template content
    
    - name: Set application secrets
      lineinfile:
        path: /etc/app/secrets.env
        line: "{{ item.key }}={{ item.value }}"
        create: yes
        mode: '0600'
      loop:
        - { key: "DB_PASSWORD", value: "{{ db_password }}" }
        - { key: "API_KEY", value: "{{ api_key }}" }
      no_log: true
    
    - name: Debug without exposing secrets
      debug:
        msg: "Configuration updated ({{ item.key }})"
      loop:
        - { key: "DB_PASSWORD", value: "{{ db_password }}" }
        - { key: "API_KEY", value: "{{ api_key }}" }
      loop_control:
        label: "{{ item.key }}"  # Only show key, not value
```

#### Input Validation

```yaml
# filepath: examples/security-validation.yml
---
- name: Input validation
  hosts: all
  
  vars_prompt:
    - name: username
      prompt: "Enter username"
      private: no
    
    - name: email
      prompt: "Enter email"
      private: no
  
  tasks:
    - name: Validate username
      assert:
        that:
          - username is match('^[a-z][a-z0-9_-]{2,15}$')
          - username not in ['root', 'admin', 'system']
        fail_msg: "Invalid username format or reserved name"
    
    - name: Validate email
      assert:
        that:
          - email is match('^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$')
        fail_msg: "Invalid email format"
    
    - name: Sanitize input for shell
      set_fact:
        safe_username: "{{ username | regex_replace('[^a-z0-9_-]', '') }}"
    
    - name: Create user
      user:
        name: "{{ safe_username }}"
        comment: "{{ email }}"
        state: present
      become: yes
```

### Troubleshooting

#### Debug Strategies

```yaml
# filepath: examples/debug-strategies.yml
---
- name: Debugging techniques
  hosts: all
  
  tasks:
    - name: Display variable
      debug:
        var: my_variable
    
    - name: Display message
      debug:
        msg: "Value is {{ my_variable }}"
    
    - name: Display with verbosity control
      debug:
        msg: "Detailed debug info"
        verbosity: 2  # Only with -vv or higher
    
    - name: Show all variables for host
      debug:
        var: hostvars[inventory_hostname]
    
    - name: Show group membership
      debug:
        var: group_names
    
    - name: Conditional debug
      debug:
        msg: "This is a web server"
      when: "'webservers' in group_names"
    
    - name: Assert for validation
      assert:
        that:
          - my_variable is defined
          - my_variable | length > 0
        fail_msg: "Variable validation failed"
        success_msg: "Variable validation passed"
```

#### Using Tags for Debugging

```bash
# Run only specific tags
ansible-playbook site.yml --tags debug

# Skip specific tags
ansible-playbook site.yml --skip-tags slow

# List all tags
ansible-playbook site.yml --list-tags

# List tasks
ansible-playbook site.yml --list-tasks

# Start at specific task
ansible-playbook site.yml --start-at-task="Deploy application"

# Step through tasks
ansible-playbook site.yml --step
```

#### Common Issues and Solutions

```yaml
# filepath: examples/troubleshooting-common.yml
---
- name: Common issues and solutions
  hosts: all
  
  tasks:
    # Issue: SSH connection problems
    - name: Test SSH connectivity
      wait_for:
        host: "{{ inventory_hostname }}"
        port: 22
        timeout: 10
      delegate_to: localhost
    
    # Issue: Python not found
    - name: Ensure Python is installed
      raw: test -e /usr/bin/python3 || (apt-get update && apt-get install -y python3)
      become: yes
      changed_when: false
    
    # Issue: Sudo password required
    - name: Test sudo access
      command: sudo -n true
      changed_when: false
      failed_when: false
      register: sudo_test
    
    - name: Report sudo status
      debug:
        msg: "{{ 'Passwordless sudo available' if sudo_test.rc == 0 else 'Sudo requires password' }}"
    
    # Issue: Slow fact gathering
    - name: Minimal facts
      setup:
        gather_subset:
          - '!all'
          - '!min'
          - network
      when: fast_mode | default(false)
    
    # Issue: Idempotency problems
    - name: Idempotent command
      command: /usr/local/bin/setup.sh
      args:
        creates: /var/lib/app/.setup_complete
      changed_when: false
```

### Scaling Ansible

#### Pull Mode with ansible-pull

```bash
# Setup ansible-pull on managed nodes
ansible-pull \
  -U https://github.com/company/ansible-repo.git \
  -i localhost, \
  -e "environment=production" \
  site.yml

# Cron job for regular pulls
*/30 * * * * ansible-pull -U https://github.com/company/ansible-repo.git site.yml
```

#### Tower/AWX Integration

```yaml
# filepath: examples/tower-workflow.yml
---
- name: AWX/Tower workflow
  hosts: localhost
  connection: local
  
  tasks:
    - name: Launch job template
      awx.awx.job_launch:
        job_template: "Deploy Application"
        extra_vars:
          environment: production
          version: 1.2.3
      register: job
    
    - name: Wait for job
      awx.awx.job_wait:
        job_id: "{{ job.id }}"
        timeout: 600
```

#### Inventory Plugins for Scale

```yaml
# filepath: examples/aws_ec2_inventory.yml
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
  - key: tags.Environment
    prefix: env
  - key: placement.availability_zone
    prefix: az

hostnames:
  - tag:Name
  - private-ip-address

compose:
  ansible_host: public_ip_address
  ansible_user: ubuntu
```

## Hands-On Lab

### Lab 1: Optimize a Playbook

```yaml
# filepath: lab/optimize-playbook.yml
---
# Before optimization
- name: Slow playbook
  hosts: all
  gather_facts: yes
  
  tasks:
    - name: Install packages one by one
      apt:
        name: "{{ item }}"
        state: present
      loop:
        - nginx
        - vim
        - git
        - curl
      become: yes

# After optimization
- name: Optimized playbook
  hosts: all
  gather_facts: no  # Only gather when needed
  strategy: free    # Don't wait for slowest host
  
  tasks:
    - name: Gather minimal facts
      setup:
        gather_subset:
          - '!all'
          - '!min'
          - distribution
      when: ansible_facts == {}
    
    - name: Install all packages at once
      apt:
        name:
          - nginx
          - vim
          - git
          - curl
        state: present
        update_cache: yes
        cache_valid_time: 3600
      become: yes
      async: 300
      poll: 0
      register: install_task
    
    - name: Wait for installation
      async_status:
        jid: "{{ install_task.ansible_job_id }}"
      register: job_result
      until: job_result.finished
      retries: 30
      delay: 10
```

### Lab 2: Complete Production-Ready Playbook

```yaml
# filepath: lab/production-playbook.yml
---
- name: Production deployment
  hosts: webservers
  become: yes
  serial: "25%"
  max_fail_percentage: 10
  
  vars:
    app_version: "{{ lookup('env', 'APP_VERSION') }}"
    deployment_id: "{{ ansible_date_time.epoch }}"
  
  pre_tasks:
    - name: Validate deployment
      assert:
        that:
          - app_version is defined
          - app_version is version('1.0.0', '>=')
        fail_msg: "Invalid app version"
    
    - name: Check system resources
      assert:
        that:
          - ansible_memfree_mb > 1024
          - ansible_mounts | selectattr('mount', 'equalto', '/') | map(attribute='size_available') | first | int > 5000000000
        fail_msg: "Insufficient system resources"
  
  tasks:
    - name: Deployment
      block:
        - name: Remove from load balancer
          uri:
            url: "http://{{ loadbalancer }}/api/remove/{{ inventory_hostname }}"
            method: POST
          delegate_to: localhost
        
        - name: Backup current version
          archive:
            path: /opt/app
            dest: "/backup/app-{{ deployment_id }}.tar.gz"
        
        - name: Deploy application
          include_role:
            name: deploy_app
          vars:
            version: "{{ app_version }}"
        
        - name: Health check
          uri:
            url: "http://localhost:8080/health"
            status_code: 200
            return_content: yes
          register: health
          retries: 10
          delay: 5
          until: health.status == 200
        
        - name: Verify version
          assert:
            that:
              - health.json.version == app_version
            fail_msg: "Version mismatch after deployment"
        
        - name: Add back to load balancer
          uri:
            url: "http://{{ loadbalancer }}/api/add/{{ inventory_hostname }}"
            method: POST
          delegate_to: localhost
      
      rescue:
        - name: Rollback
          include_tasks: rollback.yml
        
        - name: Notify failure
          community.general.mail:
            to: ops@example.com
            subject: "Deployment failed: {{ inventory_hostname }}"
            body: "Deployment of {{ app_version }} failed and was rolled back"
        
        - name: Fail deployment
          fail:
            msg: "Deployment failed, rolled back"
      
      always:
        - name: Log deployment
          lineinfile:
            path: /var/log/deployments.log
            line: "{{ ansible_date_time.iso8601 }} - {{ app_version }} - {{ 'FAILED' if ansible_failed_result is defined else 'SUCCESS' }}"
            create: yes
  
  post_tasks:
    - name: Clean old backups
      find:
        paths: /backup
        patterns: "app-*.tar.gz"
        age: 30d
      register: old_backups
    
    - name: Remove old backups
      file:
        path: "{{ item.path }}"
        state: absent
      loop: "{{ old_backups.files }}"
```

## Best Practices Checklist

### Development
- [ ] Use version control for all Ansible code
- [ ] Follow consistent naming conventions
- [ ] Document roles and playbooks
- [ ] Use meaningful task names
- [ ] Implement proper error handling
- [ ] Write idempotent tasks
- [ ] Use `changed_when` and `failed_when` appropriately

### Security
- [ ] Use Ansible Vault for secrets
- [ ] Implement least privilege
- [ ] Use `no_log` for sensitive data
- [ ] Validate all inputs
- [ ] Keep Ansible and collections updated
- [ ] Use secure SSH configurations
- [ ] Audit playbook runs

### Performance
- [ ] Use fact caching
- [ ] Gather only needed facts
- [ ] Use async for long tasks
- [ ] Optimize loops
- [ ] Use appropriate strategies
- [ ] Enable SSH pipelining
- [ ] Use `serial` for controlled rollouts

### Testing
- [ ] Syntax check all playbooks
- [ ] Use ansible-lint
- [ ] Implement check mode support
- [ ] Write integration tests
- [ ] Test in staging before production
- [ ] Automate testing in CI/CD

### Operations
- [ ] Use separate inventories per environment
- [ ] Implement proper logging
- [ ] Monitor playbook execution
- [ ] Have rollback procedures
- [ ] Document deployment processes
- [ ] Regular backup of Ansible code

## Summary

In this lesson, you learned:
- ✅ Ansible best practices
- ✅ Performance optimization techniques
- ✅ Advanced patterns and strategies
- ✅ Security best practices
- ✅ Troubleshooting techniques
- ✅ Scaling Ansible for large environments

## Additional Resources

- [Ansible Best Practices](https://docs.ansible.com/ansible/latest/user_guide/playbooks_best_practices.html)
- [Ansible Performance Tuning](https://www.ansible.com/blog/ansible-performance-tuning)
- [Ansible Security Automation](https://www.ansible.com/use-cases/security-automation)
- [Ansible Galaxy](https://galaxy.ansible.com/)
- [Ansible Community](https://forum.ansible.com/)

## Course Completion

Congratulations! You have completed the Ansible Basecamp course. You now have the skills to:
- Write and organize Ansible playbooks
- Manage infrastructure with roles and collections
- Implement security best practices
- Build CI/CD pipelines
- Scale Ansible for production use

### Next Steps
- Build real-world projects
- Contribute to Ansible community
- Explore Ansible Tower/AWX
- Stay updated with Ansible releases
- Share knowledge with others

Thank you for completing this course!
