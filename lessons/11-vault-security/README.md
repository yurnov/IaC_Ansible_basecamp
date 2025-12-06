# Lesson 11: Ansible Vault and Security

## Lesson Objectives
By the end of this lesson, you will be able to:
- Encrypt sensitive data using Ansible Vault
- Manage vault passwords securely
- Implement security best practices
- Use vault in playbooks and roles
- Understand Ansible security principles

## Prerequisites
- Completed Lesson 10: Ansible Collections
- Understanding of encryption concepts

## Duration
75 minutes

## Lesson Content

### What is Ansible Vault?

Ansible Vault is a feature that allows you to encrypt sensitive data such as:
- Passwords
- API keys
- Certificates
- Private keys
- Any sensitive variables

### Creating Encrypted Files

#### Create New Encrypted File

```bash
# Create and edit encrypted file
ansible-vault create secrets.yml

# You'll be prompted for vault password
# Opens editor (EDITOR environment variable)
```

#### Encrypt Existing File

```bash
# Encrypt an existing file
ansible-vault encrypt vars/passwords.yml

# Encrypt multiple files
ansible-vault encrypt group_vars/production.yml host_vars/db01.yml

# Encrypt with specific vault ID
ansible-vault encrypt --vault-id prod@prompt secrets.yml
```

### Viewing and Editing Encrypted Files

```bash
# View encrypted file
ansible-vault view secrets.yml

# Edit encrypted file
ansible-vault edit secrets.yml

# Decrypt file (creates unencrypted version)
ansible-vault decrypt secrets.yml

# Rekey (change password)
ansible-vault rekey secrets.yml
```

###

 Encrypted Variables

#### Encrypting Specific Strings

```bash
# Encrypt a string
ansible-vault encrypt_string 'secret_password' --name 'db_password'

# Output:
# db_password: !vault |
#           $ANSIBLE_VAULT;1.1;AES256
#           66386439653731323539366564643938613334363865636266343861666464...
```

#### Using Encrypted Strings in Files

```yaml
# filepath: examples/vars-encrypted.yml
---
# Plain variables
db_host: localhost
db_port: 5432
db_name: myapp

# Encrypted variables
db_password: !vault |
          $ANSIBLE_VAULT;1.1;AES256
          66386439653731323539366564643938613334363865636266343861666464326465373839306432
          3837306537366562323261373530386665353564303738610a363236636238303562343465346535
          38306465653066343866353933653038393466316266333539633662613531343439626533373064
          3766623437646430380a653663323034323537373039343330393865313834313065386262323834
          6234

api_key: !vault |
          $ANSIBLE_VAULT;1.1;AES256
          35623533343361626139353534653838373564626334346231316461313831396437363033336630
          6365356336373636660a336235663938656134356339386239323630636261626562313064393431
          30393439353437666437336465623165633732383965643361366534376332383639
```

### Using Vault in Playbooks

```yaml
# filepath: examples/vault-playbook.yml
---
- name: Deploy with encrypted variables
  hosts: webservers
  become: yes
  
  vars_files:
    - vars/common.yml
    - vars/secrets.yml  # Encrypted file
  
  tasks:
    - name: Create database user
      postgresql_user:
        name: "{{ db_user }}"
        password: "{{ db_password }}"  # From encrypted vars
        state: present
    
    - name: Configure application
      template:
        src: app.conf.j2
        dest: /etc/app/app.conf
      no_log: true  # Don't log sensitive info
```

Run playbook with vault:
```bash
# Prompt for vault password
ansible-playbook playbook.yml --ask-vault-pass

# Use password file
ansible-playbook playbook.yml --vault-password-file ~/.vault_pass

# Use password script
ansible-playbook playbook.yml --vault-password-file ~/.vault_pass.sh
```

### Multiple Vault IDs

```bash
# Create file with specific vault ID
ansible-vault create --vault-id prod@prompt secrets_prod.yml
ansible-vault create --vault-id dev@prompt secrets_dev.yml

# Encrypt string with vault ID
ansible-vault encrypt_string --vault-id prod@prompt 'prod_password' --name 'db_password'

# Use multiple vault IDs
ansible-playbook playbook.yml \
  --vault-id dev@~/.vault_pass_dev \
  --vault-id prod@prompt
```

