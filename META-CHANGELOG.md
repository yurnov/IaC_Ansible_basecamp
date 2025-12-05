# Proposed change summary (for PR description)

This file summarizes the set of changes that will be included in the initial refactor PR:

- Add lessons/lesson-template/README.md (new)
- Add MIGRATION.md with mapping of current -> new filenames
- Add CI workflow (.github/workflows/ci.yml) for yamllint/ansible-lint/molecule (initially non-blocking)
- Add .yamllint and .ansible-lint config stubs
- Add requirements.yml (collections)
- Add demos/demo-01-nginx with role + molecule scenario
- Update top-level README.md with quickstart and repo layout
- No content deletions in initial PR; moves will be copies to new locations. Optionally in a follow-up PR we can remove root-level MD files and fully convert to the lesson template format.

Rationale: keep initial PR small and reviewable, enable CI and runnable demos so we can iterate faster.
