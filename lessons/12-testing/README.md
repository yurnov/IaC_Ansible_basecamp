# Lesson 12: Testing Ansible Code

## Lesson Objectives
By the end of this lesson, you will be able to:
- Test playbooks and roles with different tools
- Use Molecule for role testing
- Implement syntax checking and linting
- Create integration tests
- Implement CI/CD testing pipelines

## Prerequisites
- Completed Lesson 11: Ansible Vault and Security
- Understanding of testing concepts

## Duration
90 minutes

## Lesson Content

### Why Test Ansible Code?

Testing ensures:
- **Correctness**: Code works as intended
- **Reliability**: Consistent behavior
- **Maintainability**: Easier refactoring
- **Documentation**: Tests serve as examples
- **Confidence**: Safe deployments

### Types of Testing

1. **Syntax Checking**: Validate YAML and Ansible syntax
2. **Linting**: Code style and best practices
3. **Unit Testing**: Test individual components
4. **Integration Testing**: Test complete workflows
5. **Idempotency Testing**: Verify no changes on re-run

### Syntax Checking

#### ansible-playbook --syntax-check

```bash
# Check playbook syntax
ansible-playbook playbook.yml --syntax-check

# Check with inventory
ansible-playbook -i inventory.yml playbook.yml --syntax-check

# Check multiple playbooks
ansible-playbook *.yml --syntax-check
```

```yaml
# filepath: examples/check-syntax.sh
#!/bin/bash
# Automated syntax checking

echo "Checking playbook syntax..."
for playbook in playbooks/*.yml; do
    echo "Checking $playbook"
    ansible-playbook "$playbook" --syntax-check || exit 1
done

echo "All playbooks passed syntax check!"
```

#### YAML Validation

```bash
# Install yamllint
pip install yamllint

# Check YAML files
yamllint playbook.yml

# Check all YAML files
yamllint .

# Use custom config
yamllint -c .yamllint playbook.yml
```

```yaml
# filepath: examples/.yamllint
---
extends: default

rules:
  line-length:
    max: 120
    level: warning
  
  indentation:
    spaces: 2
    indent-sequences: yes
  
  comments:
    min-spaces-from-content: 1
  
  braces:
    max-spaces-inside: 1
  
  brackets:
    max-spaces-inside: 1
  
  truthy:
    allowed-values: ['true', 'false', 'yes', 'no']
```

### Linting with ansible-lint

```bash
# Install ansible-lint
pip install ansible-lint

# Lint playbook
ansible-lint playbook.yml

# Lint all playbooks in directory
ansible-lint playbooks/

# Lint with specific rules
ansible-lint -t yaml playbook.yml

# Skip specific rules
ansible-lint -x 204,206 playbook.yml

# Generate config file
ansible-lint --generate-ignore
```

```yaml
# filepath: examples/.ansible-lint
---
# Exclude paths
exclude_paths:
  - .cache/
  - .github/
  - molecule/
  - .venv/

# Skip specific rules
skip_list:
  - '204'  # Lines should be no longer than 160 chars
  - 'role-name'  # Role name does not match ^[a-z][a-z0-9_]+$ pattern

# Enable optional rules
enable_list:
  - no-same-owner
  - yaml

# Warn instead of error for certain rules
warn_list:
  - experimental
  - unnamed-task

# Custom rule configuration
rules:
  line-length:
    max: 120
```

Common ansible-lint rules:
```yaml
# filepath: examples/lint-rules-example.yml
---
- name: Examples of common lint issues
  hosts: all
  
  tasks:
    # Bad: Unnamed task (rule 502)
    - debug:
        msg: "Hello"
    
    # Good: Named task
    - name: Display message
      debug:
        msg: "Hello"
    
    # Bad: Using command instead of module (rule 303)
    - name: Install package
      command: apt-get install nginx
    
    # Good: Use proper module
    - name: Install package
      apt:
        name: nginx
        state: present
    
    # Bad: No changed_when (rule 301)
    - name: Check status
      command: systemctl status nginx
    
    # Good: With changed_when
    - name: Check status
      command: systemctl status nginx
      changed_when: false
```

