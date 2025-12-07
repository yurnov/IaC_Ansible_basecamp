# Lesson 09: Ansible Roles

## Lesson Objectives
By the end of this lesson, you will be able to:
- Understand role structure and purpose
- Create and organize roles
- Use role dependencies
- Share roles using Ansible Galaxy
- Apply role best practices

## Prerequisites
- Completed Lesson 08: Handlers and Error Handling
- Understanding of playbook organization

## Duration
90 minutes

## Lesson Content

### What are Roles?

Roles are a way to organize playbooks into reusable components with a standardized structure.

#### Benefits of Roles
- **Reusability**: Share across projects
- **Organization**: Clear structure
- **Modularity**: Independent components
- **Collaboration**: Easy to share and maintain
- **Testing**: Easier to test individual components

### Role Directory Structure

```
roles/
└── rolename/
    ├── tasks/
    │   └── main.yml          # Main task list
    ├── handlers/
    │   └── main.yml          # Handler definitions
    ├── templates/
    │   └── config.j2         # Jinja2 templates
    ├── files/
    │   └── script.sh         # Static files
    ├── vars/
    │   └── main.yml          # Role variables (high precedence)
    ├── defaults/
    │   └── main.yml          # Default variables (low precedence)
    ├── meta/
    │   └── main.yml          # Role metadata and dependencies
    ├── library/
    │   └── custom_module.py  # Custom modules
    ├── module_utils/
    │   └── helper.py         # Module utilities
    └── README.md             # Role documentation
```

### Creating a Role

#### Using ansible-galaxy

```bash
# Create role structure
ansible-galaxy init myrole

# Create role in specific directory
ansible-galaxy init roles/myrole

# Create role with specific structure
ansible-galaxy init --init-path roles/ webserver
```

#### Manual Role Creation

```bash
mkdir -p roles/webserver/{tasks,handlers,templates,files,vars,defaults,meta}
touch roles/webserver/{tasks,handlers,vars,defaults,meta}/main.yml
```

### Simple Role Example

#### webserver Role

```yaml
# filepath: examples/roles/webserver/defaults/main.yml
---
# Default variables for webserver role
http_port: 80
https_port: 443
document_root: /var/www/html
server_admin: admin@example.com
enable_ssl: false
```

```yaml
# filepath: examples/roles/webserver/vars/main.yml
---
# Role variables (higher precedence than defaults)
nginx_package: nginx
nginx_service: nginx
config_dir: /etc/nginx
```

```yaml
# filepath: examples/roles/webserver/tasks/main.yml
---
# Main tasks for webserver role
- name: Install nginx
  apt:
    name: "{{ nginx_package }}"
    state: present
    update_cache: yes

- name: Create document root
  file:
    path: "{{ document_root }}"
    state: directory
    owner: www-data
    group: www-data
    mode: '0755'

- name: Deploy nginx configuration
  template:
    src: nginx.conf.j2
    dest: "{{ config_dir }}/nginx.conf"
    validate: 'nginx -t -c %s'
  notify: Reload nginx

- name: Deploy site configuration
  template:
    src: site.conf.j2
    dest: "{{ config_dir }}/sites-available/default"
  notify: Reload nginx

- name: Ensure nginx is started and enabled
  service:
    name: "{{ nginx_service }}"
    state: started
    enabled: yes
```

```yaml
# filepath: examples/roles/webserver/handlers/main.yml
---
# Handlers for webserver role
- name: Reload nginx
  service:
    name: "{{ nginx_service }}"
    state: reloaded

- name: Restart nginx
  service:
    name: "{{ nginx_service }}"
    state: restarted
```