```yaml
# filepath: examples/multi-vault.yml
---
- name: Using multiple vault IDs
  hosts: all
  
  vars_files:
    - secrets_dev.yml   # Encrypted with dev vault
    - secrets_prod.yml  # Encrypted with prod vault
  
  tasks:
    - name: Use development credentials
      debug:
        msg: "Dev DB: {{ dev_db_password }}"
      when: environment == "development"
    
    - name: Use production credentials
      debug:
        msg: "Prod DB: {{ prod_db_password }}"
      when: environment == "production"
```

### Vault Password Management

#### Password File

```bash
# filepath: examples/.vault_pass
my_secure_password_here
```

```bash
# Set permissions
chmod 600 .vault_pass

# Use in playbook
ansible-playbook playbook.yml --vault-password-file .vault_pass
```

#### Password Script

```bash
#!/bin/bash
# filepath: examples/.vault_pass.sh

# Get password from environment
echo $ANSIBLE_VAULT_PASSWORD

# Or from secure storage
# aws secretsmanager get-secret-value --secret-id ansible-vault | jq -r .SecretString

# Or from password manager
# pass show ansible/vault-password
```

```bash
# Make executable
chmod +x .vault_pass.sh

# Use in playbook
ansible-playbook playbook.yml --vault-password-file .vault_pass.sh
```

#### Environment Variable

```bash
# Set vault password in environment
export ANSIBLE_VAULT_PASSWORD=my_secure_password

# Or use ANSIBLE_VAULT_PASSWORD_FILE
export ANSIBLE_VAULT_PASSWORD_FILE=~/.vault_pass

# Run playbook (no need for --vault-password-file)
ansible-playbook playbook.yml
```

#### Configuration File

```ini
# filepath: examples/ansible.cfg
[defaults]
vault_password_file = ~/.vault_pass

# Or for multiple vaults
# vault_identity_list = dev@~/.vault_pass_dev, prod@~/.vault_pass_prod
```

### Security Best Practices

#### 1. No Logging for Sensitive Tasks

```yaml
# filepath: examples/no-log.yml
---
- name: Security best practices
  hosts: all
  
  tasks:
    - name: Set password (don't log)
      user:
        name: admin
        password: "{{ admin_password | password_hash('sha512') }}"
      no_log: true
    
    - name: Configure API key (don't log)
      lineinfile:
        path: /etc/app/config
        line: "api_key={{ api_key }}"
        state: present
      no_log: true
    
    - name: Debug with sanitized output
      debug:
        msg: "Password is {{ '*' * 10 }}"
```

#### 2. Restrict File Permissions

```yaml
# filepath: examples/secure-permissions.yml
---
- name: Secure file permissions
  hosts: all
  become: yes
  
  tasks:
    - name: Create config with secure permissions
      copy:
        content: |
          username={{ db_user }}
          password={{ db_password }}
        dest: /etc/app/db.conf
        owner: appuser
        group: appuser
        mode: '0600'  # Only owner can read/write
      no_log: true
    
    - name: Create SSH key with secure permissions
      copy:
        content: "{{ ssh_private_key }}"
        dest: /home/{{ ansible_user }}/.ssh/id_rsa
        owner: "{{ ansible_user }}"
        mode: '0400'  # Only owner can read
      no_log: true
```

#### 3. Use Become Sparingly

```yaml
# filepath: examples/minimal-privilege.yml
---
- name: Minimal privilege usage
  hosts: all
  
  tasks:
    # Don't need become for these
    - name: Create user file
      copy:
        content: "data"
        dest: "/home/{{ ansible_user }}/file.txt"
    
    # Only use become when necessary
    - name: Install package
      apt:
        name: nginx
        state: present
      become: yes
      become_user: root  # Be explicit
```

#### 4. Validate SSL/TLS

```yaml
# filepath: examples/validate-certs.yml
---
- name: Validate certificates
  hosts: all
  
  tasks:
    - name: Download file with cert validation
      get_url:
        url: https://example.com/file.tar.gz
        dest: /tmp/file.tar.gz
        validate_certs: yes  # Always validate in production
    
    - name: API call with cert validation
      uri:
        url: https://api.example.com/endpoint
        validate_certs: yes
        headers:
          Authorization: "Bearer {{ api_token }}"
      no_log: true
```

#### 5. Sanitize User Input

