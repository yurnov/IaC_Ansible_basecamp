# Lesson 06: Templates with Jinja2

## Lesson Objectives
By the end of this lesson, you will be able to:
- Create dynamic configuration files using Jinja2 templates
- Use Jinja2 filters and tests
- Implement control structures in templates
- Use template inheritance and includes
- Apply best practices for template management

## Prerequisites
- Completed Lesson 05: Variables and Facts
- Basic understanding of template engines

## Duration
90 minutes

## Lesson Content

### What is Jinja2?

Jinja2 is a modern templating engine for Python. Ansible uses it to:
- Generate configuration files dynamically
- Transform data
- Apply logic to template rendering

### Template Module

```yaml
# filepath: examples/template-basic.yml
---
- name: Basic template usage
  hosts: webservers
  become: yes
  
  vars:
    server_name: www.example.com
    server_port: 80
  
  tasks:
    - name: Deploy configuration from template
      template:
        src: nginx.conf.j2
        dest: /etc/nginx/nginx.conf
        owner: root
        group: root
        mode: '0644'
        backup: yes
        validate: 'nginx -t -c %s'
      notify: Reload nginx
  
  handlers:
    - name: Reload nginx
      service:
        name: nginx
        state: reloaded
```

### Basic Template Syntax

#### Variables

```jinja2
{# filepath: examples/templates/variables.j2 #}
# Simple variable substitution
server_name: {{ server_name }}
port: {{ server_port }}

# Accessing dictionary values
database_host: {{ database.host }}
database_port: {{ database.port }}

# Accessing list items
first_user: {{ users[0] }}

# Default values
timeout: {{ timeout | default(30) }}

# Ansible facts
hostname: {{ ansible_hostname }}
ip_address: {{ ansible_default_ipv4.address }}
```

#### Comments

```jinja2
{# filepath: examples/templates/comments.j2 #}
{# This is a comment - won't appear in output #}

# This is a regular comment
server_name: {{ server_name }}

{#
Multi-line comment
These lines won't appear in the output
#}
```

### Control Structures

#### Conditionals (if/elif/else)

```jinja2
{# filepath: examples/templates/conditionals.j2 #}
# Nginx Configuration

{% if ssl_enabled %}
server {
    listen 443 ssl;
    server_name {{ server_name }};
    
    ssl_certificate {{ ssl_cert_path }};
    ssl_certificate_key {{ ssl_key_path }};
    
    {% if hsts_enabled %}
    add_header Strict-Transport-Security "max-age=31536000" always;
    {% endif %}
}
{% endif %}

server {
    listen {{ http_port }};
    server_name {{ server_name }};
    
    {% if ssl_enabled and redirect_to_https %}
    return 301 https://$server_name$request_uri;
    {% else %}
    root {{ document_root }};
    index index.html index.htm;
    {% endif %}
}

# Environment-specific configuration
{% if environment == 'production' %}
error_log /var/log/nginx/error.log warn;
access_log /var/log/nginx/access.log;
{% elif environment == 'staging' %}
error_log /var/log/nginx/error.log info;
access_log /var/log/nginx/access.log detailed;
{% else %}
error_log /var/log/nginx/error.log debug;
access_log /var/log/nginx/access.log;
{% endif %}
```

#### Loops (for)

```jinja2
{# filepath: examples/templates/loops.j2 #}
# /etc/hosts

127.0.0.1   localhost

# Web servers
{% for host in groups['webservers'] %}
{{ hostvars[host]['ansible_default_ipv4']['address'] }}    {{ host }}
{% endfor %}

# Database servers
{% for host in groups['databases'] %}
{{ hostvars[host]['ansible_default_ipv4']['address'] }}    {{ host }}
{% endfor %}

# Upstream configuration
upstream backend {
    {% for server in backend_servers %}
    server {{ server.host }}:{{ server.port }} weight={{ server.weight | default(1) }};
    {% endfor %}
}

# With conditions in loop
{% for user in users %}
{% if user.active %}
{{ user.name }}: {{ user.email }}
{% endif %}
{% endfor %}

# Loop variables
{% for item in items %}
Item {{ loop.index }}: {{ item.name }}
{% if loop.first %}(first){% endif %}
{% if loop.last %}(last){% endif %}
{% endfor %}
```

