# IaC Ansible Basecamp

A comprehensive, hands-on course for learning Ansible from fundamentals to advanced topics. This bootcamp-style training takes you from zero to proficient in Infrastructure as Code (IaC) using Ansible.

## 🎯 Course Overview

This course is designed for system administrators, DevOps engineers, and developers who want to master Ansible for infrastructure automation, configuration management, and application deployment.

### What You'll Learn

- YAML syntax and Ansible fundamentals
- Writing playbooks and using ad-hoc commands
- Managing inventory and variables
- Creating dynamic configurations with Jinja2 templates
- Implementing conditionals, loops, and error handling
- Building reusable roles and collections
- Securing sensitive data with Ansible Vault
- Testing Ansible code with Molecule and other tools
- Integrating Ansible into CI/CD pipelines
- Applying best practices for production environments

## 📚 Course Structure

The course consists of 15 progressive lessons, each building on previous knowledge:

### Foundation (Lessons 00-02)
- **[Lesson 00: Prerequisites and Environment Setup](lessons/00-prerequisites/README.md)**  
  Set up your Ansible development environment and verify installation

- **[Lesson 01: YAML Fundamentals](lessons/01-yaml-fundamentals/README.md)**  
  Master YAML syntax, data types, and common pitfalls

- **[Lesson 02: Ansible Basics and Ad-Hoc Commands](lessons/02-ansible-basics/README.md)**  
  Understand Ansible architecture and execute ad-hoc commands

### Core Concepts (Lessons 03-05)
- **[Lesson 03: Inventory Management](lessons/03-inventory-management/README.md)**  
  Organize hosts and groups using static and dynamic inventories

- **[Lesson 04: Playbooks Fundamentals](lessons/04-playbooks-fundamentals/README.md)**  
  Write structured playbooks with tasks, plays, and handlers

- **[Lesson 05: Variables and Facts](lessons/05-variables-facts/README.md)**  
  Manage data with variables, facts, and variable precedence

### Intermediate Topics (Lessons 06-08)
- **[Lesson 06: Templates with Jinja2](lessons/06-templates-jinja2/README.md)**  
  Create dynamic configuration files using Jinja2 templating

- **[Lesson 07: Conditionals and Loops](lessons/07-conditionals-loops/README.md)**  
  Implement control flow with when statements and loops

- **[Lesson 08: Handlers and Error Handling](lessons/08-handlers-errors/README.md)**  
  Manage service states and implement error recovery strategies

### Advanced Concepts (Lessons 09-11)
- **[Lesson 09: Ansible Roles](lessons/09-ansible-roles/README.md)**  
  Organize playbooks into reusable, shareable roles

- **[Lesson 10: Ansible Collections](lessons/10-ansible-collections/README.md)**  
  Package and distribute Ansible content as collections

- **[Lesson 11: Ansible Vault and Security](lessons/11-vault-security/README.md)**  
  Encrypt sensitive data and implement security best practices

### Professional Practices (Lessons 12-14)
- **[Lesson 12: Testing Ansible Code](lessons/12-testing/README.md)**  
  Test roles and playbooks with Molecule, ansible-lint, and other tools

- **[Lesson 13: CI/CD with Ansible](lessons/13-cicd/README.md)**  
  Integrate Ansible into continuous delivery pipelines

- **[Lesson 14: Best Practices and Advanced Topics](lessons/14-best-practices/README.md)**  
  Optimize performance and apply production-ready patterns

## 🚀 Getting Started

### Prerequisites

- Basic Linux command-line knowledge
- SSH familiarity
- Text editor experience (vim, nano, VS Code, etc.)
- Understanding of basic DevOps concepts (helpful but not required)

### System Requirements

- **Control Node**: Linux, macOS, or WSL2 on Windows
- **Python**: 3.9 or later
- **Ansible**: 2.9 or later (ansible-core recommended)
- **Test Environment**: Access to 2-3 virtual machines or containers

### Quick Start

1. **Clone this repository**
   ```bash
   git clone https://github.com/yurnov/IaC_Ansible_basecamp.git
   cd IaC_Ansible_basecamp
   ```

2. **Set up your environment**
   ```bash
   # Create virtual environment
   python3 -m venv ansible-venv
   source ansible-venv/bin/activate

   # Install Ansible and tools
   pip install ansible ansible-lint yamllint
   
   # Install collections
   ansible-galaxy collection install -r requirements.yml
   ```

3. **Verify installation**
   ```bash
   ansible --version
   ansible localhost -m ping
   ```

4. **Start with Lesson 00**
   ```bash
   cd lessons/00-prerequisites
   # Follow the README.md instructions
   ```

## 📖 How to Use This Course

### For Self-Paced Learning

1. **Follow the lesson order** - Each lesson builds on previous knowledge
2. **Complete all labs** - Hands-on practice is essential
3. **Experiment freely** - Try variations of the examples
4. **Review summaries** - Ensure you understand key concepts before proceeding

### For Instructors

- Each lesson includes clear objectives and time estimates
- Labs can be demonstrated live or assigned as exercises
- Examples are tested and production-ready
- Additional resources provide deeper exploration

### Lesson Format

Each lesson follows a consistent structure:

- **Objectives**: What you will learn in this lesson
- **Prerequisites**: Knowledge and setup required before starting
- **Introduction**: Overview of the lesson content
- **Tasks**: Step-by-step instructions to achieve the lesson objectives
- **Summary**: Recap of what you learned and links to additional resources
