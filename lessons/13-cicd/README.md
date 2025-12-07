# Lesson 13: CI/CD with Ansible

## Lesson Objectives
By the end of this lesson, you will be able to:
- Integrate Ansible into CI/CD pipelines
- Implement automated deployment workflows
- Use Ansible with popular CI/CD tools
- Implement blue-green and canary deployments
- Build deployment pipelines with rollback capabilities

## Prerequisites
- Completed Lesson 12: Testing Ansible Code
- Understanding of CI/CD concepts
- Familiarity with Git

## Duration
90 minutes

## Lesson Content

### CI/CD with Ansible Overview

Ansible fits into CI/CD pipelines as:
- **Infrastructure provisioning**
- **Configuration management**
- **Application deployment**
- **Testing and validation**
- **Rollback automation**

### Pipeline Architecture

```
┌─────────────┐     ┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│   Source    │────▶│    Build    │────▶│    Test     │────▶│   Deploy    │
│   Control   │     │             │     │             │     │             │
└─────────────┘     └─────────────┘     └─────────────┘     └─────────────┘
     Git                 CI Tool            Ansible             Ansible
                                           Testing            Deployment
```

### GitHub Actions with Ansible

#### Basic Workflow

```yaml
# filepath: examples/.github/workflows/deploy.yml
---
name: Deploy with Ansible

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]
  workflow_dispatch:
    inputs:
      environment:
        description: 'Environment to deploy'
        required: true
        default: 'staging'
        type: choice
        options:
          - staging
          - production

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

  test:
    name: Test Playbooks
    runs-on: ubuntu-latest
    needs: lint
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v3
      
      - name: Set up Python
        uses: actions/setup-python@v4
        with:
          python-version: '3.10'
      
      - name: Install Ansible
        run: pip install ansible
      
      - name: Run check mode
        run: |
          ansible-playbook site.yml -i inventory/staging.yml --check

  deploy-staging:
    name: Deploy to Staging
    runs-on: ubuntu-latest
    needs: test
    if: github.ref == 'refs/heads/main'
    environment:
      name: staging
      url: https://staging.example.com
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v3
      
      - name: Set up Python
        uses: actions/setup-python@v4
        with:
          python-version: '3.10'
      
      - name: Install Ansible
        run: pip install ansible
      
      - name: Configure SSH
        run: |
          mkdir -p ~/.ssh
          echo "${{ secrets.SSH_PRIVATE_KEY }}" > ~/.ssh/id_rsa
          chmod 600 ~/.ssh/id_rsa
          ssh-keyscan -H ${{ secrets.STAGING_HOST }} >> ~/.ssh/known_hosts
      
      - name: Deploy to staging
        env:
          ANSIBLE_HOST_KEY_CHECKING: 'false'
        run: |
          ansible-playbook site.yml \
            -i inventory/staging.yml \
            -e "app_version=${{ github.sha }}" \
            -e "environment=staging"
      
      - name: Verify deployment
        run: |
          ansible-playbook playbooks/verify.yml -i inventory/staging.yml

  deploy-production:
    name: Deploy to Production
    runs-on: ubuntu-latest
    needs: deploy-staging
    if: github.ref == 'refs/heads/main'
    environment:
      name: production
      url: https://production.example.com
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v3
      
      - name: Set up Python
        uses: actions/setup-python@v4
        with:
          python-version: '3.10'
      
      - name: Install Ansible
        run: pip install ansible
      
      - name: Configure SSH
        run: |
          mkdir -p ~/.ssh
          echo "${{ secrets.SSH_PRIVATE_KEY }}" > ~/.ssh/id_rsa
          chmod 600 ~/.ssh/id_rsa
          ssh-keyscan -H ${{ secrets.PRODUCTION_HOST }} >> ~/.ssh/known_hosts
      
      - name: Deploy to production
        env:
          ANSIBLE_HOST_KEY_CHECKING: 'false'
          ANSIBLE_VAULT_PASSWORD: ${{ secrets.VAULT_PASSWORD }}
        run: |
          echo "$ANSIBLE_VAULT_PASSWORD" > .vault_pass
          ansible-playbook site.yml \
            -i inventory/production.yml \
            -e "app_version=${{ github.sha }}" \
            -e "environment=production" \
            --vault-password-file .vault_pass
          rm .vault_pass
      
      - name: Smoke test
        run: |
          ansible-playbook playbooks/smoke-test.yml \
            -i inventory/production.yml
```

#### Reusable Workflow

