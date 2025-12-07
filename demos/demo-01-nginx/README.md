# Demo 01 — Install and verify Nginx (molecule-based demo)

This demo contains a small role `nginx` and a Molecule scenario to verify that the role converges.

Quick steps to run locally (Linux with Docker):
1. Create and activate a virtualenv and install dependencies:
   ```bash
   python -m venv .venv
   source .venv/bin/activate
   pip install -U pip
   pip install docker molecule molecule-plugins[container] ansible-core
   ```
2. Run molecule:
   ```bash
   cd demos/demo-01-nginx
   molecule test
   ```
Notes
- The molecule scenario uses the Docker driver. Ensure Docker is available in your environment.
- This demo is intentionally minimal; expand tasks and tests to match lesson objectives.