### Check Mode (Dry Run)

```bash
# Run playbook in check mode
ansible-playbook playbook.yml --check

# Check mode with diff
ansible-playbook playbook.yml --check --diff

# Check mode for specific tags
ansible-playbook playbook.yml --check --tags configuration
```

```yaml
# filepath: examples/check-mode-playbook.yml
---
- name: Testing with check mode
  hosts: all
  
  tasks:
    - name: This task always runs in check mode
      debug:
        msg: "Always in check mode"
      check_mode: yes
    
    - name: This task never runs in check mode
      debug:
        msg: "Never in check mode"
      check_mode: no
    
    - name: Conditional behavior in check mode
      command: echo "Running command"
      when: not ansible_check_mode
      changed_when: false
```

### Molecule Testing Framework

Molecule is a testing framework designed for Ansible roles.

#### Install Molecule

```bash
# Install molecule with docker driver
pip install molecule molecule-docker

# Or with vagrant driver
pip install molecule molecule-vagrant

# Or with podman driver
pip install molecule molecule-podman
```

#### Initialize Molecule

```bash
# Initialize molecule for a role
cd roles/myrole
molecule init scenario --driver-name docker

# Create new role with molecule
molecule init role myrole --driver-name docker
```

Molecule creates this structure:
```
molecule/
└── default/
    ├── converge.yml        # Playbook to test
    ├── molecule.yml        # Molecule configuration
    ├── verify.yml          # Verification tests
    └── prepare.yml         # Pre-test setup (optional)
```

#### Molecule Configuration

```yaml
# filepath: examples/roles/webserver/molecule/default/molecule.yml
---
dependency:
  name: galaxy
  options:
    requirements-file: requirements.yml

driver:
  name: docker

platforms:
  - name: ubuntu2004
    image: geerlingguy/docker-ubuntu2004-ansible:latest
    pre_build_image: true
    privileged: true
    volumes:
      - /sys/fs/cgroup:/sys/fs/cgroup:ro
    command: /lib/systemd/systemd
  
  - name: debian11
    image: geerlingguy/docker-debian11-ansible:latest
    pre_build_image: true
    privileged: true
    volumes:
      - /sys/fs/cgroup:/sys/fs/cgroup:ro
    command: /lib/systemd/systemd

provisioner:
  name: ansible
  config_options:
    defaults:
      callbacks_enabled: profile_tasks
      stdout_callback: yaml
  playbooks:
    converge: converge.yml
    verify: verify.yml
  inventory:
    host_vars:
      ubuntu2004:
        ansible_user: ansible
      debian11:
        ansible_user: ansible

verifier:
  name: ansible

scenario:
  test_sequence:
    - dependency
    - cleanup
    - destroy
    - syntax
    - create
    - prepare
    - converge
    - idempotence
    - side_effect
    - verify
    - cleanup
    - destroy
```

#### Molecule Converge Playbook

```yaml
# filepath: examples/roles/webserver/molecule/default/converge.yml
---
- name: Converge
  hosts: all
  become: yes
  
  vars:
    http_port: 80
    server_name: test.example.com
  
  roles:
    - role: webserver
```

#### Molecule Verify Playbook

```yaml
# filepath: examples/roles/webserver/molecule/default/verify.yml
---
- name: Verify
  hosts: all
  gather_facts: yes
  
  tasks:
    - name: Check nginx is installed
      package_facts:
        manager: auto
    
    - name: Verify nginx package
      assert:
        that:
          - "'nginx' in ansible_facts.packages"
        fail_msg: "nginx is not installed"
    
    - name: Check nginx service status
      service_facts:
    
    - name: Verify nginx is running
      assert:
        that:
          - ansible_facts.services['nginx.service'].state == 'running'
          - ansible_facts.services['nginx.service'].status == 'enabled'
        fail_msg: "nginx service is not running or enabled"
    
    - name: Check nginx is listening on port 80
      wait_for:
        port: 80
        timeout: 10
      register: port_check
    
    - name: Verify port 80 is open
      assert:
        that:
          - port_check is succeeded
        fail_msg: "nginx is not listening on port 80"
    
    - name: Test HTTP response
      uri:
        url: http://localhost
        return_content: yes
      register: http_response
    
    - name: Verify HTTP response
      assert:
        that:
          - http_response.status == 200
        fail_msg: "HTTP request failed"
```