```yaml
# filepath: examples/.github/workflows/ansible-deploy.yml
---
name: Reusable Ansible Deploy

on:
  workflow_call:
    inputs:
      environment:
        required: true
        type: string
      inventory:
        required: true
        type: string
    secrets:
      ssh_key:
        required: true
      vault_password:
        required: false

jobs:
  deploy:
    name: Deploy to ${{ inputs.environment }}
    runs-on: ubuntu-latest
    environment: ${{ inputs.environment }}
    
    steps:
      - uses: actions/checkout@v3
      
      - name: Set up Python
        uses: actions/setup-python@v4
        with:
          python-version: '3.10'
      
      - name: Install Ansible
        run: pip install ansible
      
      - name: Configure SSH
        run: |
          mkdir -p ~/.ssh
          echo "${{ secrets.ssh_key }}" > ~/.ssh/id_rsa
          chmod 600 ~/.ssh/id_rsa
      
      - name: Deploy
        env:
          VAULT_PASS: ${{ secrets.vault_password }}
        run: |
          if [ -n "$VAULT_PASS" ]; then
            echo "$VAULT_PASS" > .vault_pass
            VAULT_ARGS="--vault-password-file .vault_pass"
          fi
          
          ansible-playbook site.yml \
            -i ${{ inputs.inventory }} \
            -e "app_version=${{ github.sha }}" \
            -e "environment=${{ inputs.environment }}" \
            $VAULT_ARGS
```

### GitLab CI with Ansible

```yaml
# filepath: examples/.gitlab-ci.yml
---
stages:
  - validate
  - test
  - deploy-staging
  - deploy-production

variables:
  ANSIBLE_FORCE_COLOR: "true"
  ANSIBLE_HOST_KEY_CHECKING: "false"

.ansible-base:
  image: python:3.10
  before_script:
    - pip install ansible ansible-lint
    - mkdir -p ~/.ssh
    - echo "$SSH_PRIVATE_KEY" > ~/.ssh/id_rsa
    - chmod 600 ~/.ssh/id_rsa

validate:
  extends: .ansible-base
  stage: validate
  script:
    - yamllint .
    - ansible-lint
    - ansible-playbook site.yml --syntax-check
  only:
    - merge_requests
    - main

test:
  extends: .ansible-base
  stage: test
  script:
    - ansible-playbook site.yml -i inventory/staging.yml --check
  only:
    - merge_requests
    - main

deploy-staging:
  extends: .ansible-base
  stage: deploy-staging
  environment:
    name: staging
    url: https://staging.example.com
  script:
    - |
      ansible-playbook site.yml \
        -i inventory/staging.yml \
        -e "app_version=$CI_COMMIT_SHA" \
        -e "environment=staging"
  only:
    - main

deploy-production:
  extends: .ansible-base
  stage: deploy-production
  environment:
    name: production
    url: https://production.example.com
  script:
    - echo "$VAULT_PASSWORD" > .vault_pass
    - |
      ansible-playbook site.yml \
        -i inventory/production.yml \
        -e "app_version=$CI_COMMIT_SHA" \
        -e "environment=production" \
        --vault-password-file .vault_pass
    - rm .vault_pass
  only:
    - main
  when: manual
```

### Jenkins with Ansible

```groovy
// filepath: examples/Jenkinsfile
pipeline {
    agent any
    
    environment {
        ANSIBLE_FORCE_COLOR = 'true'
        ANSIBLE_HOST_KEY_CHECKING = 'false'
    }
    
    parameters {
        choice(
            name: 'ENVIRONMENT',
            choices: ['staging', 'production'],
            description: 'Environment to deploy'
        )
        string(
            name: 'APP_VERSION',
            defaultValue: 'latest',
            description: 'Application version to deploy'
        )
    }
    
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        
        stage('Validate') {
            steps {
                sh 'pip install ansible ansible-lint yamllint'
                sh 'yamllint .'
                sh 'ansible-lint'
                sh 'ansible-playbook site.yml --syntax-check'
            }
        }
        
        stage('Test') {
            steps {
                sh """
                    ansible-playbook site.yml \
                        -i inventory/staging.yml \
                        --check
                """
            }
        }
        
        stage('Deploy') {
            steps {
                withCredentials([
                    sshUserPrivateKey(
                        credentialsId: 'ansible-ssh-key',
                        keyFileVariable: 'SSH_KEY'
                    ),
                    string(
                        credentialsId: 'ansible-vault-password',
                        variable: 'VAULT_PASSWORD'
                    )
                ]) {
                    sh '''
                        mkdir -p ~/.ssh
                        cp $SSH_KEY ~/.ssh/id_rsa
                        chmod 600 ~/.ssh/id_rsa
                        
                        echo "$VAULT_PASSWORD" > .vault_pass
                        
                        ansible-playbook site.yml \
                            -i inventory/${ENVIRONMENT}.yml \
                            -e "app_version=${APP_VERSION}" \
                            -e "environment=${ENVIRONMENT}" \
                            --vault-password-file .vault_pass
                        
                        rm .vault_pass
                    '''
                }
            }
        }
        
        stage('Verify') {
            steps {
                sh """
                    ansible-playbook playbooks/verify.yml \
                        -i inventory/${params.ENVIRONMENT}.yml
                """
            }
        }
    }
    
    post {
        success {
            echo "Deployment to ${params.ENVIRONMENT} successful!"
            slackSend(
                color: 'good',
                message: "Deployed ${params.APP_VERSION} to ${params.ENVIRONMENT}"
            )
        }
        failure {
            echo "Deployment failed!"
            slackSend(
                color: 'danger',
                message: "Deployment to ${params.ENVIRONMENT} failed!"
            )
        }
    }
}
```