```jinja2
{# filepath: examples/roles/webserver/templates/nginx.conf.j2 #}
user www-data;
worker_processes auto;
pid /run/nginx.pid;

events {
    worker_connections 768;
}

http {
    sendfile on;
    tcp_nopush on;
    tcp_nodelay on;
    keepalive_timeout 65;
    types_hash_max_size 2048;

    include /etc/nginx/mime.types;
    default_type application/octet-stream;

    access_log /var/log/nginx/access.log;
    error_log /var/log/nginx/error.log;

    gzip on;

    include /etc/nginx/conf.d/*.conf;
    include /etc/nginx/sites-enabled/*;
}
```

```jinja2
{# filepath: examples/roles/webserver/templates/site.conf.j2 #}
server {
    listen {{ http_port }};
    server_name _;
    root {{ document_root }};
    index index.html index.htm;

    {% if enable_ssl %}
    listen {{ https_port }} ssl;
    ssl_certificate {{ ssl_cert }};
    ssl_certificate_key {{ ssl_key }};
    {% endif %}

    location / {
        try_files $uri $uri/ =404;
    }
}
```

```yaml
# filepath: examples/roles/webserver/meta/main.yml
---
galaxy_info:
  author: Your Name
  description: Nginx web server role
  company: Your Company
  license: MIT
  min_ansible_version: 2.9
  platforms:
    - name: Ubuntu
      versions:
        - focal
        - jammy
    - name: Debian
      versions:
        - buster
        - bullseye
  galaxy_tags:
    - web
    - nginx
    - webserver

dependencies: []
```

### Using Roles in Playbooks

#### Basic Role Usage

```yaml
# filepath: examples/use-role-basic.yml
---
- name: Configure web servers
  hosts: webservers
  become: yes
  
  roles:
    - webserver
```

#### Role with Variables

```yaml
# filepath: examples/use-role-vars.yml
---
- name: Configure web servers with custom settings
  hosts: webservers
  become: yes
  
  roles:
    - role: webserver
      vars:
        http_port: 8080
        document_root: /var/www/mysite
        enable_ssl: true
```

#### Multiple Roles

```yaml
# filepath: examples/use-multiple-roles.yml
---
- name: Full stack deployment
  hosts: appservers
  become: yes
  
  roles:
    - common           # Common setup
    - security         # Security hardening
    - webserver        # Web server
    - application      # Application deployment
    - monitoring       # Monitoring setup
```

#### Role with Conditionals

```yaml
# filepath: examples/use-role-conditional.yml
---
- name: Conditional role application
  hosts: all
  become: yes
  
  roles:
    - role: webserver
      when: "'webservers' in group_names"
    
    - role: database
      when: "'databases' in group_names"
    
    - role: loadbalancer
      when: inventory_hostname in groups['loadbalancers']
```

#### Using import_role and include_role

```yaml
# filepath: examples/import-include-role.yml
---
- name: Dynamic role inclusion
  hosts: all
  become: yes
  
  tasks:
    # import_role: Static, processed at parse time
    - name: Import common role
      import_role:
        name: common
    
    # include_role: Dynamic, processed at runtime
    - name: Include web role conditionally
      include_role:
        name: webserver
      when: deploy_web | default(false)
    
    - name: Include role with specific tasks
      include_role:
        name: database
        tasks_from: backup
      when: backup_required | default(false)
```

### Role Dependencies

```yaml
# filepath: examples/roles/application/meta/main.yml
---
galaxy_info:
  author: Your Name
  description: Application deployment role
  min_ansible_version: 2.9

dependencies:
  - role: common
    vars:
      install_utilities: true
  
  - role: webserver
    vars:
      http_port: 8080
      enable_ssl: true
  
  - role: database
    vars:
      db_name: myapp
      db_user: appuser
    when: use_database | default(true)
```

### Complete Role Example: Application Deployment

```yaml
# filepath: examples/roles/myapp/defaults/main.yml
---
# Application defaults
app_name: myapp
app_version: latest
app_user: appuser
app_group: appuser
app_dir: /opt/myapp
app_port: 8080
app_env: production

# Repository settings
app_repo: https://github.com/user/myapp.git
app_repo_version: main

# Service settings
app_service_name: myapp
app_workers: 4

# Database settings
use_database: true
db_host: localhost
db_port: 5432
db_name: "{{ app_name }}"
db_user: "{{ app_name }}"

# Logging
log_level: info
log_dir: /var/log/{{ app_name }}
```

