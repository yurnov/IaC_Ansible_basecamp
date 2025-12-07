# Lesson 10: Ansible Collections

## Lesson Objectives
By the end of this lesson, you will be able to:
- Understand what collections are and why they exist
- Install and use collections
- Create custom collections
- Manage collection dependencies
- Use collections in playbooks and roles

## Prerequisites
- Completed Lesson 09: Ansible Roles
- Understanding of role structure

## Duration
75 minutes

## Lesson Content

### What are Collections?

Collections are a distribution format for Ansible content including:
- Modules
- Plugins
- Roles
- Playbooks
- Documentation

#### Benefits of Collections
- **Organization**: Group related content together
- **Versioning**: Independent version control
- **Distribution**: Easy sharing via Ansible Galaxy
- **Namespacing**: Avoid naming conflicts
- **Dependencies**: Manage dependencies explicitly

### Collection Structure

```
collection_namespace.collection_name/
├── docs/
├── galaxy.yml                 # Collection metadata
├── meta/
│   └── runtime.yml           # Runtime information
├── plugins/
│   ├── modules/              # Custom modules
│   ├── inventory/            # Inventory plugins
│   ├── callback/             # Callback plugins
│   ├── connection/           # Connection plugins
│   ├── filter/               # Filter plugins
│   ├── lookup/               # Lookup plugins
│   └── module_utils/         # Shared code
├── roles/                    # Roles
├── playbooks/                # Example playbooks
├── tests/                    # Tests
└── README.md                 # Documentation
```

### Using Collections

#### Installing Collections

```bash
# Install from Ansible Galaxy
ansible-galaxy collection install community.general

# Install specific version
ansible-galaxy collection install community.general:5.0.0

# Install to custom path
ansible-galaxy collection install community.general -p ./collections

# Install from requirements file
ansible-galaxy collection install -r requirements.yml

# Install from tarball
ansible-galaxy collection install namespace-collection-1.0.0.tar.gz

# Install from Git repository
ansible-galaxy collection install git+https://github.com/namespace/collection.git
```

#### Requirements File

```yaml
# filepath: examples/requirements.yml
---
collections:
  # From Galaxy
  - name: community.general
    version: ">=5.0.0"
  
  - name: ansible.posix
    version: 1.4.0
  
  - name: community.docker
    version: ">=3.0.0,<4.0.0"
  
  # From Git
  - name: https://github.com/namespace/collection.git
    type: git
    version: main
  
  # From local path
  - name: /path/to/local/collection
    type: dir
```

#### Listing Collections

```bash
# List installed collections
ansible-galaxy collection list

# List collections in specific path
ansible-galaxy collection list -p ./collections

# Show collection details
ansible-doc -t module community.general.docker_container
```

### Using Collections in Playbooks

#### Fully Qualified Collection Names (FQCN)

```yaml
# filepath: examples/fqcn-usage.yml
---
- name: Using FQCN
  hosts: all
  
  tasks:
    # Using FQCN (recommended)
    - name: Install package using community.general
      community.general.snap:
        name: code
        classic: yes
    
    - name: Manage Docker container
      community.docker.docker_container:
        name: myapp
        image: nginx:latest
        state: started
    
    - name: Manage systemd service
      ansible.posix.systemd:
        name: nginx
        state: started
        enabled: yes
```

#### Using Collections Keyword

```yaml
# filepath: examples/collections-keyword.yml
---
- name: Using collections keyword
  hosts: all
  collections:
    - community.general
    - community.docker
  
  tasks:
    # Can now use short names
    - name: Install snap package
      snap:
        name: code
        classic: yes
    
    - name: Manage Docker container
      docker_container:
        name: myapp
        image: nginx:latest
        state: started
```

#### Collections in Roles

```yaml
# filepath: examples/roles/myrole/meta/main.yml
---
collections:
  - community.general
  - community.docker
  - ansible.posix

dependencies: []
```

```yaml
# filepath: examples/roles/myrole/tasks/main.yml
---
# Tasks can use short names when collections defined in meta
- name: Install package
  snap:
    name: code
    classic: yes

- name: Manage container
  docker_container:
    name: myapp
    image: nginx:latest
```

### Common Collections

#### ansible.builtin

```yaml
# filepath: examples/builtin-collection.yml
---
- name: Using builtin modules
  hosts: all
  
  tasks:
    # These are from ansible.builtin (implicit)
    - name: Copy file
      ansible.builtin.copy:
        src: file.txt
        dest: /tmp/file.txt
    
    - name: Create user
      ansible.builtin.user:
        name: testuser
        state: present
    
    - name: Install package
      ansible.builtin.apt:
        name: nginx
        state: present
```

#### community.general

