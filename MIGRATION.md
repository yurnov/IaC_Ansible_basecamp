# Migration plan (proposed)

This file maps current repository files -> proposed new structure and lists the minimal edits to perform during the PR.

Planned moves and edits
- Rename file (fix typo):
  - `05-conditonals_and_loops.md` -> `lessons/05-conditionals_and_loops/README.md`
- Move existing lesson files into `lessons/` (move only; content preserved initially; then refactor to template):
  - `00-Introduction_to_YAML.md` -> `lessons/00-intro-yaml/README.md`
  - `01-Intro_to_Ansible.md` -> `lessons/01-intro-ansible/README.md`
  - `02-reusable_code.md` -> `lessons/02-reusable-code/README.md`
  - `03-Variables.md` -> `lessons/03-variables/README.md`
  - `04-jinja2_and_templates.md` -> `lessons/04-jinja2-and-templates/README.md`
  - `05-conditonals_and_loops.md` -> `lessons/05-conditionals_and_loops/README.md` (rename to fix typo)
  - `06-Ansible-Vault.md` -> `lessons/06-ansible-vault/README.md`
  - `08-homework.md` -> `lessons/08-homework/README.md`
- Create `lessons/lesson-template/README.md` as the canonical format for future lessons.
- Add `demos/demo-01-nginx/` containing a role and Molecule scenario.

Migration notes
- Preserve original files in the first PR (add copies under `lessons/`), then in a follow-up PR remove the root-level files to keep history tidy.
- Adjust internal links to point to new paths.
- Run ansible-lint and yamllint and fix issues iteratively (CI will initially have `|| true` to avoid blocking PRs).
- Instructor-only solutions will be placed in a `solutions/` directory or separate branch (not included in this PR).
