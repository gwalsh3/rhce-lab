# RHCE Lab - Ansible Automation on RHEL 10

A hands-on Ansible lab environment built on RHEL 10 VirtualBox VMs, used for
RHCE (EX294) exam preparation and ongoing automation reference.

## Lab Environment

| Host | Role | IP |
|------|------|----|
| rhel10-lab-rhce | Control Node | xxx.xxx.xxx.xxx |
| rhel10-lab-vm1 | Managed Node | yyy.yyy.yyy.yyy |
| rhel10-lab-vm2 | Managed Node | zzz.zzz.zzz.zzz |

- **Platform:** VirtualBox VMs on a local home lab
- **OS:** Red Hat Enterprise Linux 10
- **Ansible:** ansible-core (latest stable)
- **Operator User:** `rhce-admin` (dedicated non-root Ansible user)

## Repository Structure

```
rhce-lab/
├── ansible.cfg                      # Project-level Ansible configuration
├── inventory/
│   └── hosts                        # Static inventory - VirtualBox lab hosts
├── playbooks/
│   ├── bootstrap_ansible_user.yml   # Bootstrap rhce-admin on control node
│   ├── bootstrap_managed_nodes.yml  # Bootstrap rhce-admin on managed nodes
│   ├── update_packages.yml          # Update all packages on managed nodes; reboots if required
│   └── update_control_node.yml      # Update the Ansible control node itself; reboots if required
├── roles/                           # Custom roles (added as lab progresses)
├── group_vars/                      # Group variable files
└── host_vars/                       # Host variable files
```
## Bootstrap Setup

These playbooks automate the initial lab environment setup. They are designed
to be idempotent — safe to re-run at any time.

### 1. Bootstrap the Control Node

Creates the `rhce-admin` operator user, `ansible` group, sudoers config,
working directory structure, `ansible.cfg`, and SSH keypair.

```bash
# Run as root on the control node
ansible-playbook playbooks/bootstrap_ansible_user.yml
```

### 2. Bootstrap the Managed Nodes

Creates `rhce-admin` on each managed node and pushes the control node's
public key to enable passwordless SSH.

```bash
# Run as root on the control node
ansible-playbook -i inventory/hosts playbooks/bootstrap_managed_nodes.yml \
  --ask-pass --ask-become-pass
```

### 3. Verify Connectivity

```bash
cd ~/ansible
ansible -i inventory/hosts RHEL10_VMs -m ping
```

Expected output:
RHEL10-VM1 | SUCCESS => { "ping": "pong" }
RHEL10-VM2 | SUCCESS => { "ping": "pong" }

## Package Management

### Update Managed Nodes

Updates all packages on the managed nodes. Installs `yum-utils` if not present,
then uses `needs-restarting` to detect whether a kernel or core library update
requires a reboot. Reboots automatically and waits for nodes to return if needed.

```bash
ansible-playbook -i inventory/hosts playbooks/update_packages.yml
```

### Update the Control Node

Updates the control node itself using `localhost` with a local connection —
no inventory required. Applies the same reboot logic as the managed node playbook.

> **Note:** `ansible-core` on the control node is managed by `dnf` (rpm). Do not
> use pip to manage Ansible on this host, as it will conflict with the rpm install.

```bash
ansible-playbook playbooks/update_control_node.yml
```

## Usage

All Ansible commands should be run from the project root as `rhce-admin`
to ensure the correct `ansible.cfg` is loaded:

```bash
# Always cd into the project directory first
cd ~/ansible

# Run a playbook
ansible-playbook -i inventory/hosts playbooks/your_playbook.yml

# Ad-hoc command against all VMs
ansible -i inventory/hosts RHEL10_VMs -m command -a "uptime"
```

## Prerequisites

- RHEL 10 VMs with network connectivity between control and managed nodes
- `ansible-core` installed on the control node
- Root SSH access to managed nodes for initial bootstrap only
- Recommended collections for full module support:

```bash
ansible-galaxy collection install ansible.posix
ansible-galaxy collection install community.crypto
ansible-galaxy collection install community.general
```

## Certification Context

This lab supports preparation for the **Red Hat Certified Engineer (RHCE)**
exam — EX294 — which tests proficiency in Ansible automation on RHEL systems.

Previously completed: **Red Hat Certified System Administrator (RHCSA)** — EX200 ✓

## Author

Greg Walsh — Technology Professional | JPMorgan Chase
AWS Certified | RHCSA (RHEL 10) | RHCE (In Progress)