```yaml
# filepath: examples/community-general.yml
---
- name: Using community.general
  hosts: all
  
  tasks:
    - name: Manage snap packages
      community.general.snap:
        name: "{{ item }}"
        state: present
      loop:
        - code
        - kubectl
    
    - name: Install gem
      community.general.gem:
        name: bundler
        state: present
    
    - name: Manage npm packages
      community.general.npm:
        name: express
        global: yes
    
    - name: Manage Python packages with pipx
      community.general.pipx:
        name: ansible-lint
        state: present
```

#### ansible.posix

```yaml
# filepath: examples/ansible-posix.yml
---
- name: Using ansible.posix
  hosts: all
  
  tasks:
    - name: Manage SELinux
      ansible.posix.selinux:
        policy: targeted
        state: enforcing
    
    - name: Set file ACL
      ansible.posix.acl:
        path: /var/www
        entity: www-data
        etype: user
        permissions: rwx
        state: present
    
    - name: Synchronize files
      ansible.posix.synchronize:
        src: /src/directory
        dest: /dest/directory
```

#### community.docker

```yaml
# filepath: examples/community-docker.yml
---
- name: Using community.docker
  hosts: all
  
  tasks:
    - name: Create Docker network
      community.docker.docker_network:
        name: myapp_network
    
    - name: Create Docker volume
      community.docker.docker_volume:
        name: myapp_data
    
    - name: Run container
      community.docker.docker_container:
        name: webapp
        image: nginx:latest
        state: started
        ports:
          - "80:80"
        networks:
          - name: myapp_network
        volumes:
          - myapp_data:/usr/share/nginx/html
    
    - name: Execute command in container
      community.docker.docker_container_exec:
        container: webapp
        command: nginx -t
```

#### community.kubernetes

```yaml
# filepath: examples/community-kubernetes.yml
---
- name: Using community.kubernetes
  hosts: localhost
  connection: local
  
  tasks:
    - name: Create namespace
      community.kubernetes.k8s:
        name: myapp
        kind: Namespace
        state: present
    
    - name: Create deployment
      community.kubernetes.k8s:
        state: present
        definition:
          apiVersion: apps/v1
          kind: Deployment
          metadata:
            name: nginx
            namespace: myapp
          spec:
            replicas: 3
            selector:
              matchLabels:
                app: nginx
            template:
              metadata:
                labels:
                  app: nginx
              spec:
                containers:
                  - name: nginx
                    image: nginx:latest
    
    - name: Get pod information
      community.kubernetes.k8s_info:
        kind: Pod
        namespace: myapp
      register: pod_list
```

### Creating a Collection

#### Initialize Collection

```bash
# Create collection structure
ansible-galaxy collection init mycompany.mycollection

# Create in specific directory
ansible-galaxy collection init mycompany.mycollection --init-path ./collections
```

#### Collection Metadata

```yaml
# filepath: examples/mycompany/mycollection/galaxy.yml
---
namespace: mycompany
name: mycollection
version: 1.0.0
readme: README.md
authors:
  - Your Name <you@example.com>
description: Custom collection for company infrastructure
license:
  - MIT
license_file: LICENSE
tags:
  - infrastructure
  - automation
  - mycompany
dependencies:
  community.general: ">=5.0.0"
  ansible.posix: ">=1.4.0"
repository: https://github.com/mycompany/mycollection
documentation: https://docs.mycompany.com/ansible
homepage: https://www.mycompany.com
issues: https://github.com/mycompany/mycollection/issues
```

#### Runtime Configuration

```yaml
# filepath: examples/mycompany/mycollection/meta/runtime.yml
---
requires_ansible: ">=2.12.0"
plugin_routing:
  modules:
    old_module_name:
      redirect: mycompany.mycollection.new_module_name
      deprecation:
        removal_version: 2.0.0
        warning_text: Use new_module_name instead
```

#### Custom Module

```python
#!/usr/bin/python
# filepath: examples/mycompany/mycollection/plugins/modules/hello.py

from ansible.module_utils.basic import AnsibleModule

DOCUMENTATION = r'''
---
module: hello
short_description: Simple hello world module
description:
    - This module prints a hello message
options:
    name:
        description: Name to greet
        required: true
        type: str
author:
    - Your Name (@yourgithub)
'''

EXAMPLES = r'''
- name: Say hello
  mycompany.mycollection.hello:
    name: World
'''

RETURN = r'''
message:
    description: The greeting message
    type: str
    returned: always
    sample: "Hello, World!"
'''

def main():
    module = AnsibleModule(
        argument_spec=dict(
            name=dict(type='str', required=True)
        ),
        supports_check_mode=True
    )
    
    name = module.params['name']
    message = f"Hello, {name}!"
    
    result = dict(
        changed=False,
        message=message
    )
    
    module.exit_json(**result)

if __name__ == '__main__':
    main()
```