#### Running Molecule Tests

```bash
# Run full test sequence
molecule test

# Create instance
molecule create

# Run converge (apply role)
molecule converge

# Run verify tests
molecule verify

# Test idempotence
molecule idempotence

# Login to instance
molecule login

# Destroy instances
molecule destroy

# List instances
molecule list

# Run specific scenario
molecule test -s alternate
```

### Integration Testing

```yaml
# filepath: examples/tests/integration/test-playbook.yml
---
- name: Integration test for web application
  hosts: test_servers
  become: yes
  
  vars:
    app_name: myapp
    app_port: 8080
  
  tasks:
    - name: Deploy application
      include_role:
        name: myapp
    
    - name: Wait for application to start
      wait_for:
        port: "{{ app_port }}"
        delay: 5
        timeout: 60
    
    - name: Test health endpoint
      uri:
        url: "http://localhost:{{ app_port }}/health"
        return_content: yes
      register: health_check
      retries: 5
      delay: 2
      until: health_check.status == 200
    
    - name: Verify health check response
      assert:
        that:
          - health_check.status == 200
          - "'healthy' in health_check.content"
        fail_msg: "Health check failed"
    
    - name: Test API endpoint
      uri:
        url: "http://localhost:{{ app_port }}/api/version"
        return_content: yes
      register: api_response
    
    - name: Verify API response
      assert:
        that:
          - api_response.status == 200
          - api_response.json.version is defined
        fail_msg: "API test failed"
    
    - name: Test database connection
      command: "{{ app_dir }}/bin/check-db.sh"
      register: db_check
      changed_when: false
    
    - name: Verify database connectivity
      assert:
        that:
          - db_check.rc == 0
        fail_msg: "Database connection failed"
```

### Testinfra Integration

```python
# filepath: examples/roles/webserver/molecule/default/tests/test_default.py
"""
Testinfra tests for webserver role
"""
import os
import pytest
import testinfra.utils.ansible_runner

testinfra_hosts = testinfra.utils.ansible_runner.AnsibleRunner(
    os.environ['MOLECULE_INVENTORY_FILE']
).get_hosts('all')


def test_nginx_package_installed(host):
    """Verify nginx package is installed"""
    nginx = host.package('nginx')
    assert nginx.is_installed


def test_nginx_service_running(host):
    """Verify nginx service is running"""
    nginx = host.service('nginx')
    assert nginx.is_running
    assert nginx.is_enabled


def test_nginx_listening_on_port_80(host):
    """Verify nginx is listening on port 80"""
    assert host.socket('tcp://0.0.0.0:80').is_listening


def test_nginx_config_file_exists(host):
    """Verify nginx config file exists"""
    config = host.file('/etc/nginx/nginx.conf')
    assert config.exists
    assert config.is_file
    assert config.user == 'root'
    assert config.group == 'root'


def test_nginx_responds_to_http(host):
    """Verify nginx responds to HTTP requests"""
    cmd = host.run('curl -s -o /dev/null -w "%{http_code}" http://localhost')
    assert cmd.stdout == '200'


@pytest.mark.parametrize('path', [
    '/var/log/nginx',
    '/var/www/html',
    '/etc/nginx/sites-available',
    '/etc/nginx/sites-enabled'
])
def test_nginx_directories_exist(host, path):
    """Verify required directories exist"""
    directory = host.file(path)
    assert directory.exists
    assert directory.is_directory
```

### CI/CD Integration

#### GitHub Actions