```yaml
# filepath: examples/roles/myapp/tasks/main.yml
---
# Include task files
- name: Include pre-installation tasks
  include_tasks: pre-install.yml

- name: Include installation tasks
  include_tasks: install.yml

- name: Include configuration tasks
  include_tasks: configure.yml

- name: Include service tasks
  include_tasks: service.yml

- name: Include post-installation tasks
  include_tasks: post-install.yml
```

```yaml
# filepath: examples/roles/myapp/tasks/pre-install.yml
---
- name: Create application user
  user:
    name: "{{ app_user }}"
    system: yes
    shell: /bin/bash
    home: "{{ app_dir }}"
    create_home: no

- name: Create application group
  group:
    name: "{{ app_group }}"
    system: yes

- name: Install system dependencies
  apt:
    name:
      - git
      - python3
      - python3-pip
      - python3-venv
    state: present
    update_cache: yes

- name: Create application directories
  file:
    path: "{{ item }}"
    state: directory
    owner: "{{ app_user }}"
    group: "{{ app_group }}"
    mode: '0755'
  loop:
    - "{{ app_dir }}"
    - "{{ log_dir }}"
    - "{{ app_dir }}/config"
    - "{{ app_dir }}/data"
```

```yaml
# filepath: examples/roles/myapp/tasks/install.yml
---
- name: Clone application repository
  git:
    repo: "{{ app_repo }}"
    dest: "{{ app_dir }}"
    version: "{{ app_repo_version }}"
  become_user: "{{ app_user }}"
  notify: Restart application

- name: Create Python virtual environment
  command: python3 -m venv {{ app_dir }}/venv
  args:
    creates: "{{ app_dir }}/venv"
  become_user: "{{ app_user }}"

- name: Install Python dependencies
  pip:
    requirements: "{{ app_dir }}/requirements.txt"
    virtualenv: "{{ app_dir }}/venv"
  become_user: "{{ app_user }}"
  notify: Restart application
```

```yaml
# filepath: examples/roles/myapp/tasks/configure.yml
---
- name: Deploy application configuration
  template:
    src: config.ini.j2
    dest: "{{ app_dir }}/config/app.conf"
    owner: "{{ app_user }}"
    group: "{{ app_group }}"
    mode: '0640'
  notify: Restart application

- name: Deploy environment file
  template:
    src: env.j2
    dest: "{{ app_dir }}/.env"
    owner: "{{ app_user }}"
    group: "{{ app_group }}"
    mode: '0600'
  notify: Restart application

- name: Set up log rotation
  template:
    src: logrotate.j2
    dest: /etc/logrotate.d/{{ app_name }}
    mode: '0644'
```

```yaml
# filepath: examples/roles/myapp/tasks/service.yml
---
- name: Deploy systemd service file
  template:
    src: systemd.service.j2
    dest: /etc/systemd/system/{{ app_service_name }}.service
    mode: '0644'
  notify:
    - Reload systemd
    - Restart application

- name: Enable and start application service
  service:
    name: "{{ app_service_name }}"
    enabled: yes
    state: started
```

```yaml
# filepath: examples/roles/myapp/tasks/post-install.yml
---
- name: Run database migrations
  command: "{{ app_dir }}/venv/bin/python {{ app_dir }}/manage.py migrate"
  become_user: "{{ app_user }}"
  when: use_database
  run_once: true

- name: Collect static files
  command: "{{ app_dir }}/venv/bin/python {{ app_dir }}/manage.py collectstatic --noinput"
  become_user: "{{ app_user }}"
  run_once: true

- name: Health check
  uri:
    url: "http://localhost:{{ app_port }}/health"
    status_code: 200
  retries: 5
  delay: 3
```