#### Loop Variables

```jinja2
{# filepath: examples/templates/loop-variables.j2 #}
{% for server in servers %}
# Server {{ loop.index }} of {{ loop.length }}
server_{{ loop.index0 }} = {{ server.host }}  {# 0-based index #}

{% if loop.first %}
# This is the first server
{% endif %}

{% if loop.last %}
# This is the last server
{% endif %}

{% if loop.previtem is defined %}
# Previous: {{ loop.previtem.host }}
{% endif %}

{% if loop.nextitem is defined %}
# Next: {{ loop.nextitem.host }}
{% endif %}

{% endfor %}
```

### Filters

Filters transform data within templates.

#### Common Filters

```jinja2
{# filepath: examples/templates/filters-common.j2 #}
# String filters
uppercase: {{ name | upper }}
lowercase: {{ name | lower }}
capitalize: {{ name | capitalize }}
title: {{ name | title }}

# Default values
port: {{ custom_port | default(80) }}
enabled: {{ ssl_enabled | default(false) }}

# List filters
packages: {{ packages | join(', ') }}
first_package: {{ packages | first }}
last_package: {{ packages | last }}
package_count: {{ packages | length }}
unique_items: {{ items | unique | join(', ') }}
sorted_items: {{ items | sort | join(', ') }}

# Dictionary filters
all_keys: {{ my_dict | list }}
all_values: {{ my_dict | dict2items }}

# Numeric filters
rounded: {{ 3.14159 | round(2) }}
absolute: {{ -42 | abs }}
random_number: {{ 100 | random }}

# Boolean filters
as_bool: {{ "yes" | bool }}
```

#### Type Conversion Filters

```jinja2
{# filepath: examples/templates/filters-conversion.j2 #}
# Convert to string
port_string: "{{ port | string }}"

# Convert to integer
port_int: {{ "8080" | int }}

# Convert to float
percentage: {{ "0.95" | float }}

# Convert to boolean
is_enabled: {{ "yes" | bool }}
is_disabled: {{ "false" | bool }}

# Convert to list
items: {{ "a,b,c" | split(',') }}

# JSON handling
json_data: {{ my_dict | to_json }}
yaml_data: {{ my_dict | to_yaml }}
pretty_json: {{ my_dict | to_nice_json }}
pretty_yaml: {{ my_dict | to_nice_yaml }}
```

#### File and Path Filters

```jinja2
{# filepath: examples/templates/filters-path.j2 #}
# Path manipulation
filename: {{ file_path | basename }}
directory: {{ file_path | dirname }}
without_extension: {{ file_path | splitext | first }}
extension: {{ file_path | splitext | last }}

# Examples:
# /var/www/app/index.html
basename: {{ "/var/www/app/index.html" | basename }}           # index.html
dirname: {{ "/var/www/app/index.html" | dirname }}             # /var/www/app
splitext_0: {{ "/var/www/app/index.html" | splitext | first }} # /var/www/app/index
splitext_1: {{ "/var/www/app/index.html" | splitext | last }}  # .html
```

#### Date and Time Filters

```jinja2
{# filepath: examples/templates/filters-datetime.j2 #}
# Date/time formatting
current_date: {{ ansible_date_time.date }}
current_time: {{ ansible_date_time.time }}
iso_timestamp: {{ ansible_date_time.iso8601 }}
epoch: {{ ansible_date_time.epoch }}

# Custom date format
formatted_date: {{ ansible_date_time.date | strftime('%B %d, %Y') }}
```

#### IP Address Filters

```jinja2
{# filepath: examples/templates/filters-ipaddr.j2 #}
# IP address manipulation
{% set ip = "192.168.1.100/24" %}

ip_address: {{ ip | ipaddr('address') }}        # 192.168.1.100
network: {{ ip | ipaddr('network') }}           # 192.168.1.0
netmask: {{ ip | ipaddr('netmask') }}           # 255.255.255.0
prefix: {{ ip | ipaddr('prefix') }}             # 24
broadcast: {{ ip | ipaddr('broadcast') }}       # 192.168.1.255

# IP version check
{% if ip | ipv4 %}
This is an IPv4 address
{% endif %}
```

