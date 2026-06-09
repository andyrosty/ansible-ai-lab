Ansible AI Lab
==============

This repository contains Ansible playbooks for preparing and operating Debian/Ubuntu hosts as a lightweight AI lab for development, experiments, and demos.

It currently supports:

- Host connectivity and environment preflight checks
- Base system + Python AI stack bootstrap
- Swap provisioning for memory-constrained machines
- Ollama installation, model pull, and test run
- Runtime status checks for memory, swap, Ollama service, API, and models

Prerequisites
-------------

- Control machine with Ansible installed
- Target hosts running Debian/Ubuntu
- SSH access to targets with a user that can escalate with `sudo`
- Internet access on target hosts (for apt, pip, and Ollama/model downloads)

Repository layout
-----------------

- `inventory/hosts.yml`: Host/group inventory (default is group `ai_lab`)
- `playbooks/preflight.yml`: Connectivity and host sanity checks
- `playbooks/bootstrap.yml`: Base package + Python environment bootstrap
- `playbooks/swap.yml`: Swap file creation and persistence
- `playbooks/ollama.yml`: Ollama install/start, model pull, and smoke test
- `playbooks/status.yml`: Runtime checks for system and Ollama
- `ansible.cfg`: Default inventory path and Ansible behavior
- `Makefile`: Shortcuts for common commands

Inventory
---------

Default inventory: `inventory/hosts.yml`

```yaml
all:
  children:
    ai_lab:
      hosts:
        ai-lab-01:
          ansible_host: 192.168.50.160
          ansible_user: andrew
```

Update `ansible_host` and `ansible_user` for your environment, and add more hosts under `ai_lab` as needed.

Configuration defaults
----------------------

`ansible.cfg` is set up to:

- Use `inventory/hosts.yml` by default
- Disable host key checking
- Disable retry files
- Auto-detect Python interpreter on hosts (`auto_silent`)

Quick start
-----------

1. Validate connectivity:

   ```bash
   make ping
   ```

2. Run preflight checks:

   ```bash
   make preflight
   ```

3. Bootstrap AI tooling:

   ```bash
   make bootstrap
   ```

Optional next steps:

```bash
ansible-playbook -i inventory/hosts.yml playbooks/swap.yml
ansible-playbook -i inventory/hosts.yml playbooks/ollama.yml
ansible-playbook -i inventory/hosts.yml playbooks/status.yml
```

Playbooks
---------

### `preflight.yml`

Runs basic host checks:

- Gathers facts and prints hostname/OS/kernel/CPU/memory/default IPv4
- Displays routes via `ip route`
- Checks DNS resolution with `getent hosts archive.ubuntu.com`

Run:

```bash
ansible-playbook -i inventory/hosts.yml playbooks/preflight.yml
```

### `bootstrap.yml`

Prepares a general AI-ready baseline:

- Updates apt cache
- Installs core packages (`curl`, `git`, `build-essential`, `python3-venv`, etc.)
- Creates `/opt/ai-lab`
- Creates venv at `/opt/ai-lab/.venv`
- Upgrades `pip`, `wheel`, `setuptools`
- Installs starter Python packages:
  - `jupyterlab`
  - `numpy`, `pandas`, `matplotlib`
  - `requests`
  - `transformers`, `sentence-transformers`, `accelerate`
  - `torch`

Run:

```bash
ansible-playbook -i inventory/hosts.yml playbooks/bootstrap.yml
```

### `swap.yml`

Creates and enables swap (default: 8 GB at `/swapfile`):

- Creates swap file if missing
- Sets secure permissions
- Formats and enables swap
- Persists mount in `/etc/fstab`
- Prints `free -h` output

Run:

```bash
ansible-playbook -i inventory/hosts.yml playbooks/swap.yml
```

### `ollama.yml`

Installs and verifies Ollama:

- Installs Ollama if not already present
- Ensures `ollama` service is enabled and running
- Displays installed Ollama version
- Pulls starter model `llama3.2:1b`
- Runs a one-line model test prompt

Run:

```bash
ansible-playbook -i inventory/hosts.yml playbooks/ollama.yml
```

### `status.yml`

Checks runtime lab status:

- Shows memory/swap (`free -h`)
- Reads Ollama systemd service state
- Calls local Ollama API (`/api/tags`)
- Lists local models (`ollama list`)

Run:

```bash
ansible-playbook -i inventory/hosts.yml playbooks/status.yml
```

Make targets
------------

Available shortcuts:

- `make help`
- `make ping`
- `make preflight`
- `make bootstrap`
- `make syntax-check`
- `make dry-run`

Examples:

```bash
make preflight PLAYBOOK_ARGS="--limit ai-lab-01"
make bootstrap PLAYBOOK_ARGS="--ask-become-pass --limit ai-lab-01"
```

`syntax-check` and `dry-run` currently cover `preflight.yml` and `bootstrap.yml`.

Customization
-------------

- Add host/group variables under `group_vars/ai_lab/`
- Add role-based structure under `roles/` as the setup grows
- Adjust package lists in `playbooks/bootstrap.yml`
- Change swap size/path in `playbooks/swap.yml` vars (`swap_size_mb`, `swap_file`)
- Change Ollama model in `playbooks/ollama.yml`

Notes
-----

- `bootstrap.yml` installs `torch` from PyPI; GPU/CUDA setups may need custom package indexes or versions.
- `ollama.yml` pulls a model, which can take time and bandwidth.
- `status.yml` assumes Ollama is installed and running on target hosts.

License
-------

This project is licensed under the terms in `LICENSE`.
