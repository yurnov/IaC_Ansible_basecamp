# Lesson 01: YAML Fundamentals

## Lesson Objectives
By the end of this lesson, you will be able to:
- Understand YAML syntax and structure
- Write valid YAML files for Ansible
- Recognize common YAML pitfalls
- Use proper data types in YAML

## Prerequisites
- Completed Lesson 00: Prerequisites and Environment Setup

## Duration
45 minutes

## Lesson Content

### What is YAML?

YAML (YAML Ain't Markup Language) is a human-readable data serialization language. Ansible uses YAML for:
- Playbooks
- Variable files
- Inventory files (optional)
- Configuration files

### Basic YAML Syntax Rules

1. **Indentation**: Use spaces (not tabs), typically 2 spaces
2. **Key-Value Pairs**: `key: value`
3. **Lists**: Start with `-`
4. **Case Sensitive**: `Name` ≠ `name`
5. **File Extension**: `.yml` or `.yaml`

### Data Types

#### Strings

```yaml
# filepath: examples/strings.yml
# Simple strings (no quotes needed)
name: John Doe
city: New York

# Quoted strings (when needed)
message: "Hello: World"  # Colon requires quotes
path: '/home/user/file'  # Consistent style

# Multi-line strings
description: |
  This is a multi-line string.
  Each line is preserved.
  Perfect for scripts.

summary: >
  This is a folded string.
  Line breaks become spaces.
  Good for long descriptions.
```

#### Numbers

```yaml
# filepath: examples/numbers.yml
# Integers
port: 80
count: 42
negative: -10

# Floats
percentage: 98.6
pi: 3.14159

# Octal (leading zero)
file_mode: 0644

# Hexadecimal
color: 0xFF00AA
```

#### Booleans

```yaml
# filepath: examples/booleans.yml
# All these are valid boolean true
enabled: true
enabled: True
enabled: TRUE
enabled: yes
enabled: Yes
enabled: on

# All these are valid boolean false
disabled: false
disabled: False
disabled: FALSE
disabled: no
disabled: No
disabled: off
```

#### Lists

```yaml
# filepath: examples/lists.yml
# Simple list
fruits:
  - apple
  - banana
  - orange

# Inline list (flow style)
colors: [red, green, blue]

# List of dictionaries
users:
  - name: alice
    role: admin
  - name: bob
    role: developer
```

#### Dictionaries (Maps)

```yaml
# filepath: examples/dictionaries.yml
# Simple dictionary
person:
  name: John
  age: 30
  city: Boston

# Nested dictionaries
server:
  web:
    hostname: web01
    ip: 192.168.1.10
  database:
    hostname: db01
    ip: 192.168.1.20

# Inline dictionary (flow style)
coordinates: {x: 10, y: 20, z: 30}
```

### Complex Structures

```yaml
# filepath: examples/complex.yml
# Combining data types
application:
  name: MyApp
  version: 1.2.3
  enabled: true
  
  servers:
    - hostname: web01
      ip: 192.168.1.10
      ports: [80, 443]
      ssl_enabled: true
      
    - hostname: web02
      ip: 192.168.1.11
      ports: [80, 443]
      ssl_enabled: true
  
  database:
    host: db.example.com
    port: 5432
    credentials:
      username: dbuser
      password: "{{ vault_db_password }}"
    
  features:
    caching: true
    monitoring: true
    backup: false
```

### Ansible-Specific YAML Features

#### Comments

```yaml
# filepath: examples/comments.yml
# This is a comment
---
# Play definition
- name: Example playbook  # Inline comment
  hosts: webservers
  
  tasks:
    # This task installs nginx
    - name: Install nginx
      apt:
        name: nginx
        state: present
```

#### Document Markers

```yaml
# filepath: examples/document_markers.yml
---
# Start of document (optional but recommended)
- name: First play
  hosts: web

...
# End of document (optional)
```

#### Anchors and Aliases (Advanced)

```yaml
# filepath: examples/anchors.yml
---
# Define anchor
defaults: &default_settings
  timeout: 30
  retries: 3
  delay: 5

# Use alias
task1:
  <<: *default_settings
  name: Task 1

task2:
  <<: *default_settings
  name: Task 2
  timeout: 60  # Override specific value
```

### Common Pitfalls

#### 1. Indentation Errors

```yaml
# filepath: examples/pitfall_indentation.yml
# ❌ WRONG - Inconsistent indentation
tasks:
  - name: Task 1
     command: echo "hello"
   - name: Task 2
    command: echo "world"

# ✅ CORRECT - Consistent 2-space indentation
tasks:
  - name: Task 1
    command: echo "hello"
  - name: Task 2
    command: echo "world"
```

#### 2. Quotes Issues

```yaml
# filepath: examples/pitfall_quotes.yml
# ❌ WRONG - Unquoted colon
message: Error: Something went wrong

# ✅ CORRECT - Quoted when contains special chars
message: "Error: Something went wrong"

# ❌ WRONG - Unquoted starting special char
path: *important

# ✅ CORRECT - Quoted to prevent interpretation
path: "*important"
```

#### 3. Boolean Confusion

```yaml
# filepath: examples/pitfall_booleans.yml
# ❌ WRONG - String that looks like boolean
enabled: "yes"  # This is a string, not boolean

# ✅ CORRECT - Actual boolean
enabled: yes  # or true

# Be careful with these values as strings
country: "NO"  # Norway, not boolean false
answer: "ON"   # Not boolean true
```

#### 4. Tabs vs Spaces

```yaml
# filepath: examples/pitfall_tabs.yml
# ❌ WRONG - Using tabs (will cause errors)
tasks:
→ - name: Task  # → represents a tab

# ✅ CORRECT - Using spaces
tasks:
  - name: Task
```

### YAML Validation

#### Using yamllint

```bash
# Install yamllint
pip install yamllint

# Check a file
yamllint playbook.yml

# Create .yamllint config
cat > ~/.yamllint << 'EOF'
extends: default
rules:
  line-length:
    max: 120
  indentation:
    spaces: 2
EOF
```

#### Using Python

```python
# filepath: examples/validate.py
import yaml
import sys

def validate_yaml(file_path):
    try:
        with open(file_path, 'r') as file:
            yaml.safe_load(file)
        print(f"✅ {file_path} is valid YAML")
        return True
    except yaml.YAMLError as e:
        print(f"❌ {file_path} has errors:")
        print(e)
        return False

if __name__ == "__main__":
    validate_yaml(sys.argv[1])
```

#### Using Ansible

```bash
# Validate playbook syntax
ansible-playbook --syntax-check playbook.yml

# Dry run (check mode)
ansible-playbook --check playbook.yml
```

## Hands-On Lab

### Lab 1: Create Valid YAML Files

Create the following files and validate them:

1. **inventory.yml** - A simple inventory
```yaml
# filepath: lab/inventory.yml
---
all:
  children:
    webservers:
      hosts:
        web01:
          ansible_host: 192.168.1.10
        web02:
          ansible_host: 192.168.1.11
    
    databases:
      hosts:
        db01:
          ansible_host: 192.168.1.20
```

2. **variables.yml** - Variable definitions
```yaml
# filepath: lab/variables.yml
---
app_name: myapp
app_version: 1.0.0
app_port: 8080

app_users:
  - username: admin
    role: administrator
  - username: user1
    role: developer

database:
  host: localhost
  port: 5432
  name: myapp_db
```

3. Validate your files:
```bash
yamllint inventory.yml
yamllint variables.yml
```

### Lab 2: Fix YAML Errors

Fix the errors in this playbook:

```yaml
# filepath: lab/broken.yml
---
- name: Broken playbook
  hosts: webservers
  
  tasks:
  - name: Install packages
    apt:
    name: nginx
      state: present
    
   - name: Start service
     service:
       name: nginx
    state: started
    
  - name: Create file
    copy:
      content: Hello: World
      dest: /tmp/test.txt
```

<details>
<summary>Solution</summary>

```yaml
# filepath: lab/fixed.yml
---
- name: Fixed playbook
  hosts: webservers
  
  tasks:
    - name: Install packages
      apt:
        name: nginx
        state: present
    
    - name: Start service
      service:
        name: nginx
        state: started
    
    - name: Create file
      copy:
        content: "Hello: World"
        dest: /tmp/test.txt
```

</details>

### Lab 3: Convert JSON to YAML

Convert this JSON to YAML:

```json
{
  "users": [
    {
      "name": "alice",
      "groups": ["sudo", "docker"],
      "shell": "/bin/bash"
    },
    {
      "name": "bob",
      "groups": ["docker"],
      "shell": "/bin/zsh"
    }
  ]
}
```

<details>
<summary>Solution</summary>

```yaml
# filepath: lab/users.yml
---
users:
  - name: alice
    groups:
      - sudo
      - docker
    shell: /bin/bash
  
  - name: bob
    groups:
      - docker
    shell: /bin/zsh
```

</details>

## Summary

In this lesson, you learned:
- ✅ YAML syntax and structure
- ✅ Different data types in YAML
- ✅ Common pitfalls and how to avoid them
- ✅ How to validate YAML files
- ✅ Ansible-specific YAML features

## Additional Resources

- [YAML Official Documentation](https://yaml.org/)
- [YAML Lint Online Tool](http://www.yamllint.com/)
- [Ansible YAML Syntax](https://docs.ansible.com/ansible/latest/reference_appendices/YAMLSyntax.html)

## Next Steps

Proceed to [Lesson 02: Ansible Basics and Ad-Hoc Commands](../02-ansible-basics/README.md) to start using Ansible.