#### Hash and Encryption Filters

```jinja2
{# filepath: examples/templates/filters-hash.j2 #}
# Password hashing
password_hash: {{ 'mypassword' | password_hash('sha512') }}
md5_hash: {{ 'data' | hash('md5') }}
sha1_hash: {{ 'data' | hash('sha1') }}
sha256_hash: {{ 'data' | hash('sha256') }}

# Base64 encoding
encoded: {{ 'hello' | b64encode }}
decoded: {{ 'aGVsbG8=' | b64decode }}
```

#### Custom Filters with Map and Select

```jinja2
{# filepath: examples/templates/filters-advanced.j2 #}
# Map filter - apply attribute to all items
{% set users = [{'name': 'alice', 'age': 30}, {'name': 'bob', 'age': 25}] %}
names: {{ users | map(attribute='name') | list }}

# Select filter - filter items by test
active_users: {{ users | selectattr('active', 'equalto', true) | list }}

# Reject filter - opposite of select
inactive_users: {{ users | rejectattr('active', 'equalto', true) | list }}

# Combine filters
sorted_names: {{ users | map(attribute='name') | sort | join(', ') }}

# Extract values
{% set servers = [{'host': 'web1', 'port': 80}, {'host': 'web2', 'port': 8080}] %}
all_ports: {{ servers | map(attribute='port') | list | join(', ') }}
```

### Tests

Tests check conditions and return boolean values.

```jinja2
{# filepath: examples/templates/tests.j2 #}
# Type tests
{% if variable is defined %}
Variable is defined
{% endif %}

{% if variable is undefined %}
Variable is not defined
{% endif %}

{% if value is none %}
Value is None
{% endif %}

{% if port is number %}
Port is a number
{% endif %}

{% if name is string %}
Name is a string
{% endif %}

{% if items is iterable %}
Items is iterable
{% endif %}

# Comparison tests
{% if value is equalto 5 %}
Value equals 5
{% endif %}

{% if path is file %}
Path is a file
{% endif %}

{% if path is directory %}
Path is a directory
{% endif %}

# Custom tests
{% if ip_address is match('192\.168\..*') %}
IP is in 192.168.0.0/16 network
{% endif %}

{% if version is version('2.0', '>=') %}
Version is 2.0 or higher
{% endif %}
```

### Whitespace Control

```jinja2
{# filepath: examples/templates/whitespace.j2 #}
# Without whitespace control
{% for item in items %}
{{ item }}
{% endfor %}

# With whitespace control (-)
{% for item in items -%}
{{ item }}
{% endfor %}

# Remove whitespace before
{%- for item in items %}
{{ item }}
{% endfor %}

# Remove whitespace before and after
{%- for item in items -%}
{{ item }}
{% endfor -%}
```

### Template Inheritance

#### Base Template

```jinja2
{# filepath: examples/templates/base.conf.j2 #}
# Base configuration for {{ app_name }}
# Generated: {{ ansible_date_time.iso8601 }}

{% block global %}
# Global settings
user {{ app_user }};
worker_processes {{ worker_processes }};
{% endblock %}

{% block events %}
events {
    worker_connections {{ worker_connections | default(1024) }};
}
{% endblock %}

{% block http %}
http {
    include mime.types;
    default_type application/octet-stream;
    
    {% block http_settings %}
    sendfile on;
    keepalive_timeout 65;
    {% endblock %}
    
    {% block servers %}
    # Server configurations will be added here
    {% endblock %}
}
{% endblock %}
```

#### Child Template

```jinja2
{# filepath: examples/templates/webserver.conf.j2 #}
{% extends "base.conf.j2" %}

{% block http_settings %}
{{ super() }}
gzip on;
gzip_types text/plain text/css application/json;
{% endblock %}

{% block servers %}
server {
    listen {{ http_port }};
    server_name {{ server_name }};
    root {{ document_root }};
    
    location / {
        try_files $uri $uri/ =404;
    }
}
{% endblock %}
```