#### Custom Filter Plugin

```python
# filepath: examples/mycompany/mycollection/plugins/filter/custom_filters.py

class FilterModule:
    """Custom Jinja2 filters"""
    
    def filters(self):
        return {
            'reverse_string': self.reverse_string,
            'multiply': self.multiply,
            'to_json_sorted': self.to_json_sorted,
        }
    
    def reverse_string(self, text):
        """Reverse a string"""
        return text[::-1]
    
    def multiply(self, value, multiplier):
        """Multiply a value"""
        return value * multiplier
    
    def to_json_sorted(self, data):
        """Convert to JSON with sorted keys"""
        import json
        return json.dumps(data, sort_keys=True, indent=2)
```

#### Using Custom Collection

```yaml
# filepath: examples/use-custom-collection.yml
---
- name: Use custom collection
  hosts: localhost
  connection: local
  collections:
    - mycompany.mycollection
  
  tasks:
    - name: Use custom module
      hello:
        name: Ansible User
      register: greeting
    
    - name: Display greeting
      debug:
        var: greeting.message
    
    - name: Use custom filter
      debug:
        msg: "{{ 'Ansible' | reverse_string }}"
    
    - name: Use multiply filter
      debug:
        msg: "{{ 5 | multiply(3) }}"
```

### Building and Publishing Collections

#### Build Collection

```bash
# Build collection tarball
cd collections/ansible_collections/mycompany/mycollection
ansible-galaxy collection build

# This creates: mycompany-mycollection-1.0.0.tar.gz
```

#### Publish to Galaxy

```bash
# Login to Galaxy
ansible-galaxy login

# Publish collection
ansible-galaxy collection publish mycompany-mycollection-1.0.0.tar.gz

# Publish with API key
ansible-galaxy collection publish mycompany-mycollection-1.0.0.tar.gz --api-key=YOUR_API_KEY
```

#### Install Custom Collection

```bash
# Install from tarball
ansible-galaxy collection install mycompany-mycollection-1.0.0.tar.gz

# Install from Galaxy
ansible-galaxy collection install mycompany.mycollection
```

### Collection Dependencies

```yaml
# filepath: examples/collection-with-deps/galaxy.yml
---
namespace: mycompany
name: app
version: 1.0.0

dependencies:
  community.general: ">=5.0.0"
  community.docker: ">=3.0.0"
  ansible.posix: ">=1.4.0"
  mycompany.common: ">=1.0.0"
```

## Hands-On Lab

### Lab 1: Using Popular Collections

```yaml
# filepath: lab/requirements.yml
---
collections:
  - name: community.general
    version: ">=5.0.0"
  - name: community.docker
    version: ">=3.0.0"
  - name: ansible.posix
    version: ">=1.4.0"
```

```bash
# Install collections
ansible-galaxy collection install -r lab/requirements.yml
```

```yaml
# filepath: lab/use-collections.yml
---
- name: Using multiple collections
  hosts: localhost
  connection: local
  
  collections:
    - community.general
    - community.docker
  
  tasks:
    - name: Install snap packages
      snap:
        name: "{{ item }}"
        state: present
      loop:
        - code
        - kubectl
      become: yes
    
    - name: Create Docker network
      docker_network:
        name: app_network
        state: present
    
    - name: Run nginx container
      docker_container:
        name: web
        image: nginx:latest
        state: started
        networks:
          - name: app_network
        ports:
          - "8080:80"
```

### Lab 2: Create a Simple Collection

```bash
# Create collection structure
mkdir -p collections/ansible_collections/training
cd collections/ansible_collections/training
ansible-galaxy collection init demo
```

```yaml
# filepath: lab/collections/ansible_collections/training/demo/galaxy.yml
---
namespace: training
name: demo
version: 1.0.0
readme: README.md
authors:
  - Training User
description: Demo collection for training
license:
  - MIT
tags:
  - demo
  - training
```

```python
# filepath: lab/collections/ansible_collections/training/demo/plugins/modules/info.py
#!/usr/bin/python

from ansible.module_utils.basic import AnsibleModule
import platform

DOCUMENTATION = r'''
---
module: info
short_description: Get system information
description: Returns basic system information
options: {}
'''

def main():
    module = AnsibleModule(
        argument_spec=dict(),
        supports_check_mode=True
    )
    
    result = dict(
        changed=False,
        system=platform.system(),
        release=platform.release(),
        machine=platform.machine(),
        python_version=platform.python_version()
    )
    
    module.exit_json(**result)

if __name__ == '__main__':
    main()
```