### Deployment Strategies

#### Blue-Green Deployment

```yaml
# filepath: examples/blue-green-deploy.yml
---
- name: Blue-Green Deployment
  hosts: localhost
  connection: local
  gather_facts: no
  
  vars:
    app_version: "{{ lookup('env', 'APP_VERSION') }}"
    environments:
      blue:
        servers: "{{ groups['blue'] }}"
        loadbalancer_pool: blue_pool
      green:
        servers: "{{ groups['green'] }}"
        loadbalancer_pool: green_pool
  
  tasks:
    - name: Determine current active environment
      set_fact:
        current_env: "{{ 'blue' if active_environment == 'green' else 'green' }}"
        target_env: "{{ 'green' if active_environment == 'blue' else 'blue' }}"
    
    - name: Deploy to inactive environment
      include_tasks: deploy-app.yml
      vars:
        target_hosts: "{{ environments[target_env].servers }}"
        environment: "{{ target_env }}"
    
    - name: Run smoke tests on new deployment
      uri:
        url: "http://{{ item }}:8080/health"
        status_code: 200
      loop: "{{ environments[target_env].servers }}"
      register: health_checks
      retries: 5
      delay: 10
    
    - name: Verify all health checks passed
      assert:
        that: health_checks is succeeded
        fail_msg: "Health checks failed on new deployment"
    
    - name: Switch load balancer to new environment
      community.general.haproxy:
        backend: app_backend
        host: "{{ environments[target_env].loadbalancer_pool }}"
        state: enabled
    
    - name: Disable old environment in load balancer
      community.general.haproxy:
        backend: app_backend
        host: "{{ environments[current_env].loadbalancer_pool }}"
        state: disabled
    
    - name: Update active environment marker
      set_fact:
        active_environment: "{{ target_env }}"
        cacheable: yes
```

#### Canary Deployment

```yaml
# filepath: examples/canary-deploy.yml
---
- name: Canary Deployment
  hosts: all
  become: yes
  serial:
    - 1        # Deploy to 1 server first (canary)
    - 25%      # Then 25% of remaining
    - 100%     # Then all remaining
  
  vars:
    app_version: "{{ lookup('env', 'APP_VERSION') }}"
    canary_duration: 300  # 5 minutes
  
  tasks:
    - name: Deploy application
      include_role:
        name: deploy_app
      vars:
        version: "{{ app_version }}"
    
    - name: Wait for canary stabilization
      pause:
        seconds: "{{ canary_duration }}"
      when: ansible_play_batch[0] == inventory_hostname
    
    - name: Check error rate
      uri:
        url: "http://{{ inventory_hostname }}:8080/metrics"
        return_content: yes
      register: metrics
      when: ansible_play_batch[0] == inventory_hostname
    
    - name: Verify canary metrics
      assert:
        that:
          - metrics.json.error_rate | float < 0.01
          - metrics.json.response_time | float < 200
        fail_msg: "Canary metrics exceeded thresholds"
      when: ansible_play_batch[0] == inventory_hostname
    
    - name: Continue deployment
      debug:
        msg: "Canary successful, continuing deployment"
```

#### Rolling Deployment with Rollback