```yaml
# filepath: examples/roles/myapp/handlers/main.yml
---
- name: Reload systemd
  systemd:
    daemon_reload: yes

- name: Restart application
  service:
    name: "{{ app_service_name }}"
    state: restarted

- name: Reload application
  service:
    name: "{{ app_service_name }}"
    state: reloaded
```

### Ansible Galaxy

#### Searching for Roles

```bash
# Search for roles
ansible-galaxy search nginx

# Search with specific criteria
ansible-galaxy search --author geerlingguy nginx

# Get role information
ansible-galaxy info geerlingguy.nginx
```

#### Installing Roles

```bash
# Install role from Galaxy
ansible-galaxy install geerlingguy.nginx

# Install to specific directory
ansible-galaxy install geerlingguy.nginx -p ./roles

# Install specific version
ansible-galaxy install geerlingguy.nginx,2.8.0

# Install from GitHub
ansible-galaxy install git+https://github.com/user/role.git

# Install from requirements file
ansible-galaxy install -r requirements.yml
```

#### Requirements File

```yaml
# filepath: examples/requirements.yml
---
# From Galaxy
- name: geerlingguy.nginx
  version: 2.8.0

- name: geerlingguy.postgresql
  version: 3.2.0

# From GitHub
- src: https://github.com/user/custom-role.git
  version: main
  name: custom-role

# From Git with specific version
- src: git+https://github.com/user/another-role.git
  version: v1.0.0
  name: another-role

# From local path
- src: /path/to/local/role
  name: local-role
```

#### Publishing Roles to Galaxy

```bash
# Login to Galaxy
ansible-galaxy login

# Import role from GitHub
ansible-galaxy import username role-name

# Create role for Galaxy
ansible-galaxy init --init-path roles/ galaxy-role
```

## Hands-On Lab

### Lab 1: Create a Common Role

```bash
# Create role structure
ansible-galaxy init roles/common
```

```yaml
# filepath: lab/roles/common/defaults/main.yml
---
timezone: UTC
ntp_servers:
  - 0.pool.ntp.org
  - 1.pool.ntp.org

common_packages:
  - vim
  - curl
  - wget
  - git
  - htop
  - tree
```

```yaml
# filepath: lab/roles/common/tasks/main.yml
---
- name: Update package cache
  apt:
    update_cache: yes
    cache_valid_time: 3600

- name: Install common packages
  apt:
    name: "{{ common_packages }}"
    state: present

- name: Set timezone
  timezone:
    name: "{{ timezone }}"

- name: Configure NTP
  template:
    src: ntp.conf.j2
    dest: /etc/ntp.conf
  notify: Restart NTP

- name: Ensure NTP is running
  service:
    name: ntp
    state: started
    enabled: yes
```

```yaml
# filepath: lab/roles/common/handlers/main.yml
---
- name: Restart NTP
  service:
    name: ntp
    state: restarted
```

### Lab 2: Create an Application Role

```bash
ansible-galaxy init roles/webapp
```

```yaml
# filepath: lab/roles/webapp/defaults/main.yml
---
webapp_name: mywebapp
webapp_version: latest
webapp_port: 5000
webapp_dir: /opt/{{ webapp_name }}
webapp_user: webapp
webapp_repo: https://github.com/user/webapp.git
```

```yaml
# filepath: lab/roles/webapp/meta/main.yml
---
dependencies:
  - role: common
    vars:
      timezone: America/New_York
  
  - role: python
    vars:
      python_version: "3.10"
```