### Template Includes

```jinja2
{# filepath: examples/templates/main.conf.j2 #}
# Main configuration

# Include common settings
{% include 'common_settings.j2' %}

# Server block
server {
    listen {{ port }};
    server_name {{ server_name }};
    
    # Include security headers
    {% include 'security_headers.j2' %}
    
    # Include SSL configuration
    {% if ssl_enabled %}
    {% include 'ssl_config.j2' %}
    {% endif %}
}
```

```jinja2
{# filepath: examples/templates/security_headers.j2 #}
add_header X-Frame-Options "SAMEORIGIN" always;
add_header X-Content-Type-Options "nosniff" always;
add_header X-XSS-Protection "1; mode=block" always;
```

### Macros

```jinja2
{# filepath: examples/templates/macros.j2 #}
{% macro render_server(name, host, port) -%}
server {
    server_name {{ name }};
    listen {{ port }};
    
    location / {
        proxy_pass http://{{ host }}:{{ port }};
    }
}
{%- endmacro %}

# Use the macro
{% for server in servers %}
{{ render_server(server.name, server.host, server.port) }}
{% endfor %}

# Macro with default values
{% macro database_config(host, port=5432, max_conn=100) -%}
host = {{ host }}
port = {{ port }}
max_connections = {{ max_conn }}
{%- endmacro %}

# Call with defaults
{{ database_config('db.example.com') }}

# Call with custom values
{{ database_config('db2.example.com', 3306, 200) }}
```

### Real-World Template Examples

#### Nginx Virtual Host

```jinja2
{# filepath: examples/templates/nginx-vhost.j2 #}
# {{ server_name }} virtual host configuration
# Managed by Ansible - DO NOT EDIT MANUALLY

{% if ssl_enabled %}
# HTTPS server
server {
    listen 443 ssl http2;
    server_name {{ server_name }};
    
    root {{ document_root }};
    index index.html index.htm index.php;
    
    # SSL configuration
    ssl_certificate {{ ssl_cert_path }};
    ssl_certificate_key {{ ssl_key_path }};
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers HIGH:!aNULL:!MD5;
    ssl_prefer_server_ciphers on;
    
    # Security headers
    add_header Strict-Transport-Security "max-age=31536000" always;
    add_header X-Frame-Options "SAMEORIGIN" always;
    add_header X-Content-Type-Options "nosniff" always;
    
    # Logging
    access_log /var/log/nginx/{{ server_name }}-access.log;
    error_log /var/log/nginx/{{ server_name }}-error.log;
    
    # Application
    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }
    
    {% if php_enabled %}
    # PHP handling
    location ~ \.php$ {
        fastcgi_pass unix:/var/run/php/php{{ php_version }}-fpm.sock;
        fastcgi_index index.php;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        include fastcgi_params;
    }
    {% endif %}
    
    # Deny access to hidden files
    location ~ /\. {
        deny all;
    }
}
{% endif %}

# HTTP server
server {
    listen 80;
    server_name {{ server_name }};
    
    {% if ssl_enabled and redirect_to_https %}
    # Redirect all HTTP to HTTPS
    return 301 https://$server_name$request_uri;
    {% else %}
    root {{ document_root }};
    index index.html index.htm index.php;
    
    access_log /var/log/nginx/{{ server_name }}-access.log;
    error_log /var/log/nginx/{{ server_name }}-error.log;
    
    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }
    {% endif %}
}
```

#### Application Configuration

