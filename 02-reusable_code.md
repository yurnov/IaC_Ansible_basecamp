# Reusable Ansible Playbook

## Roles intro

Roles are ways of automatically loading certain vars_files, tasks, and handlers based on a known file structure. Grouping content by roles also allows easy sharing of roles with other users

## Example Project and role folders structure

Project structure:
```
site.yml
webservers.yml
fooservers.yml
roles/
├── common/
│   ├── tasks/
│   ├── handlers/
│   ├── files/
│   ├── templates/
│   ├── vars/
│   ├── defaults/
│   └── meta/
└──  webservers/
    ├── tasks/
    ├── defaults/
    └── meta/
```

Role structure:
```
common/
├── README.md
├── defaults
│   └── main.yml
├── files
├── handlers
│   └── main.yml
├── meta
│   └── main.yml
├── tasks
│   └── main.yml
├── templates
├── tests
│   ├── inventory
│   └── test.yml
└── vars
    └── main.yml
```

Roles expect files to be in certain directory names. Roles must include at least one of these directories, however it is perfectly fine to exclude any which are not being used. When in use, each directory must contain a `main.yml` file, which contains the relevant content:

`tasks` - contains the main list of tasks to be executed by the role.
`handlers` - contains handlers, which may be used by this role or even anywhere outside this role.
`defaults` - default variables for the role.
`vars` - other variables for the role.
`files` - contains files which can be deployed via this role.
`templates` - contains templates which can be deployed via this role.
`meta` - defines some meta data for this role.

## Using Roles

The classic (original) way to use roles is via the roles: option for a given play:

```
---
- hosts: webservers
  roles:
    - common
    - webservers
```

As well, path to the role can be specified that is useful for common roles for a few projects:

```
---
- hosts: webservers
  roles:
    - common
    - webservers
    - "../../common/playbook/roles/my_common_role"
```

Other way is to use `import_role` or `include_role` in tasks:

```
---
- hosts: webservers
  tasks:
    - ansible.builtin.import_rol:
        name: example
    - ansible.builtin.include_role:
        name: example
    - ansible.builtin.debug:
        msg: "after we ran our role"
```

Difference between `import_role` or `include_role` is that `include_role` dynamically loads and executes a specified role as the task and `import_role` act much like the `role:`, so, it's more static.
## Additional reading: 
- [Creating reusable playbok](https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_reuse.html)
