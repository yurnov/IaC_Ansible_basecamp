## Variables

Ansible uses variables to manage differences between systems. With Ansible, you can execute tasks and playbooks on multiple different systems with a single command. To represent the variations among those different systems, you can create variables with standard YAML syntax, including lists and dictionaries. You can define these variables in your playbooks, in your inventory, in re-usable files or roles, or at the command line. You can also create variables during a playbook run by registering the return value or values of a task as a new variable.

After you create variables, either by defining them in a file, passing them at the command line, or registering the return value or values of a task as a new variable, you can use those variables in module arguments, in conditional “when” statements, in templates, and in loops. The ansible-examples github repository contains many examples of using variables in Ansible.

### Defining simple variables

You can define a simple variable using standard YAML syntax. For example:

```
foo: bar
```
### Referencing simple variables
After you define a variable, use Jinja2 syntax to reference it. Jinja2 variables use double curly braces. For example, following example will print `bar` (as value of `foo`):

```
debug:
  msg: "{{ foo }}"
```

To have valid YAML syntax variables must be quoted.

### Defining list variables

You can define variables with multiple values using YAML lists. For example:

```
region:
  - northeast
  - southeast
  - midwest
```

### Referencing list variables

When you use variables defined as a list (also called an array), you can use individual, specific fields from that list. The first item in a list is item 0, the second item is item 1. For example:

```
debug:
  msg: "{{ region[0] }}"
```

### Dictionary variables

A dictionary stores the data in key-value pairs. Usually, dictionaries are used to store related data, such as the information contained in an ID or a user profile.

#### Defining variables as key:value dictionaries
You can define more complex variables using YAML dictionaries. A YAML dictionary maps keys to values. For example:

```
foo:
  field1: one
  field2: two
```

#### Referencing key:value dictionary variables

When you use variables defined as a key:value dictionary (also called a hash), you can use individual, specific fields from that dictionary using either bracket notation or dot notation:

```
foo['field1']
foo.field1
```

Both of these examples reference the same value (“one”). Bracket notation always works. Dot notation can cause problems because some keys collide with attributes and methods of python dictionaries. Use bracket notation if you use keys which start and end with two underscores (which are reserved for special meanings in python) or are any of the known public attributes.

### Registering variables

You can create variables from the output of an Ansible task with the task keyword register. You can use registered variables in any later tasks in your play. For example:

```
- hosts: web_servers

  tasks:

     - shell: /usr/bin/foo
       register: foo_result
       ignore_errors: True

     - shell: /usr/bin/bar
       when: foo_result.rc == 5
```

For more examples of using registered variables in conditions on later tasks, see Conditionals. Registered variables may be simple variables, list variables, dictionary variables, or complex nested data structures. The documentation for each module includes a RETURN section describing the return values for that module. To see the values for a particular task, run your playbook with -v.

Registered variables are stored in memory. You cannot cache registered variables for use in future plays. Registered variables are only valid on the host for the rest of the current playbook run.

Registered variables are host-level variables. When you register a variable in a task with a loop, the registered variable contains a value for each item in the loop. The data structure placed in the variable during the loop will contain a `results` attribute, that is a list of all responses from the module.

### Defining variables from the file

Example of defining variables in play and in file:

```
---
- hosts: all
  remote_user: root
  vars:
    my_var1: foo
    my_var2: bar
    my_var3:
      - value1
      - value2
      - value3
  vars_files:
    - /path/external_vars.yml

  tasks:

  - name: this is just a placeholder
    module: ...
```

### Defining variable in runtime

You can define variables when you run your playbook by passing variables at the command line using the `--extra-vars` (or `-e`) argument:

```
ansible-palybook my_playbook.yaml -e "my_var1=foo my_var2=bar"
```

or (the same result):

```
ansible-palybook my_playbook.yaml --extra-vars '{"my_var1":"foo", "my_var2":"bar"}'
```

Vars can be defined from JSON file in runtime:

```
ansible-palybook my_playbook.yaml --extra-vars "@my_external_vars.json"
```

### Understanding variable priority

In general, Ansible gives precedence to variables that were defined more recently, more actively, and with more explicit scope. Variables in the the defaults folder inside a role are easily overridden. Anything in the vars directory of the role overrides previous versions of that variable in the namespace. Host and/or inventory variables override role defaults, but explicit includes such as the vars directory or an include_vars task override inventory variables.

## Future reading

- [Using variables](https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_variables.html)