```yaml
# filepath: lab/roles/webapp/tasks/main.yml
---
- name: Create webapp user
  user:
    name: "{{ webapp_user }}"
    system: yes
    create_home: no

- name: Create application directory
  file:
    path: "{{ webapp_dir }}"
    state: directory
    owner: "{{ webapp_user }}"
    mode: '0755'

- name: Clone webapp repository
  git:
    repo: "{{ webapp_repo }}"
    dest: "{{ webapp_dir }}"
    version: "{{ webapp_version }}"
  become_user: "{{ webapp_user }}"
  notify: Restart webapp

- name: Install webapp dependencies
  pip:
    requirements: "{{ webapp_dir }}/requirements.txt"
    virtualenv: "{{ webapp_dir }}/venv"
  become_user: "{{ webapp_user }}"

- name: Deploy webapp service
  template:
    src: webapp.service.j2
    dest: /etc/systemd/system/{{ webapp_name }}.service
  notify:
    - Reload systemd
    - Restart webapp

- name: Start webapp service
  service:
    name: "{{ webapp_name }}"
    state: started
    enabled: yes
```

### Lab 3: Complete Multi-Role Playbook

```yaml
# filepath: lab/site.yml
---
- name: Configure all servers
  hosts: all
  become: yes
  roles:
    - common

- name: Configure web servers
  hosts: webservers
  become: yes
  roles:
    - role: nginx
      vars:
        nginx_port: 80
        nginx_ssl_port: 443
    
    - role: webapp
      vars:
        webapp_port: 5000

- name: Configure database servers
  hosts: databases
  become: yes
  roles:
    - role: postgresql
      vars:
        postgresql_version: 14
        postgresql_databases:
          - name: webapp_db
            owner: webapp_user
```

### Lab 4: Role with Tags

```yaml
# filepath: lab/roles/maintenance/tasks/main.yml
---
- name: Update system packages
  apt:
    upgrade: dist
    update_cache: yes
  tags:
    - packages
    - update

- name: Clean package cache
  apt:
    autoclean: yes
    autoremove: yes
  tags:
    - cleanup
    - packages

- name: Backup configuration files
  archive:
    path: /etc
    dest: /backup/etc-{{ ansible_date_time.epoch }}.tar.gz
  tags:
    - backup

- name: Clean old logs
  find:
    paths: /var/log
    patterns: "*.log"
    age: 30d
  register: old_logs
  tags:
    - cleanup
    - logs

- name: Remove old logs
  file:
    path: "{{ item.path }}"
    state: absent
  loop: "{{ old_logs.files }}"
  tags:
    - cleanup
    - logs
```

Run with tags:
```bash
# Run only backup tasks
ansible-playbook site.yml --tags backup

# Run cleanup tasks
ansible-playbook site.yml --tags cleanup

# Skip package updates
ansible-playbook site.yml --skip-tags packages
```

## Best Practices

### 1. Role Organization
- Keep roles focused on single responsibility
- Use meaningful role names
- Document role variables and requirements
- Include example playbooks

### 2. Variable Management
- Use defaults for common values
- Use vars for role-specific constants
- Document required variables
- Provide sensible defaults

### 3. Dependencies
- Minimize role dependencies
- Document dependency requirements
- Version pin dependencies
- Test with and without dependencies

### 4. Testing
- Test roles independently
- Use Molecule for role testing
- Include integration tests
- Test on multiple platforms

### 5. Documentation
- Maintain comprehensive README
- Document all variables
- Include usage examples
- Keep changelog updated

## Summary

In this lesson, you learned:
- ✅ Role structure and organization
- ✅ Creating reusable roles
- ✅ Using role dependencies
- ✅ Working with Ansible Galaxy
- ✅ Best practices for role development

## Additional Resources

- [Ansible Roles](https://docs.ansible.com/ansible/latest/user_guide/playbooks_reuse_roles.html)
- [Ansible Galaxy](https://galaxy.ansible.com/)
- [Role Best Practices](https://docs.ansible.com/ansible/latest/user_guide/playbooks_best_practices.html)
- [Molecule Testing](https://molecule.readthedocs.io/)

## Next Steps

Proceed to [Lesson 10: Ansible Collections](../10-ansible-collections/README.md) to learn about the modern way to package and distribute Ansible content.