```yaml
# filepath: examples/rolling-deploy.yml
---
- name: Rolling Deployment with Rollback
  hosts: webservers
  become: yes
  serial: 2
  max_fail_percentage: 0
  
  vars:
    app_version: "{{ lookup('env', 'APP_VERSION') }}"
    rollback_version: "{{ lookup('env', 'ROLLBACK_VERSION') }}"
  
  tasks:
    - name: Backup current version
      block:
        - name: Get current version
          command: cat /opt/app/VERSION
          register: current_version
          changed_when: false
          failed_when: false
        
        - name: Save rollback version
          set_fact:
            saved_rollback_version: "{{ current_version.stdout }}"
            cacheable: yes
    
    - name: Deploy new version
      block:
        - name: Remove from load balancer
          community.general.haproxy:
            backend: app_backend
            host: "{{ inventory_hostname }}"
            state: disabled
          delegate_to: loadbalancer
        
        - name: Deploy application
          include_role:
            name: deploy_app
          vars:
            version: "{{ app_version }}"
        
        - name: Health check
          uri:
            url: "http://localhost:8080/health"
            status_code: 200
          retries: 10
          delay: 5
        
        - name: Add back to load balancer
          community.general.haproxy:
            backend: app_backend
            host: "{{ inventory_hostname }}"
            state: enabled
          delegate_to: loadbalancer
      
      rescue:
        - name: Rollback on failure
          include_role:
            name: deploy_app
          vars:
            version: "{{ saved_rollback_version }}"
        
        - name: Restart application
          service:
            name: myapp
            state: restarted
        
        - name: Fail deployment
          fail:
            msg: "Deployment failed, rolled back to {{ saved_rollback_version }}"
```

### Infrastructure as Code Pipeline

```yaml
# filepath: examples/infrastructure-pipeline.yml
---
- name: Infrastructure Provisioning Pipeline
  hosts: localhost
  connection: local
  
  tasks:
    - name: Provision infrastructure
      block:
        - name: Create VPC
          amazon.aws.ec2_vpc_net:
            name: "{{ project_name }}-vpc"
            cidr_block: 10.0.0.0/16
            region: "{{ aws_region }}"
          register: vpc
        
        - name: Create subnets
          amazon.aws.ec2_vpc_subnet:
            vpc_id: "{{ vpc.vpc.id }}"
            cidr: "{{ item.cidr }}"
            az: "{{ item.az }}"
            tags:
              Name: "{{ item.name }}"
          loop:
            - { cidr: "10.0.1.0/24", az: "us-east-1a", name: "public-1a" }
            - { cidr: "10.0.2.0/24", az: "us-east-1b", name: "public-1b" }
          register: subnets
        
        - name: Launch EC2 instances
          amazon.aws.ec2_instance:
            name: "{{ project_name }}-{{ item }}"
            instance_type: t3.medium
            image_id: ami-0c55b159cbfafe1f0
            vpc_subnet_id: "{{ subnets.results[0].subnet.id }}"
            security_group: "{{ security_group.group_id }}"
            tags:
              Environment: "{{ environment }}"
              Application: "{{ app_name }}"
          loop: "{{ range(1, instance_count + 1) | list }}"
          register: instances
        
        - name: Wait for instances
          wait_for:
            host: "{{ item.public_ip_address }}"
            port: 22
            timeout: 300
          loop: "{{ instances.results }}"
      
      rescue:
        - name: Clean up on failure
          include_tasks: cleanup-infrastructure.yml
        
        - name: Fail pipeline
          fail:
            msg: "Infrastructure provisioning failed"

- name: Configure infrastructure
  hosts: newly_created
  become: yes
  
  roles:
    - common
    - security
    - monitoring
    - application
```

### Deployment Verification

```yaml
# filepath: examples/verify-deployment.yml
---
- name: Verify Deployment
  hosts: all
  gather_facts: yes
  
  vars:
    expected_version: "{{ lookup('env', 'APP_VERSION') }}"
    health_endpoint: "http://localhost:8080/health"
    metrics_endpoint: "http://localhost:8080/metrics"
  
  tasks:
    - name: Check application version
      uri:
        url: "{{ health_endpoint }}"
        return_content: yes
      register: health
    
    - name: Verify version
      assert:
        that:
          - health.json.version == expected_version
        fail_msg: "Version mismatch: expected {{ expected_version }}, got {{ health.json.version }}"
    
    - name: Check service status
      service_facts:
    
    - name: Verify services are running
      assert:
        that:
          - ansible_facts.services[item + '.service'].state == 'running'
        fail_msg: "Service {{ item }} is not running"
      loop:
        - myapp
        - nginx
    
    - name: Check application metrics
      uri:
        url: "{{ metrics_endpoint }}"
        return_content: yes
      register: metrics
    
    - name: Verify metrics are healthy
      assert:
        that:
          - metrics.json.error_rate | float < 0.05
          - metrics.json.response_time | float < 500
        fail_msg: "Application metrics are unhealthy"
    
    - name: Test database connectivity
      command: /opt/app/bin/check-db
      changed_when: false
    
    - name: Check disk space
      assert:
        that:
          - ansible_mounts | selectattr('mount', 'equalto', '/') | map(attribute='size_available') | first | int > 1000000000
        fail_msg: "Low disk space"
```