```yaml
# filepath: lab/collections/ansible_collections/training/demo/roles/setup/tasks/main.yml
---
- name: Create demo directory
  file:
    path: /tmp/demo
    state: directory
    mode: '0755'

- name: Create demo file
  copy:
    content: "Hello from training.demo collection!"
    dest: /tmp/demo/info.txt
```

Build and test:

```bash
# Build collection
cd lab/collections/ansible_collections/training/demo
ansible-galaxy collection build

# Install collection
ansible-galaxy collection install training-demo-1.0.0.tar.gz
```

```yaml
# filepath: lab/test-collection.yml
---
- name: Test custom collection
  hosts: localhost
  connection: local
  
  tasks:
    - name: Use custom module
      training.demo.info:
      register: sys_info
    
    - name: Display system info
      debug:
        var: sys_info
    
    - name: Use collection role
      include_role:
        name: training.demo.setup
```

### Lab 3: Collection with Filters

```python
# filepath: lab/collections/ansible_collections/training/demo/plugins/filter/text_filters.py

class FilterModule:
    """Custom text filters"""
    
    def filters(self):
        return {
            'shout': self.shout,
            'whisper': self.whisper,
            'truncate': self.truncate,
        }
    
    def shout(self, text):
        """Convert to uppercase and add exclamation"""
        return f"{text.upper()}!"
    
    def whisper(self, text):
        """Convert to lowercase"""
        return text.lower()
    
    def truncate(self, text, length=10, suffix='...'):
        """Truncate text to specified length"""
        if len(text) <= length:
            return text
        return text[:length] + suffix
```

```yaml
# filepath: lab/test-filters.yml
---
- name: Test custom filters
  hosts: localhost
  connection: local
  
  vars:
    message: "Hello Ansible"
    long_text: "This is a very long message that needs to be truncated"
  
  tasks:
    - name: Use shout filter
      debug:
        msg: "{{ message | training.demo.shout }}"
    
    - name: Use whisper filter
      debug:
        msg: "{{ message | training.demo.whisper }}"
    
    - name: Use truncate filter
      debug:
        msg: "{{ long_text | training.demo.truncate(20, '...') }}"
```

### Lab 4: Multi-Collection Playbook

```yaml
# filepath: lab/multi-collection.yml
---
- name: Complex multi-collection deployment
  hosts: servers
  become: yes
  
  collections:
    - community.general
    - ansible.posix
    - community.docker
  
  tasks:
    - name: Set timezone using posix
      timezone:
        name: America/New_York
    
    - name: Install Docker using snap
      snap:
        name: docker
        state: present
    
    - name: Set SELinux policy
      selinux:
        policy: targeted
        state: permissive
      when: ansible_os_family == "RedHat"
    
    - name: Deploy containerized application
      docker_container:
        name: myapp
        image: myapp:latest
        state: started
        restart_policy: unless-stopped
        ports:
          - "80:8080"
        env:
          APP_ENV: production
          DB_HOST: "{{ db_host }}"
    
    - name: Configure firewall using firewalld
      firewalld:
        port: 80/tcp
        permanent: yes
        state: enabled
      notify: Reload firewalld
  
  handlers:
    - name: Reload firewalld
      systemd:
        name: firewalld
        state: reloaded
```

## Best Practices

### 1. Use FQCN
Always use Fully Qualified Collection Names for clarity:
```yaml
# Good
community.general.docker_container:

# Avoid
docker_container:
```

### 2. Pin Collection Versions
Specify versions in requirements:
```yaml
collections:
  - name: community.general
    version: ">=5.0.0,<6.0.0"
```

### 3. Document Collection Usage
Document which collections your playbooks/roles need:
```yaml
# requirements.yml at project root
```

### 4. Test Collections
Test your custom collections thoroughly:
```bash
ansible-test sanity
ansible-test units
ansible-test integration
```

### 5. Version Semantically
Follow semantic versioning for your collections:
- MAJOR.MINOR.PATCH
- Breaking changes: increment MAJOR
- New features: increment MINOR
- Bug fixes: increment PATCH

## Summary

In this lesson, you learned:
- ✅ Understanding Ansible collections
- ✅ Installing and using collections
- ✅ Creating custom collections
- ✅ Building and publishing collections
- ✅ Managing collection dependencies
- ✅ Best practices for collections

## Additional Resources

- [Ansible Collections Overview](https://docs.ansible.com/ansible/latest/user_guide/collections_using.html)
- [Developing Collections](https://docs.ansible.com/ansible/latest/dev_guide/developing_collections.html)
- [Ansible Galaxy Collections](https://galaxy.ansible.com/)
- [Collection Index](https://docs.ansible.com/ansible/latest/collections/index.html)

## Next Steps

Proceed to [Lesson 11: Ansible Vault and Security](../11-vault-security/README.md) to learn about securing sensitive data in Ansible.