```yaml
# filepath: examples/input-validation.yml
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
    - name: Validate username format
      assert:
        that:
          - username is match('^[a-z][a-z0-9_-]{2,15}$')
        fail_msg: "Invalid username format"
    
    - name: Validate email format
      assert:
        that:
          - email is match('^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$')
        fail_msg: "Invalid email format"
    
    - name: Create user
      user:
        name: "{{ username }}"
        comment: "{{ email }}"
        state: present
      become: yes
```

### Secrets Management Integration

#### AWS Secrets Manager

```yaml
# filepath: examples/aws-secrets.yml
---
- name: Use AWS Secrets Manager
  hosts: localhost
  connection: local
  
  tasks:
    - name: Get secret from AWS
      set_fact:
        db_credentials: "{{ lookup('aws_secret', 'prod/db/credentials', region='us-east-1') | from_json }}"
      no_log: true
    
    - name: Use credentials
      debug:
        msg: "DB User: {{ db_credentials.username }}"
```

#### HashiCorp Vault

```yaml
# filepath: examples/hashicorp-vault.yml
---
- name: Use HashiCorp Vault
  hosts: all
  
  vars:
    vault_addr: https://vault.example.com:8200
    vault_token: "{{ lookup('env', 'VAULT_TOKEN') }}"
  
  tasks:
    - name: Get secret from Vault
      set_fact:
        db_password: "{{ lookup('hashi_vault', 'secret=secret/data/database:password token={{ vault_token }} url={{ vault_addr }}') }}"
      no_log: true
    
    - name: Configure database
      postgresql_db:
        name: myapp
        login_password: "{{ db_password }}"
      no_log: true
```

#### Azure Key Vault

```yaml
# filepath: examples/azure-keyvault.yml
---
- name: Use Azure Key Vault
  hosts: localhost
  connection: local
  
  tasks:
    - name: Get secret from Azure Key Vault
      azure_rm_keyvaultsecret_info:
        vault_uri: https://myvault.vault.azure.net
        name: db-password
      register: secret_info
      no_log: true
    
    - name: Use secret
      set_fact:
        db_password: "{{ secret_info.secrets[0].secret }}"
      no_log: true
```

## Hands-On Lab

### Lab 1: Basic Vault Usage

```bash
# Create encrypted file
ansible-vault create lab/secrets.yml
```

```yaml
# filepath: lab/secrets.yml (content before encryption)
---
db_password: SuperSecret123!
api_key: sk-1234567890abcdef
ssl_private_key: |
  -----BEGIN PRIVATE KEY-----
  MIIEvQIBADANBgkqhkiG9w0BAQEFAASCBKcwggSjAgEAAoIBAQC...
  -----END PRIVATE KEY-----
```

```yaml
# filepath: lab/deploy-secure.yml
---
- name: Secure deployment
  hosts: webservers
  become: yes
  
  vars_files:
    - secrets.yml
  
  tasks:
    - name: Create database
      postgresql_db:
        name: myapp
        login_password: "{{ db_password }}"
      no_log: true
    
    - name: Deploy SSL certificate
      copy:
        content: "{{ ssl_private_key }}"
        dest: /etc/ssl/private/server.key
        owner: root
        group: root
        mode: '0400'
      no_log: true
```

Run playbook:
```bash
ansible-playbook lab/deploy-secure.yml --ask-vault-pass
```

### Lab 2: Encrypt Specific Variables

```bash
# Encrypt individual strings
ansible-vault encrypt_string 'prod_db_pass_123' --name 'prod_db_password' >> lab/group_vars/production.yml
ansible-vault encrypt_string 'dev_db_pass_456' --name 'dev_db_password' >> lab/group_vars/development.yml
```

```yaml
# filepath: lab/group_vars/production.yml
---
environment: production
db_host: prod-db.example.com
db_port: 5432
db_name: myapp_prod

prod_db_password: !vault |
          $ANSIBLE_VAULT;1.1;AES256
          ...encrypted content...
```

```yaml
# filepath: lab/multi-env-deploy.yml
---
- name: Multi-environment deployment
  hosts: all
  become: yes
  
  tasks:
    - name: Display environment
      debug:
        msg: "Deploying to {{ environment }}"
    
    - name: Configure database connection
      template:
        src: db_config.j2
        dest: /etc/app/db.conf
        mode: '0600'
      no_log: true
```

### Lab 3: Multiple Vault IDs

```bash
# Create development secrets
ansible-vault create --vault-id dev@prompt lab/secrets_dev.yml

# Create production secrets  
ansible-vault create --vault-id prod@prompt lab/secrets_prod.yml
```