```jinja2
{# filepath: examples/templates/app-config.ini.j2 #}
# {{ app_name }} Configuration
# Environment: {{ environment }}
# Generated: {{ ansible_date_time.iso8601 }}

[application]
name = {{ app_name }}
version = {{ app_version }}
environment = {{ environment }}
debug = {{ debug_mode | default(false) | lower }}

[server]
host = {{ app_host | default('0.0.0.0') }}
port = {{ app_port | default(8080) }}
workers = {{ app_workers | default(ansible_processor_vcpus) }}
threads = {{ app_threads | default(2) }}

[database]
host = {{ db_host }}
port = {{ db_port | default(5432) }}
name = {{ db_name }}
user = {{ db_user }}
password = {{ db_password }}
pool_size = {{ db_pool_size | default(10) }}
max_overflow = {{ db_max_overflow | default(20) }}

{% if redis_enabled %}
[redis]
host = {{ redis_host }}
port = {{ redis_port | default(6379) }}
db = {{ redis_db | default(0) }}
{% if redis_password is defined %}
password = {{ redis_password }}
{% endif %}
{% endif %}

[logging]
level = {{ log_level | upper }}
format = {{ log_format | default('%(asctime)s - %(name)s - %(levelname)s - %(message)s') }}
file = {{ log_file | default('/var/log/' + app_name + '.log') }}

{% if sentry_dsn is defined %}
[sentry]
dsn = {{ sentry_dsn }}
environment = {{ environment }}
{% endif %}

[security]
secret_key = {{ secret_key }}
allowed_hosts = {{ allowed_hosts | join(',') }}
{% if cors_enabled %}
cors_origins = {{ cors_origins | join(',') }}
{% endif %}
```

#### Systemd Service File

```jinja2
{# filepath: examples/templates/systemd-service.j2 #}
[Unit]
Description={{ app_name }} - {{ app_description }}
After=network.target
{% if db_service is defined %}
Requires={{ db_service }}
After={{ db_service }}
{% endif %}

[Service]
Type={{ service_type | default('simple') }}
User={{ app_user }}
Group={{ app_group | default(app_user) }}
WorkingDirectory={{ app_directory }}

{% if environment_vars is defined %}
# Environment variables
{% for key, value in environment_vars.items() %}
Environment="{{ key }}={{ value }}"
{% endfor %}
{% endif %}

# Security settings
PrivateTmp=true
NoNewPrivileges=true
{% if restrict_address_families %}
RestrictAddressFamilies=AF_UNIX AF_INET AF_INET6
{% endif %}

# Resource limits
LimitNOFILE={{ max_open_files | default(65536) }}
{% if memory_limit is defined %}
MemoryLimit={{ memory_limit }}
{% endif %}

# Start command
ExecStart={{ app_directory }}/bin/start.sh
ExecReload=/bin/kill -HUP $MAINPID
ExecStop=/bin/kill -TERM $MAINPID

# Restart policy
Restart={{ restart_policy | default('on-failure') }}
RestartSec={{ restart_sec | default(10) }}

[Install]
WantedBy=multi-user.target
```

#### Docker Compose

```jinja2
{# filepath: examples/templates/docker-compose.yml.j2 #}
# {{ app_name }} Docker Compose Configuration
# Generated: {{ ansible_date_time.iso8601 }}

version: '3.8'

services:
  web:
    image: {{ web_image }}:{{ web_version }}
    container_name: {{ app_name }}-web
    restart: unless-stopped
    ports:
      - "{{ web_port }}:80"
    environment:
      - APP_ENV={{ environment }}
      - APP_DEBUG={{ debug_mode | default(false) | lower }}
      - DB_HOST=db
      - DB_PORT=5432
      - DB_DATABASE={{ db_name }}
      - DB_USERNAME={{ db_user }}
      - DB_PASSWORD={{ db_password }}
    volumes:
      - ./app:/var/www/html
      - ./logs:/var/log/{{ app_name }}
    networks:
      - {{ app_name }}_network
    depends_on:
      - db
      {% if redis_enabled %}
      - redis
      {% endif %}
  
  db:
    image: postgres:{{ postgres_version | default('13') }}
    container_name: {{ app_name }}-db
    restart: unless-stopped
    environment:
      - POSTGRES_DB={{ db_name }}
      - POSTGRES_USER={{ db_user }}
      - POSTGRES_PASSWORD={{ db_password }}
    volumes:
      - db_data:/var/lib/postgresql/data
    networks:
      - {{ app_name }}_network
  
  {% if redis_enabled %}
  redis:
    image: redis:{{ redis_version | default('6-alpine') }}
    container_name: {{ app_name }}-redis
    restart: unless-stopped
    networks:
      - {{ app_name }}_network
  {% endif %}

networks:
  {{ app_name }}_network:
    driver: bridge

volumes:
  db_data:
```

