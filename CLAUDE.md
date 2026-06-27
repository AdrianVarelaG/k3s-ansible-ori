# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

An Ansible collection (`k3s.orchestration`) that provisions a K3s Kubernetes cluster across Debian/Ubuntu/RHEL/SUSE/Arch nodes and Raspberry Pi. This is a homelab fork of [k3s-io/k3s-ansible](https://github.com/k3s-io/k3s-ansible) with local customizations layered on top of the upstream roles (see "Local divergence" below).

## Commands

```bash
# Provision / configure the whole cluster (idempotent — re-run to reconfigure)
ansible-playbook playbooks/site.yml -i inventory.yml

# Upgrade K3s (set k3s_version in inventory.yml first)
ansible-playbook playbooks/upgrade.yml -i inventory.yml

# Tear down K3s on all nodes
ansible-playbook playbooks/reset.yml -i inventory.yml

# Rolling reboot
ansible-playbook playbooks/reboot.yml -i inventory.yml

# Run against a subset of hosts / a single role
ansible-playbook playbooks/site.yml -i inventory.yml --limit server
ansible-playbook playbooks/site.yml -i inventory.yml --tags <tag>

# Lint (must pass in CI — .github/workflows/lint.yml)
yamllint .
ansible-lint

# Build the Galaxy collection artifact (CI builds on push to master)
ansible-galaxy collection build

# Local 5-node test cluster (LibVirt/VirtualBox)
vagrant up
```

After a successful run the kubeconfig is merged into `~/.kube/config` under the `k3s-ansible` context: `kubectl config use-context k3s-ansible`.

## Architecture

`playbooks/site.yml` orchestrates three plays in order:

1. **Cluster prep** (all hosts): `prereq` → `airgap` → `raspberrypi`
2. **Servers** (`server` group): `k3s_server`
3. **Agents** (`agent` group): `k3s_agent`

Role responsibilities:
- `roles/prereq` — OS prerequisites: sysctls (IPv4 forwarding), SELinux deps, validates the inventory/Ansible version. Local additions live here (swap, avahi, rmem).
- `roles/airgap` — distributes pre-downloaded K3s binary + images when `airgap_dir` is set; otherwise a no-op.
- `roles/raspberrypi` — Pi-specific cgroup/boot fixes, dispatched per-OS via `tasks/prereq/<Distribution>.yml`.
- `roles/k3s_server` — installs the K3s server. **HA/topology is inferred from inventory**: multiple hosts in `server` ⇒ embedded etcd HA (requires an odd count), unless `use_external_database: true`. The first server bootstraps the cluster; others join.
- `roles/k3s_agent` — joins worker nodes to the server using the shared `token`.
- `roles/k3s_upgrade` — used by `upgrade.yml` to roll K3s versions node by node.

### Inventory is the control surface

`inventory.yml` is where all cluster config lives — there are no per-environment var files. Key vars under `k3s_cluster.vars`: `k3s_version`, `token` (cluster join secret), `ansible_user`, `api_endpoint`, and many optional knobs (`extra_server_args`, `extra_agent_args`, `use_external_database`, `airgap_dir`, `extra_manifests`, `kubeconfig`). `inventory-sample.yml` documents every supported var. Group membership in `server`/`agent` determines topology — there is no separate "HA mode" flag.

`ansible.cfg` sets `roles_path=./roles`, `become=True`, `host_key_checking=False`, and `private_key_file=~/.ssh/home_lab_wize` (homelab-specific SSH key). `inventory.yml` is the default inventory.

## Local divergence from upstream

This fork carries uncommitted/local changes — be aware they are not in upstream and follow upstream conventions when extending them:
- `roles/prereq/tasks/main.yml` adds: disabling swap on Ubuntu, raising `net.core.rmem_max`, and installing/configuring `avahi-daemon` (mDNS reflector) with a matching `roles/prereq/handlers/main.yaml`.
- `roles/storage/` currently contains only `readme.md` (reference notes on `lsblk`/`fdisk`/`mkfs`) — the role has **no tasks yet** and is not wired into any playbook.

## Conventions

- Always use FQCN module names (`ansible.builtin.*`, `ansible.posix.*`, `community.general.*`) — `ansible-lint` enforces this. Collection deps are declared in `galaxy.yml`.
- yamllint allows lines up to 180 chars (warning); `truthy` must be `true`/`false`; no octal values. See `.yamllint`.
- `ansible-lint` warn-list (non-blocking) is in `.ansible-lint`; everything else is an error.
- Requires Ansible 8.0+ / ansible-core 2.15+ (`meta/runtime.yml`).
- Roles must work when inventory hosts are aliased (see commit history) — reference hosts via `ansible_host`/groups, not raw inventory names.