```yaml
# filepath: lab/secrets_dev.yml
---
db_password: dev_password_123
api_key: dev_api_key_456
```

```yaml
# filepath: lab/secrets_prod.yml
---
db_password: prod_password_789
api_key: prod_api_key_012
```

```yaml
# filepath: lab/multi-vault-deploy.yml
---
- name: Deployment with multiple vaults
  hosts: "{{ target_env }}"
  become: yes
  
  vars_files:
    - "secrets_{{ target_env }}.yml"
  
  tasks:
    - name: Configure application
      template:
        src: app.conf.j2
        dest: /etc/app/app.conf
      no_log: true
```

Run with multiple vaults:
```bash
ansible-playbook lab/multi-vault-deploy.yml \
  -e target_env=dev \
  --vault-id dev@prompt

ansible-playbook lab/multi-vault-deploy.yml \
  -e target_env=prod \
  --vault-id prod@prompt
```

### Lab 4: Password Script

```bash
#!/bin/bash
# filepath: lab/.vault_pass.sh

# Example 1: From environment variable
if [ -n "$ANSIBLE_VAULT_PASSWORD" ]; then
    echo "$ANSIBLE_VAULT_PASSWORD"
    exit 0
fi

# Example 2: From password manager (pass)
if command -v pass &> /dev/null; then
    pass show ansible/vault-password 2>/dev/null
    exit 0
fi

# Example 3: From AWS Secrets Manager
if command -v aws &> /dev/null; then
    aws secretsmanager get-secret-value \
        --secret-id ansible-vault-password \
        --query SecretString \
        --output text 2>/dev/null
    exit 0
fi

echo "Error: Could not retrieve vault password" >&2
exit 1
```

```bash
# Make executable
chmod +x lab/.vault_pass.sh

# Test it
./lab/.vault_pass.sh

# Use in playbook
ansible-playbook playbook.yml --vault-password-file lab/.vault_pass.sh
```

### Lab 5: Security Audit Playbook

```yaml
# filepath: lab/security-audit.yml
---
- name: Security audit
  hosts: all
  become: yes
  
  tasks:
    - name: Check for world-readable sensitive files
      find:
        paths:
          - /etc
          - /root
        patterns:
          - "*password*"
          - "*key*"
          - "*secret*"
        file_type: file
        recurse: yes
      register: sensitive_files
    
    - name: Check file permissions
      stat:
        path: "{{ item.path }}"
      loop: "{{ sensitive_files.files }}"
      register: file_stats
      loop_control:
        label: "{{ item.path }}"
    
    - name: Report world-readable files
      debug:
        msg: "WARNING: {{ item.item.path }} is world-readable!"
      when: item.stat.mode[-1] != '0'
      loop: "{{ file_stats.results }}"
      loop_control:
        label: "{{ item.item.path }}"
    
    - name: Check SSH configuration
      lineinfile:
        path: /etc/ssh/sshd_config
        regexp: "{{ item.regexp }}"
        line: "{{ item.line }}"
        state: present
      check_mode: yes
      register: ssh_check
      loop:
        - { regexp: '^PermitRootLogin', line: 'PermitRootLogin no' }
        - { regexp: '^PasswordAuthentication', line: 'PasswordAuthentication no' }
        - { regexp: '^PermitEmptyPasswords', line: 'PermitEmptyPasswords no' }
    
    - name: Report SSH misconfigurations
      debug:
        msg: "SSH configuration needs update"
      when: ssh_check.changed
```

## Summary

In this lesson, you learned:
- ✅ Encrypting data with Ansible Vault
- ✅ Managing vault passwords securely
- ✅ Using encrypted variables in playbooks
- ✅ Implementing security best practices
- ✅ Integrating with secrets management systems
- ✅ Auditing security configurations

## Additional Resources

- [Ansible Vault Documentation](https://docs.ansible.com/ansible/latest/user_guide/vault.html)
- [Ansible Security](https://docs.ansible.com/ansible/latest/user_guide/playbooks_best_practices.html#security)
- [Managing Secrets](https://docs.ansible.com/ansible/latest/user_guide/vault.html#managing-vault-passwords)

## Next Steps

Proceed to [Lesson 12: Testing Ansible Code](../12-testing/README.md) to learn how to test your Ansible playbooks and roles.
