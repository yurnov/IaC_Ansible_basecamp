# Lesson NN — Lesson Title (template)

Estimated time: 30–60 minutes  
Level: Beginner / Intermediate / Advanced

## Learning objectives
- Brief, specific objective 1
- Brief, specific objective 2
- What learners will be able to do after the lesson

## Prerequisites
- Software: Python 3.10+, ansible-core X.Y+, pip
- A virtual environment and required collections (see top-level README)
- Basic knowledge: YAML, Linux shell

## Outline
1. Concept: concise explanation
2. Demo: step-by-step playbook walkthrough
3. Hands-on: exercises with expected outputs
4. Summary and further reading

## Files for this lesson
- playbooks/ — example playbooks used in the lesson
- roles/ — sample roles (if applicable)
- inventory/ — sample inventory and host_vars/group_vars
- solutions/ — instructor-only solutions (optional)

## Hands-on exercise (sample)
- Exercise: Deploy a simple role that installs and configures X
- Steps:
  1. Run: `ansible-playbook -i inventory/hosts playbooks/site.yml`
  2. Verify with: `ansible -i inventory/hosts all -m command -a 'systemctl status foo'`

## Instructor notes
- Estimated live demo time
- Common gotchas
- Which parts to skip or extend depending on audience level
