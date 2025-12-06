# Lesson 00: Prerequisites and Environment Setup

## Lesson Objectives
By the end of this lesson, you will be able to:
- Set up a proper Ansible development environment
- Understand the basic tools and requirements for Ansible
- Verify your installation is working correctly
- Understand the course structure and expectations

## Prerequisites
- Basic Linux command-line knowledge
- SSH familiarity
- Text editor experience (vim, nano, VS Code, etc.)
- Basic understanding of YAML syntax (will be covered in next lesson)

## Duration
30 minutes

## Lesson Content

### What You'll Need

1. **Control Node Requirements**
   - Linux, macOS, or WSL2 on Windows
   - Python 3.9 or later
   - SSH client

2. **Managed Nodes Requirements**
   - SSH server running
   - Python 3.9 or later (for most modules)
   - Proper user accounts with sudo access

### Installing Ansible

#### Using pip (Recommended for Learning)

```bash
# Create a virtual environment
python3 -m venv ~/ansible-venv
source ~/ansible-venv/bin/activate

# Install ansible-core and common collections
pip install ansible-core
pip install ansible-lint yamllint

# Verify installation
ansible --version
```

#### Using Package Managers

```bash
# Ubuntu/Debian
sudo apt update
sudo apt install ansible

# macOS
brew install ansible

# RHEL/CentOS/Rocky
sudo dnf install ansible-core
```

### Verify Your Installation

```bash
# Check Ansible version
ansible --version

# Check Python version
python3 --version

# Test localhost connection
ansible localhost -m ping
```

Expected output:
```json
localhost | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
```

### Course Structure

This course is organized into progressive lessons:
- **Lessons 00-02**: Foundations (YAML, Ansible basics, inventory)
- **Lessons 03-05**: Core concepts (playbooks, variables, modules)
- **Lessons 06-08**: Intermediate (templates, handlers, conditionals, loops)
- **Lessons 09-11**: Advanced (roles, collections, error handling)
- **Lessons 12-14**: Best practices (testing, CI/CD, security)

### Lab Environment Setup

For this course, you'll need access to at least 2-3 test machines:
- 1 control node (your laptop/workstation)
- 2-3 managed nodes (VMs or containers)

#### Option 1: Using Docker (Easiest)

```bash
# Create test containers
docker run -d --name ansible-node1 \
  -p 2221:22 \
  ghcr.io/ansible-community/community-test-images/ubuntu2204:latest

docker run -d --name ansible-node2 \
  -p 2222:22 \
  ghcr.io/ansible-community/community-test-images/ubuntu2204:latest
```

#### Option 2: Using Vagrant

```ruby
# Vagrantfile
Vagrant.configure("2") do |config|
  (1..2).each do |i|
    config.vm.define "node#{i}" do |node|
      node.vm.box = "ubuntu/jammy64"
      node.vm.hostname = "ansible-node#{i}"
      node.vm.network "private_network", ip: "192.168.56.#{10+i}"
    end
  end
end
```

#### Option 3: Cloud VMs

Use any cloud provider (AWS, Azure, GCP, DigitalOcean) to create small VMs.

### Setting Up SSH Keys

```bash
# Generate SSH key if you don't have one
ssh-keygen -t ed25519 -C "ansible-training"

# Copy key to managed nodes
ssh-copy-id user@managed-node-ip
```

## Hands-On Lab

### Lab 1: Verify Ansible Installation

1. Check your Ansible version:
   ```bash
   ansible --version
   ```

2. Create a simple test:
   ```bash
   mkdir -p ~/ansible-training
   cd ~/ansible-training
   echo "localhost ansible_connection=local" > inventory.ini
   ansible -i inventory.ini localhost -m setup | head -20
   ```

### Lab 2: Test Managed Node Connectivity

1. Create an inventory file with your test nodes
2. Test connectivity using the ping module
3. Gather facts from all nodes

## Summary

In this lesson, you:
- ✅ Installed Ansible and verified the installation
- ✅ Set up a lab environment for practice
- ✅ Understood the course structure
- ✅ Tested basic connectivity to managed nodes

## Additional Resources

- [Ansible Installation Guide](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html)
- [Ansible Documentation](https://docs.ansible.com/)
- [Ansible Community](https://forum.ansible.com/)

## Next Steps

Proceed to [Lesson 01: YAML Fundamentals](../01-yaml-fundamentals/README.md) to learn the syntax used in Ansible playbooks.