```yaml
# filepath: examples/.github/workflows/ansible-test.yml
---
name: Ansible Tests

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

jobs:
  lint:
    name: Lint Ansible Code
    runs-on: ubuntu-latest
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v3
      
      - name: Set up Python
        uses: actions/setup-python@v4
        with:
          python-version: '3.10'
      
      - name: Install dependencies
        run: |
          pip install ansible ansible-lint yamllint
      
      - name: Run yamllint
        run: yamllint .
      
      - name: Run ansible-lint
        run: ansible-lint
      
      - name: Syntax check
        run: |
          ansible-playbook playbooks/*.yml --syntax-check

  molecule:
    name: Molecule Tests
    runs-on: ubuntu-latest
    strategy:
      matrix:
        distro:
          - ubuntu2004
          - debian11
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v3
      
      - name: Set up Python
        uses: actions/setup-python@v4
        with:
          python-version: '3.10'
      
      - name: Install dependencies
        run: |
          pip install molecule molecule-docker docker
      
      - name: Run Molecule tests
        run: |
          cd roles/webserver
          molecule test
        env:
          PY_COLORS: '1'
          ANSIBLE_FORCE_COLOR: '1'
          MOLECULE_DISTRO: ${{ matrix.distro }}

  integration:
    name: Integration Tests
    runs-on: ubuntu-latest
    needs: [lint, molecule]
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v3
      
      - name: Set up Python
        uses: actions/setup-python@v4
        with:
          python-version: '3.10'
      
      - name: Install Ansible
        run: pip install ansible
      
      - name: Run integration tests
        run: |
          ansible-playbook tests/integration/test-playbook.yml \
            -i tests/integration/inventory.yml \
            --extra-vars "test_mode=true"
```

#### GitLab CI

```yaml
# filepath: examples/.gitlab-ci.yml
---
stages:
  - lint
  - test
  - deploy

variables:
  PIP_CACHE_DIR: "$CI_PROJECT_DIR/.cache/pip"

cache:
  paths:
    - .cache/pip

lint:yaml:
  stage: lint
  image: python:3.10
  before_script:
    - pip install yamllint
  script:
    - yamllint .
  only:
    - merge_requests
    - main

lint:ansible:
  stage: lint
  image: python:3.10
  before_script:
    - pip install ansible ansible-lint
  script:
    - ansible-lint
    - ansible-playbook playbooks/*.yml --syntax-check
  only:
    - merge_requests
    - main

test:molecule:
  stage: test
  image: python:3.10
  services:
    - docker:dind
  variables:
    DOCKER_HOST: tcp://docker:2375
  before_script:
    - pip install molecule molecule-docker docker
  script:
    - cd roles/webserver
    - molecule test
  only:
    - merge_requests
    - main

test:integration:
  stage: test
  image: python:3.10
  before_script:
    - pip install ansible
  script:
    - ansible-playbook tests/integration/test-playbook.yml -i tests/integration/inventory.yml
  only:
    - merge_requests
    - main
```

## Hands-On Lab

### Lab 1: Basic Testing Setup

```bash
# Create test structure
mkdir -p tests/{unit,integration}
```

```yaml
# filepath: lab/.yamllint
---
extends: default
rules:
  line-length:
    max: 120
  indentation:
    spaces: 2
```

```yaml
# filepath: lab/.ansible-lint
---
skip_list:
  - '204'
  - 'role-name'

warn_list:
  - experimental
```

```bash
# Run tests
yamllint .
ansible-lint
ansible-playbook site.yml --syntax-check
ansible-playbook site.yml --check
```

### Lab 2: Create Molecule Tests for a Role

```bash
# Create role with molecule
cd lab
molecule init role webserver --driver-name docker
cd webserver
```

```yaml
# filepath: lab/webserver/molecule/default/molecule.yml
---
dependency:
  name: galaxy

driver:
  name: docker

platforms:
  - name: instance
    image: geerlingguy/docker-ubuntu2004-ansible:latest
    pre_build_image: true
    privileged: true
    command: /lib/systemd/systemd
    volumes:
      - /sys/fs/cgroup:/sys/fs/cgroup:ro

provisioner:
  name: ansible

verifier:
  name: ansible
```

```yaml
# filepath: lab/webserver/molecule/default/converge.yml
---
- name: Converge
  hosts: all
  become: yes
  
  roles:
    - role: webserver
      vars:
        http_port: 80
        server_name: test.local
```

