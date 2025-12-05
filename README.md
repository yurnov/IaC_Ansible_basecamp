# IaC Ansible Basecamp (Proposed refactor for 2025)

This repository contains training materials for an Ansible course. This branch/proposal modernizes the training for 2025 by:

- Organizing lessons into per-lesson directories under `lessons/`
- Adding a lesson template and syllabus information
- Adding CI checks (yamllint, ansible-lint) and stubs for Molecule testing
- Adding an example demo role with a Molecule scenario (demos/demo-01-nginx)
- Adding lint configs and a migration plan

Quick goals
- Make examples runnable and CI-checked
- Use modern Ansible practices (ansible-core + collections)
- Provide a consistent lesson format for instructors

Quick start (developer)
1. Clone and create a virtualenv
   ```bash
   python -m venv .venv
   source .venv/bin/activate
   pip install -U pip
   pip install ansible-core ansible-lint yamllint molecule docker molecule-docker
   ```
2. Install collections/roles referenced in `requirements.yml`:
   ```bash
   ansible-galaxy collection install -r requirements.yml
   ansible-galaxy role install -r requirements.yml || true
   ```

Repository layout (proposed)
- lessons/
  - 00-intro-yaml/README.md
  - 01-intro-ansible/README.md
  - ...
- demos/
  - demo-01-nginx/
    - roles/nginx/...
    - molecule/...
- .github/workflows/ci.yml
- .yamllint
- .ansible-lint
- requirements.yml
- MIGRATION.md (maps original files -> new locations)

How I suggest we proceed
1. Review this proposed content (below).
2. I open a PR implementing these files and moving/renaming the existing lesson files into `lessons/` directories (I can either move them as-is or convert them to the lesson template format).
3. Iterate on content edits or run CI & fix lints.

If you want, I can:
- Open the PR that implements everything here (including moving all lesson files).
- Or open only the CI + template + demo PR first, then follow with the content migration PR.

Tell me which you prefer and if you'd like me to include full converted lesson contents in the PR.
