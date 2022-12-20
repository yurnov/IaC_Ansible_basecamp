# Ansible 

![Ansible logo](https://upload.wikimedia.org/wikipedia/commons/2/24/Ansible_logo.svg)

## Agenda

- What is Ansible
- Why Ansible
- Ansible Use Cases
- Ansible Concept
- Ansible Architecture
- Playbooks

## What is Ansible

- Ansible is OpenSource configuration managment and provisioning tool, similar to Chef, Puppet, Salt, etc.
- Ansible uses SSH (in most common case) to connect your servers and run confugiried Tasks. Ansible let you control and configure multilpe nodes from a single machine.
- What makes Ansible different from other managment software is that Ansible is fully agent-less and use SSH infrastructure.
- Ansible project was founded in 2013 and bought by RedHat in 2015.

[Ansible quckstart guide](https://www.ansible.com/resources/videos/quick-start-video)

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

## Ansible concept

These concepts are common to all uses of Ansible.

- Control node
- Managed nodes
- Inventory
- Modules
- Tasks
- Playbooks

### Control Node

Any machine with Ansible installed.

### Managed nodes

The network devices (and/or servers) you manage with Ansible. Managed nodes are also sometimes called “hosts”. Ansible is not installed on managed nodes. In general case managed nodes needs to be accessiable by ssh from control node and need to have Python installed.

### Inventory

A list of managed nodes. An inventory file is also sometimes called a “hostfile”. Your inventory can specify information like IP address for each managed node. An inventory can also organize managed nodes, creating and nesting groups for easier scaling.

The inventory file can be in many format such as YAML, INI, etc.

Example of inventory file 


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

Future reading: [how to build your inventory](https://docs.ansible.com/ansible/latest/user_guide/intro_inventory.html#intro-inventory)

### Modules

The units of code Ansible executes. Each module has a particular use, from administering users on a specific type of database to managing VLAN interfaces on a specific type of network device. You can invoke a single module with a task, or invoke several different modules in a playbook. For an idea of how many modules Ansible includes, take a look at the [list of all modules](https://docs.ansible.com/ansible/latest/collections/index_module.html).

Each module is mostly standalone and can be written in standart scription lagnadge (such as Python, Perl, Ruby or Bash, most of modules written in Python). One of the giuding properties of modules is idempotency, whith means that even if an operation is repeated multiple times it will always place the system into the same state.

For example, task

```
    - name: Ensure Nginx is running
      ansible.builtin.systemd: name=nginx started=yes
```

will ensure that systemd service nginx is started, that means that service will start if it stopped and do nothing if it started already and several execution of this task will keep system in same desired state.

In addition to official modules you may use a various of comunity supported or write your own.

Be aware that correct syntax is `collection_name.module_name` and can be  

### Tasks

The units of action in Ansible. You can execute a single task once with an ad-hoc command.

### Playbooks

Ordered lists of tasks, saved so you can run those tasks in that order repeatedly. Playbooks can include variables as well as tasks. Playbooks are written in YAML and are easy to read, write, share and understand. 

Furure reading: [Intro to Playbooks](https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_intro.html#about-playbooks)

## Ansible architecture and recap

![](img/ansible.png)

## Playbooks

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

Example playbook with multiple plays:

```
---
- hosts: web
  remote_user: root

  tasks:
  - name: ensure apache is at the latest version
    yum:
      name: httpd
      state: latest
  - name: write the apache config file
    template:
      src: /srv/httpd.j2
      dest: /etc/httpd.conf

- hosts: db
  remote_user: root

  tasks:
  - name: ensure postgresql is at the latest version
    yum:
      name: postgresql
      state: latest
  - name: ensure that postgresql is started
    service:
      name: postgresql
      state: started
```

### Selecting an Ansible package and version to install

Ansible’s community packages are distributed in two ways: a minimalist language and runtime package called `ansible-core`, and a much larger “batteries included” package called `ansible`, which adds a community-curated selection of Ansible Collections for automating a wide variety of devices. Choose the package that fits your needs. It's easier to use `ansible`, but prom security point of view it better to use `ansible-core` with additional Collections and dependency that you need for your Project.

Please find the differnce between two methonds:

`ansible-core`:
```
#python3 -m pip install ansible-core
...
#python3 -m pip list
Package      Version
------------ -------
ansible-core 2.13.6
cffi         1.15.1
cryptography 38.0.4
Jinja2       3.1.2
MarkupSafe   2.1.1
packaging    21.3
pip          20.0.2
pycparser    2.21
pyparsing    3.0.9
PyYAML       6.0
resolvelib   0.8.1
setuptools   45.2.0
wheel        0.34.2
# ansible-galaxy collection list
usage: ansible-galaxy [-h] [--version] [-v] TYPE ...

Perform various Role and Collection related operations.

positional arguments:
  TYPE
    collection   Manage an Ansible Galaxy collection.
    role         Manage an Ansible Galaxy role.

optional arguments:
  --version      show program's version number, config file location, configured module search path, module location,
                 executable location and exit
  -h, --help     show this help message and exit
  -v, --verbose  Causes Ansible to print more debug messages. Adding multiple -v will increase the verbosity, the
                 builtin plugins currently evaluate up to -vvvvvv. A reasonable level to start is -vvv, connection
                 debugging might require -vvvv.
ERROR! - None of the provided paths were usable. Please specify a valid path with --collections-path
```

`ansible`:
```
#python3 -m pip install ansible
...
#python3 -m pip list
Package      Version
------------ -------
ansible      6.6.0
ansible-core 2.13.6
cffi         1.15.1
cryptography 38.0.4
Jinja2       3.1.2
MarkupSafe   2.1.1
packaging    21.3
pip          20.0.2
pycparser    2.21
pyparsing    3.0.9
PyYAML       6.0
resolvelib   0.8.1
setuptools   45.2.0
wheel        0.34.2
#ansible-galaxy collection list

# /usr/local/lib/python3.8/dist-packages/ansible_collections
Collection                    Version
----------------------------- -------
amazon.aws                    3.5.0
ansible.netcommon             3.1.3
ansible.posix                 1.4.0
ansible.utils                 2.7.0
ansible.windows               1.12.0
arista.eos                    5.0.1
awx.awx                       21.8.0
azure.azcollection            1.14.0
check_point.mgmt              2.3.0
chocolatey.chocolatey         1.3.1
cisco.aci                     2.3.0
cisco.asa                     3.1.0
cisco.dnac                    6.6.0
cisco.intersight              1.0.20
cisco.ios                     3.3.2
cisco.iosxr                   3.3.1
cisco.ise                     2.5.8
cisco.meraki                  2.11.0
cisco.mso                     2.1.0
cisco.nso                     1.0.3
cisco.nxos                    3.2.0
cisco.ucs                     1.8.0
cloud.common                  2.1.2
cloudscale_ch.cloud           2.2.2
community.aws                 3.6.0
community.azure               1.1.0
community.ciscosmb            1.0.5
community.crypto              2.8.1
community.digitalocean        1.22.0
community.dns                 2.4.0
community.docker              2.7.1
community.fortios             1.0.0
community.general             5.8.0
community.google              1.0.0
community.grafana             1.5.3
community.hashi_vault         3.4.0
community.hrobot              1.6.0
community.libvirt             1.2.0
community.mongodb             1.4.2
community.mysql               3.5.1
community.network             4.0.1
community.okd                 2.2.0
community.postgresql          2.3.0
community.proxysql            1.4.0
community.rabbitmq            1.2.3
community.routeros            2.3.1
community.sap                 1.0.0
community.sap_libs            1.3.0
community.skydive             1.0.0
community.sops                1.4.1
community.vmware              2.10.1
community.windows             1.11.1
community.zabbix              1.8.0
containers.podman             1.9.4
cyberark.conjur               1.2.0
cyberark.pas                  1.0.14
dellemc.enterprise_sonic      1.1.2
dellemc.openmanage            5.5.0
dellemc.os10                  1.1.1
dellemc.os6                   1.0.7
dellemc.os9                   1.0.4
f5networks.f5_modules         1.20.0
fortinet.fortimanager         2.1.6
fortinet.fortios              2.1.7
frr.frr                       2.0.0
gluster.gluster               1.0.2
google.cloud                  1.0.2
hetzner.hcloud                1.8.2
hpe.nimble                    1.1.4
ibm.qradar                    2.1.0
ibm.spectrum_virtualize       1.10.0
infinidat.infinibox           1.3.7
infoblox.nios_modules         1.4.0
inspur.ispim                  1.2.0
inspur.sm                     2.3.0
junipernetworks.junos         3.1.0
kubernetes.core               2.3.2
lowlydba.sqlserver            1.0.4
mellanox.onyx                 1.0.0
netapp.aws                    21.7.0
netapp.azure                  21.10.0
netapp.cloudmanager           21.21.0
netapp.elementsw              21.7.0
netapp.ontap                  21.24.1
netapp.storagegrid            21.11.1
netapp.um_info                21.8.0
netapp_eseries.santricity     1.3.1
netbox.netbox                 3.8.1
ngine_io.cloudstack           2.2.4
ngine_io.exoscale             1.0.0
ngine_io.vultr                1.1.2
openstack.cloud               1.10.0
openvswitch.openvswitch       2.1.0
ovirt.ovirt                   2.3.1
purestorage.flasharray        1.14.0
purestorage.flashblade        1.10.0
purestorage.fusion            1.1.1
sensu.sensu_go                1.13.1
servicenow.servicenow         1.0.6
splunk.es                     2.1.0
t_systems_mms.icinga_director 1.31.4
theforeman.foreman            3.7.0
vmware.vmware_rest            2.2.0
vultr.cloud                   1.3.0
vyos.vyos                     3.0.1
wti.remote                    1.0.4
#
```

The `ansible` or `ansible-core` packages may be available in your operating systems package manager, and you are free to install these packages with your preferred method. Official [installation instructions](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html) covers only installation with `pip`.