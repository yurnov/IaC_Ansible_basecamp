# Ansible 

![Ansible logo](https://upload.wikimedia.org/wikipedia/commons/2/24/Ansible_logo.svg)

## Agenda
- What is Ansible
- Why Ansible
- Ansible Use Cases
- Arthitecture of Ansible
- Configuration Managment with Ansible

## What is Ansible

- Ansible is OpenSource configuration managment and provisioning tool, similar to Chef, Puppet, Salt, etc.
- Ansible uses SSH (in most common case) to connect your servers and run confugiried Tasks. Ansible let you control and configure multilpe nodes from a single machine.
- What makes Ansible different from other managment software is that Ansible is fully agent-less and use SSH infrastructure.
- Ansible project was founded in 2013 and bought by RedHat in 2015.

## Why Ansible

- **No Agent**. As long as machine can be ssh'd into and able to execute Python code, it can be configuried with Ansible and even more.
- **Idempodent**. Ansible whole achitecture is structured around the concept of idempodency. The core idea is that you only do things if they are needed and that things are repeatable without side effect.
- **Declarative**. Some of other configuration managment tool tend to be procedural do this and then do that and so on. Ansible works by you writing a description of desired state of machine and then Ansible takes steps to fulfill that description.
- **Thiny Learning Curve**. Ansible is quite easy to learn. It doesn't requrie any extra knowladge.

## Ansible Use Cases

- Provisioning
- Configuration Managment
- App deployment
- Continius delivery
- Orchestration
- Other

## Ansible architecture

![](img/ansible.png)

### Inventory

The Inventory is description of nodes that can be accessed by Ansible. By default, the inventory is described by a configuration file. The configuration file lists either the IP address or hostnames of each node that is accessiable by Ansible.

Commonly used prectice to assign hosts into groups such as web-servers, databases, etc. The inventory file can be in many format such as YAML, INI, etc.

### Example of inventory file

```
[web]
web01.example.com
web02.example.com
web03.example.com

[loadbalancer]
lb01.example.com
lb02.example.com

[db]
psql[01:03].example.com

[external:children]
web
loadbalancer
```
### Playbooks

Playbook are simple YAML files. These file are descriptions of the desired state of your systems. Ansible then does the hard work of getting your systems to that state no matter that whet state they are currently in. 

Playbook are simple to write and maintain as it written in a simple (close to natural) language and self-explanatory in most cases. 

**Playbook** contain **Plays**.

**Plays** contain **Tasks**

**Tasks** call **Modules**

### Exammple of simple ansible playbook

```
---
- hosts: web
  remote_user: root
  tasks:
    - name: Ensure Nginx is a latest version
      yum:
        name: httpd
        state: latest

    - name: Ensure Nginx is running
      systemd:
        name: nginx
        started: yes
        enabled: yes
```

Same playbook in other form:

```
---
- hosts: web
  remote_user: root
  tasks:
    - name: Ensure Nginx is a latest version
      yum: name=httpd state=latest

    - name: Ensure Nginx is running
      systemd: name=nginx started=yes enabled=yes
```

## Modules

Modules control system resources, packages, files or nearly anything else.

Over 450 modules ship with Ansible and enable regular users to easly work with complex systems.

Do you remember playbooks above? It use `yum` and `systemd` modules.

Each module is mostly standalone and can be written in standart scription lagnadge (such as Python, Perl, Ruby or Bash, most of modules written in Python). One of the giuding properties of modules is idempotency, whith means that even if an operation is repeated multiple times it will always place the system into the same state.

For example, task

```
    - name: Ensure Nginx is running
      systemd: name=nginx started=yes
```

will ensure that systemd service nginx is started, that means that service will start if it stopped and do nothing if it started already and several execution of this task will keep system in same desired state.

In addition to official modules 
