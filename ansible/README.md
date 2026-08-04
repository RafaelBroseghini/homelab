# ansible

Bootstraps the Raspberry Pi control-plane host with base packages, CLI tools, and a periodic update timer.

## Usage

```bash
ansible-playbook -v -i inventory.yaml playbook.yaml
```

Target host is `raspberrypi` under the `lab` group (see `inventory.yaml`). Ansible resolves it via SSH config / `/etc/hosts` — there is no explicit IP in the inventory.

## What the playbook installs

| Component | Notes |
|-----------|-------|
| System updates | `apt update && full-upgrade` on every run |
| fzf, zoxide, ripgrep | Shell productivity tools |
| ufw | Firewall |
| Go 1.23.6 | arm64 tarball to `/usr/local/go` |
| ArgoCD CLI | v2.14.2 arm64 binary |
| monitor.timer | Hourly apt update/upgrade/autoclean/autoremove |

## Systemd timer

`units/update/monitor.timer` fires `monitor.service` every hour (`OnCalendar=*-*-* *:00:00`). The service runs apt maintenance as a oneshot.

After editing unit files:

```bash
sudo systemctl daemon-reload
sudo systemctl restart monitor.timer
```

## Scope

This playbook prepares the **host OS**, not the k8s cluster. k3s installation and cluster bootstrap are outside this playbook — see the root README and [`k8s/README.md`](../k8s/README.md) for GitOps setup.