## Hands-On Lab

### Lab 1: Basic Template

Create a simple Nginx configuration template:

```yaml
# filepath: lab/playbook-template-basic.yml
---
- name: Deploy Nginx configuration
  hosts: webservers
  become: yes
  
  vars:
    server_name: example.com
    server_port: 80
    document_root: /var/www/html
    
  tasks:
    - name: Deploy nginx config from template
      template:
        src: templates/nginx-simple.conf.j2
        dest: /etc/nginx/sites-available/{{ server_name }}
        backup: yes
      notify: Reload nginx
    
    - name: Enable site
      file:
        src: /etc/nginx/sites-available/{{ server_name }}
        dest: /etc/nginx/sites-enabled/{{ server_name }}
        state: link
      notify: Reload nginx
  
  handlers:
    - name: Reload nginx
      service:
        name: nginx
        state: reloaded
```

```jinja2
{# filepath: lab/templates/nginx-simple.conf.j2 #}
server {
    listen {{ server_port }};
    server_name {{ server_name }};
    root {{ document_root }};
    index index.html index.htm;
    
    location / {
        try_files $uri $uri/ =404;
    }
}
```

### Lab 2: Advanced Template with Loops

```yaml
# filepath: lab/playbook-template-advanced.yml
---
- name: Generate hosts file
  hosts: localhost
  connection: local
  
  tasks:
    - name: Generate /etc/hosts from template
      template:
        src: templates/hosts.j2
        dest: /tmp/hosts
```

```jinja2
{# filepath: lab/templates/hosts.j2 #}
# /etc/hosts - Generated by Ansible
# Date: {{ ansible_date_time.iso8601 }}

127.0.0.1   localhost
::1         localhost ip6-localhost ip6-loopback

{% for group in groups %}
{% if group != 'ungrouped' and group != 'all' %}
# {{ group | upper }} servers
{% for host in groups[group] %}
{{ hostvars[host]['ansible_default_ipv4']['address'] | default('127.0.0.1') }}    {{ host }} {{ hostvars[host]['ansible_hostname'] | default(host) }}
{% endfor %}

{% endif %}
{% endfor %}
```

### Lab 3: Multi-Environment Configuration

```yaml
# filepath: lab/group_vars/production.yml
---
environment: production
app_port: 8080
workers: 4
debug_mode: false
log_level: warning
database:
  host: prod-db.example.com
  port: 5432
  pool_size: 20
```

```yaml
# filepath: lab/group_vars/staging.yml
---
environment: staging
app_port: 8081
workers: 2
debug_mode: true
log_level: debug
database:
  host: staging-db.example.com
  port: 5432
  pool_size: 10
```

```yaml
# filepath: lab/playbook-multienv.yml
---
- name: Deploy application configuration
  hosts: all
  
  tasks:
    - name: Deploy app config
      template:
        src: templates/app-config.j2
        dest: /tmp/{{ inventory_hostname }}-config.ini
      delegate_to: localhost
```

```jinja2
{# filepath: lab/templates/app-config.j2 #}
# Application Configuration
# Host: {{ inventory_hostname }}
# Environment: {{ environment }}

[app]
port = {{ app_port }}
workers = {{ workers }}
debug = {{ debug_mode | lower }}
log_level = {{ log_level }}

[database]
host = {{ database.host }}
port = {{ database.port }}
pool_size = {{ database.pool_size }}

[server]
hostname = {{ ansible_hostname }}
os = {{ ansible_distribution }} {{ ansible_distribution_version }}
cpu_cores = {{ ansible_processor_vcpus }}
memory_mb = {{ ansible_memtotal_mb }}
```

### Lab 4: Template with Filters

