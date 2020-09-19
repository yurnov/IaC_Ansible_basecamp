# Матеріали модуля IaaC (Ansible) курсу DevOsp Basecamp for Telco

## Що таке Infrastructure as Code?

Infrastructure as code (IaC) is the process of managing and provisioning computer data centers through machine-readable definition files, rather than physical hardware configuration or interactive configuration tools.

The IT infrastructure managed by this process comprises both physical equipment, such as bare-metal servers, as well as virtual machines, and associated configuration resources. The definitions may be in a version control system. It can use either scripts or declarative definitions, rather than manual processes, but the term is more often used to promote declarative approaches.

## Інструменти IaC

| Tool      | Released by | Method        | Approach                   | Comments      |
|-----------|-------------|---------------|----------------------------|---------------|
| Chef      | Chef        | Pull          | Delcarative and imperative | Ruby          |
| Puppet    | Puppet      | Pull          | Declarative                | Ruby          |
| SaltStack | SaltStack   | Push and Pull | Delcarative and imparative | Python        |
| Terraform | HirashiCorp | Push          | Declarative                | Go, HCL, JSON |
| Ansible   | RedHat      | Push          | Declarative and imparative | Python, YAML  |

## Configuration managment

Configuration management (CM) is a systems engineering process for establishing and maintaining consistency of a product's performance, functional, and physical attributes with its requirements, design, and operational information throughout its life.