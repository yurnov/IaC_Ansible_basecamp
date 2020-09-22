# IaaC (Ansible) курсу DevOsp Basecamp for Telco

## Що таке Infrastructure as Code?

Infrastructure as code (IaC) is the process of managing and provisioning computer data centers through machine-readable definition files, rather than physical hardware configuration or interactive configuration tools.

The IT infrastructure managed by this process comprises both physical equipment, such as bare-metal servers, as well as virtual machines, and associated configuration resources. The definitions may be in a version control system. It can use either scripts or declarative definitions, rather than manual processes, but the term is more often used to promote declarative approaches.

## IaC Tools

| Tool      | Released by | Method        | Approach                   | Comments      |
|-----------|-------------|---------------|----------------------------|---------------|
| Chef      | Chef        | Pull          | Delcarative and imperative | Ruby          |
| Puppet    | Puppet      | Pull          | Declarative                | Ruby          |
| SaltStack | SaltStack   | Push and Pull | Delcarative and imparative | Python        |
| Terraform | HirashiCorp | Push          | Declarative                | Go, HCL, JSON |
| Ansible   | RedHat      | Push          | Declarative and imparative | Python, YAML  |

**Delcarative** = define WHAT end result you whant
**Imperative** = define exact steps - HOW

## Configuration managment

Configuration management (CM) is a systems engineering process for establishing and maintaining consistency of a product's performance, functional, and physical attributes with its requirements, design, and operational information throughout its life.

Configuration Managment allow is to automate and manage:
- infrastructure/platform
- services that run on that platform

## Ansible vs Terraform

| Ansible             | Terraform                      |
|---------------------|--------------------------------|
|Mainly configuration tool|Mainly infrastructure provisioning tool|
|more mature          |relatively new                  |
written in Python     |written in Go                   |
|better for configuring that infrastructure|better for infrastructure|
