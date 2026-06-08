Ansible AI Lab
==============

This repository contains a small Ansible configuration for bootstrapping and checking Linux hosts that you want to use as an "AI lab" (for development, experimentation, or demos).

The playbooks assume Debian/Ubuntu-based machines with `apt` available.

Project layout
--------------

- `ansible.cfg` – Local Ansible configuration.
- `inventory/hosts.yml` – Inventory with the `ai_lab` group and example host.
- `group_vars/ai_lab/` – Group‑level variables for hosts in the `ai_lab` group (currently empty, can be extended).
- `playbooks/preflight.yml` – Runs basic connectivity and environment checks.
- `playbooks/bootstrap.yml` – Installs base tools and Python AI packages on the target node(s).
- `roles/` – Reserved for reusable roles (none defined yet).

Requirements
------------

Control machine (where you run Ansible):

- Python 3.x
- Ansible 2.13+ (or any reasonably recent version)

Managed nodes (targets in the `ai_lab` group):

- Debian/Ubuntu-based Linux
- SSH access from the control machine
- A user account with sudo privileges (configured as `ansible_user` in the inventory)

Inventory
---------

The default inventory is at `inventory/hosts.yml` and looks like this:

```yaml
all:
  children:
    ai_lab:
      hosts:
        ai-lab-01:
          ansible_host: 192.168.50.160
          ansible_user: andrew
```

Update `ansible_host` and `ansible_user` to match your environment. Add more hosts under `ai_lab` as needed.

If you want to override the default remote user or other connection settings globally, you can also adjust `ansible.cfg`.

Playbooks
---------

### Preflight checks

Runs basic checks to ensure connectivity and sanity of the target node(s).

File: `playbooks/preflight.yml`

It:

- Gathers facts and prints a summary (hostname, OS, kernel, CPU, memory, IP).
- Shows the default network routes (`ip route`).
- Checks DNS resolution for `archive.ubuntu.com`.

Run:

```bash
ansible-playbook -i inventory/hosts.yml playbooks/preflight.yml
```

### Bootstrap AI lab node(s)

Installs base packages and prepares a Python environment for AI work.

File: `playbooks/bootstrap.yml`

It:

- Updates apt cache.
- Installs common utilities (curl, wget, git, vim, htop, unzip, etc.).
- Creates `/opt/ai-lab` owned by the connecting user.
- Creates a Python virtual environment at `/opt/ai-lab/.venv`.
- Upgrades `pip`, `wheel`, `setuptools` in the venv.
- Installs initial Python AI/data packages, including:
  - jupyterlab
  - numpy, pandas, matplotlib
  - requests
  - transformers, sentence-transformers, accelerate
  - torch

Run:

```bash
ansible-playbook -i inventory/hosts.yml playbooks/bootstrap.yml
```

Note: The `bootstrap` playbook assumes enough disk space and that installing `torch` from PyPI is acceptable. For GPUs or specific CUDA versions you may need to customize the package list.

Customization
-------------

- Add or override group-level variables in `group_vars/ai_lab/`.
- Add roles under `roles/` and include them from the playbooks if this setup grows.
- Modify the package lists in `playbooks/bootstrap.yml` to match your preferred stack.

Usage examples
--------------

Ping all AI lab nodes to verify connectivity:

```bash
ansible -i inventory/hosts.yml ai_lab -m ping
```

Run preflight checks:

```bash
ansible-playbook -i inventory/hosts.yml playbooks/preflight.yml
```

Bootstrap the environment:

```bash
ansible-playbook -i inventory/hosts.yml playbooks/bootstrap.yml
```

License
-------

This project is licensed under the terms described in `LICENSE`.