```jinja2
{# filepath: lab/templates/system-report.j2 #}
# System Report for {{ ansible_hostname }}
# Generated: {{ ansible_date_time.iso8601 }}

## System Information
- Hostname: {{ ansible_hostname | upper }}
- FQDN: {{ ansible_fqdn }}
- Operating System: {{ ansible_distribution }} {{ ansible_distribution_version }}
- Kernel: {{ ansible_kernel }}
- Architecture: {{ ansible_architecture }}

## Hardware
- CPU Cores: {{ ansible_processor_vcpus }}
- Memory: {{ (ansible_memtotal_mb / 1024) | round(2) }} GB
- Swap: {{ (ansible_swaptotal_mb / 1024) | round(2) }} GB

## Network
- Primary IP: {{ ansible_default_ipv4.address | default('N/A') }}
- All IPs: {{ ansible_all_ipv4_addresses | join(', ') }}
- Interfaces: {{ ansible_interfaces | join(', ') }}

## Disk Information
{% for device, info in ansible_devices.items() %}
{% if info.size != "0.00 Bytes" %}
- {{ device }}: {{ info.size }} ({{ info.model | default('Unknown') }})
{% endif %}
{% endfor %}

## Installed Packages (sample)
{% if packages is defined %}
{{ packages | sort | join('\n') }}
{% endif %}

---
Report generated at {{ ansible_date_time.date }} {{ ansible_date_time.time }}
```

## Best Practices

### 1. Template Organization

```
templates/
├── base/
│   ├── base.conf.j2
│   └── common_settings.j2
├── nginx/
│   ├── nginx.conf.j2
│   ├── vhost.conf.j2
│   └── ssl.conf.j2
├── application/
│   ├── app.ini.j2
│   └── env.j2
└── systemd/
    └── service.j2
```

### 2. Use Ansible Managed Comments

```jinja2
{# filepath: examples/ansible-managed.j2 #}
# {{ ansible_managed }}
# This file is managed by Ansible - DO NOT EDIT MANUALLY
```

### 3. Validate Templates Before Deployment

```yaml
# filepath: examples/validate-template.yml
---
- name: Deploy with validation
  hosts: webservers
  
  tasks:
    - name: Deploy nginx config
      template:
        src: nginx.conf.j2
        dest: /etc/nginx/nginx.conf
        validate: 'nginx -t -c %s'
      notify: Reload nginx
```

### 4. Use Default Values

```jinja2
{# filepath: examples/defaults.j2 #}
port: {{ http_port | default(80) }}
workers: {{ worker_processes | default(ansible_processor_vcpus) }}
timeout: {{ request_timeout | default(30) }}
```

### 5. Comment Your Templates

```jinja2
{# filepath: examples/commented-template.j2 #}
{# 
  Nginx configuration template
  Variables required:
  - server_name: Domain name for the virtual host
  - document_root: Root directory for static files
  - ssl_enabled: Boolean to enable/disable SSL
#}

server {
    {# Listen on standard HTTP port #}
    listen 80;
    
    {# Server name from variable #}
    server_name {{ server_name }};
    
    {% if ssl_enabled %}
    {# SSL configuration block #}
    listen 443 ssl;
    ssl_certificate {{ ssl_cert }};
    ssl_certificate_key {{ ssl_key }};
    {% endif %}
}
```

## Summary

In this lesson, you learned:
- ✅ Creating Jinja2 templates for dynamic configuration
- ✅ Using variables, filters, and tests
- ✅ Implementing control structures (if/for)
- ✅ Template inheritance and includes
- ✅ Real-world template examples
- ✅ Best practices for template management

## Additional Resources

- [Jinja2 Documentation](https://jinja.palletsprojects.com/)
- [Ansible Template Module](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/template_module.html)
- [Ansible Filters](https://docs.ansible.com/ansible/latest/user_guide/playbooks_filters.html)
- [Ansible Tests](https://docs.ansible.com/ansible/latest/user_guide/playbooks_tests.html)

## Next Steps

Proceed to [Lesson 07: Conditionals and Loops](../07-conditionals-loops/README.md) to learn advanced playbook control flow.