## Hands-On Lab

### Lab 1: Basic GitHub Actions Pipeline

```yaml
# filepath: lab/.github/workflows/deploy-app.yml
---
name: Deploy Application

on:
  push:
    branches: [main]
  workflow_dispatch:

jobs:
  deploy:
    runs-on: ubuntu-latest
    
    steps:
      - uses: actions/checkout@v3
      
      - name: Set up Python
        uses: actions/setup-python@v4
        with:
          python-version: '3.10'
      
      - name: Install Ansible
        run: |
          pip install ansible
          ansible-galaxy collection install -r requirements.yml
      
      - name: Deploy application
        env:
          ANSIBLE_HOST_KEY_CHECKING: 'false'
        run: |
          ansible-playbook deploy.yml \
            -i inventory/production.yml \
            -e "app_version=${{ github.sha }}"
```

### Lab 2: Multi-Stage Deployment

```yaml
# filepath: lab/multi-stage-deploy.yml
---
- name: Multi-stage deployment
  hosts: "{{ target_environment }}"
  become: yes
  
  vars:
    app_version: "{{ lookup('env', 'APP_VERSION') }}"
    stages:
      - name: pre-deployment
        tasks_file: pre-deploy.yml
      - name: deployment
        tasks_file: deploy.yml
      - name: post-deployment
        tasks_file: post-deploy.yml
      - name: verification
        tasks_file: verify.yml
  
  tasks:
    - name: Execute deployment stages
      include_tasks: "{{ item.tasks_file }}"
      loop: "{{ stages }}"
      loop_control:
        label: "{{ item.name }}"
```

### Lab 3: Rollback Playbook

```yaml
# filepath: lab/rollback.yml
---
- name: Rollback Application
  hosts: all
  become: yes
  serial: 2
  
  vars:
    rollback_version: "{{ lookup('env', 'ROLLBACK_VERSION') | default('previous') }}"
  
  tasks:
    - name: Get deployment history
      command: cat /opt/app/.deploy_history
      register: deploy_history
      changed_when: false
    
    - name: Determine rollback version
      set_fact:
        target_version: "{{ deploy_history.stdout_lines[-2] if rollback_version == 'previous' else rollback_version }}"
    
    - name: Remove from load balancer
      uri:
        url: "http://loadbalancer/api/pool/remove/{{ inventory_hostname }}"
        method: POST
      delegate_to: localhost
    
    - name: Stop application
      service:
        name: myapp
        state: stopped
    
    - name: Deploy previous version
      git:
        repo: https://github.com/company/app.git
        dest: /opt/app
        version: "{{ target_version }}"
    
    - name: Start application
      service:
        name: myapp
        state: started
    
    - name: Health check
      uri:
        url: http://localhost:8080/health
        status_code: 200
      retries: 10
      delay: 5
    
    - name: Add back to load balancer
      uri:
        url: "http://loadbalancer/api/pool/add/{{ inventory_hostname }}"
        method: POST
      delegate_to: localhost
```

## Best Practices

### 1. Version Control Everything
- Playbooks
- Inventory
- Variables
- Requirements files

### 2. Use Environments
- Separate staging and production
- Use environment-specific inventories
- Isolate credentials

### 3. Implement Gates
- Manual approval for production
- Automated testing before deploy
- Health checks after deploy

### 4. Monitor Deployments
- Log all changes
- Track deployment metrics
- Alert on failures

### 5. Plan for Rollback
- Always have a rollback plan
- Test rollback procedures
- Automate rollback when possible

## Summary

In this lesson, you learned:
- ✅ Integrating Ansible with CI/CD tools
- ✅ Building deployment pipelines
- ✅ Implementing deployment strategies
- ✅ Creating rollback procedures
- ✅ Verifying deployments
- ✅ CI/CD best practices

## Additional Resources

- [Ansible and CI/CD](https://docs.ansible.com/ansible/latest/reference_appendices/tower.html)
- [GitHub Actions Documentation](https://docs.github.com/en/actions)
- [GitLab CI Documentation](https://docs.gitlab.com/ee/ci/)
- [Jenkins Documentation](https://www.jenkins.io/doc/)

## Next Steps

Proceed to [Lesson 14: Best Practices and Advanced Topics](../14-best-practices/README.md) to learn advanced techniques and best practices.
