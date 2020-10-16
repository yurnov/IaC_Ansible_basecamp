## Conditionals 

In a playbook, you may want to execute different tasks, or have different goals, depending on the value of a fact (data about the remote system), a variable, or the result of a previous task. You may want the value of some variables to depend on the value of other variables. Or you may want to create additional groups of hosts based on whether the hosts match other criteria. You can do all of these things with conditionals.

Ansible uses Jinja2 tests and filters in conditionals. Ansible supports all the standard tests and filters, and adds some unique ones as well.

Example of conditionals:

```
tasks:
  - name: Register a variable, ignore errors and continue
    command: /bin/false
    register: result
    ignore_errors: true

  - name: Run only if the task that registered the "result" variable fails
    command: /bin/something
    when: result is failed

  - name: Run only if the task that registered the "result" variable succeeds
    command: /bin/something_else
    when: result is succeeded

  - name: Run only if the task that registered the "result" variable is skipped
    command: /bin/still/something_else
    when: result is skipped
```

## Loops

Sometimes you want to repeat a task multiple times. In computer programming, this is called a loop.

Example of sіmple loop:

```
- debug:
    mgs: "{{ item }}"
  with_items:
    - one
    - two
    - three
```

example of more complex loop:

```
- name: Add several users
  ansible.builtin.user:
    name: "{{ item.name }}"
    state: present
    groups: "{{ item.groups }}"
  loop:
    - { name: 'testuser1', groups: 'wheel' }
    - { name: 'testuser2', groups: 'root' }
```

## Future reading
[Playbooks conditionals](https://docs.ansible.com/ansible/latest/user_guide/playbooks_conditionals.html)
[Playbook loops](https://docs.ansible.com/ansible/latest/user_guide/playbooks_loops.html)