```yaml
# filepath: lab/webserver/molecule/default/verify.yml
---
- name: Verify
  hosts: all
  
  tasks:
    - name: Check nginx is installed
      command: nginx -v
      changed_when: false
    
    - name: Check nginx service
      service_facts:
    
    - name: Verify nginx is running
      assert:
        that:
          - ansible_facts.services['nginx.service'].state == 'running'
    
    - name: Test HTTP response
      uri:
        url: http://localhost
        status_code: 200
```

```bash
# Run molecule tests
molecule test
```

### Lab 3: Integration Test Suite

```yaml
# filepath: lab/tests/integration/inventory.yml
---
all:
  hosts:
    testserver:
      ansible_host: localhost
      ansible_connection: local
```

```yaml
# filepath: lab/tests/integration/test-full-stack.yml
---
- name: Full stack integration test
  hosts: testserver
  become: yes
  
  vars:
    test_user: testuser
    test_db: testdb
  
  pre_tasks:
    - name: Install test dependencies
      apt:
        name:
          - curl
          - postgresql
        state: present
  
  roles:
    - database
    - application
    - webserver
  
  post_tasks:
    - name: Wait for all services
      wait_for:
        port: "{{ item }}"
        timeout: 30
      loop:
        - 80
        - 5432
        - 8080
    
    - name: Test web interface
      uri:
        url: http://localhost
        return_content: yes
      register: web_test
    
    - name: Verify web response
      assert:
        that:
          - web_test.status == 200
          - "'Welcome' in web_test.content"
    
    - name: Test application API
      uri:
        url: http://localhost:8080/api/health
        return_content: yes
      register: api_test
    
    - name: Verify API response
      assert:
        that:
          - api_test.status == 200
          - api_test.json.status == 'healthy'
    
    - name: Test database connection
      postgresql_ping:
        db: "{{ test_db }}"
      register: db_test
    
    - name: Verify database
      assert:
        that:
          - db_test is succeeded
```

### Lab 4: CI/CD Pipeline

```yaml
# filepath: lab/.github/workflows/test.yml
---
name: Test Ansible Code

on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest
    
    steps:
      - uses: actions/checkout@v3
      
      - name: Set up Python
        uses: actions/setup-python@v4
        with:
          python-version: '3.10'
      
      - name: Install dependencies
        run: |
          pip install ansible ansible-lint yamllint molecule molecule-docker
      
      - name: Lint
        run: |
          yamllint .
          ansible-lint
      
      - name: Syntax check
        run: |
          find . -name '*.yml' -type f | xargs ansible-playbook --syntax-check
      
      - name: Molecule test
        run: |
          for role in roles/*; do
            if [ -d "$role/molecule" ]; then
              cd "$role"
              molecule test
              cd -
            fi
          done
```

## Best Practices

### 1. Test Pyramid
- Many unit tests (fast, isolated)
- Some integration tests (slower, realistic)
- Few end-to-end tests (slowest, complete)

### 2. Continuous Testing
- Run tests on every commit
- Automate in CI/CD pipeline
- Fast feedback loops

### 3. Test Coverage
- Test happy paths
- Test error conditions
- Test edge cases
- Test idempotence

### 4. Maintainable Tests
- Clear test names
- Independent tests
- Repeatable results
- Clean test data

### 5. Version Control
- Store tests with code
- Review test changes
- Document test requirements
- Track test metrics

## Summary

In this lesson, you learned:
- ✅ Syntax checking and linting
- ✅ Using Molecule for role testing
- ✅ Writing integration tests
- ✅ Implementing CI/CD testing
- ✅ Best practices for testing

## Additional Resources

- [Ansible Testing Strategies](https://docs.ansible.com/ansible/latest/reference_appendices/test_strategies.html)
- [Molecule Documentation](https://molecule.readthedocs.io/)
- [ansible-lint](https://ansible-lint.readthedocs.io/)
- [Testinfra](https://testinfra.readthedocs.io/)

## Next Steps

Proceed to [Lesson 13: CI/CD with Ansible](../13-cicd/README.md) to learn about integrating Ansible into continuous delivery pipelines.
