# Kubernetes Cluster Setup with Ansible

Set up a Kubernetes cluster on Arch Linux with CRI-O using Ansible.

> **Note**: This project only works on Arch Linux.

## Prerequisites

- SSH access to all nodes with root privileges
- SSH key configured (`~/.ssh/id_ed25519` or update `hosts.ini`)
- Ansible installed locally
- Copy `hosts.ini.sample` to `hosts.ini` and configure your nodes

## Quick Start

```bash
# 1. Configure hosts
cp hosts.ini.sample hosts.ini
vim hosts.ini

# 2. Install dependencies on all nodes
ansible-playbook -i hosts.ini 01-install-deps.yml --ask-become-pass

# 3. Initialize control plane
ansible-playbook -i hosts.ini 02-setup-control-plane.yml --ask-become-pass

# 4. Join worker nodes
ansible-playbook -i hosts.ini 03-join-workers.yml --ask-become-pass

# 5. Install CNI (choose one)
ansible-playbook -i hosts.ini 04-install-cilium.yml --ask-become-pass

# 5a. Install Longhorn dependencies (required before Longhorn)
ansible-playbook -i hosts.ini 04a-install-longhorn-deps.yml --ask-become-pass

# 6. Install apps (Longhorn, Cert-Manager, Ingress-Nginx)
ansible-playbook -i hosts.ini 05-install-apps.yml --ask-become-pass

# 7. Remove NoSchedule taint from control plane (allows scheduling pods on control plane)
ansible-playbook -i hosts.ini 06-remove-noschedule.yml --ask-become-pass
```

## Playbooks

| Order | Playbook | Description |
|-------|----------|-------------|
| 1 | `01-install-deps.yml` | Install CRI-O, kubeadm, kubectl, kubelet on all nodes |
| 2 | `02-setup-control-plane.yml` | Initialize first control plane node |
| 3 | `03-join-workers.yml` | Join worker nodes to cluster |
| 3b | `03b-join-control-plane.yml` | Join additional control plane nodes (HA) |
| 4 | `04-install-cilium.yml` | Install Cilium CNI |
| 4a | `04a-install-longhorn-deps.yml` | Install Longhorn dependencies (required before Longhorn) |
| 5 | `05-install-apps.yml` | Install Longhorn, Cert-Manager, Ingress-Nginx |
| 6 | `06-remove-noschedule.yml` | Remove NoSchedule taint from control plane |

## HA Multi-Master Setup

```bash
# After 02-setup-control-plane.yml completes:
ansible-playbook -i hosts.ini 03b-join-control-plane.yml --ask-become-pass
# Repeat for each additional control plane node
```

## Verify Cluster

```bash
# SSH to control plane
ssh root@kube1.kube.internal

# Check nodes
kubectl get nodes

# Check all pods
kubectl get pods -A
```

## Configuration

Edit `hosts.ini`:

```ini
[kube]           # All Kubernetes nodes
kube1.kube.internal
kube2.kube.internal
kube3.kube.internal

[main]           # Control plane nodes
kube1.kube.internal

[workers]        # Worker nodes (optional group)
kube2.kube.internal
kube3.kube.internal

[kube:vars]
ansible_user=root
ansible_ssh_private_key_file=~/.ssh/id_ed25519
```

## Troubleshooting

```bash
# View kubelet logs
ssh root@kube1.kube.internal "journalctl -u kubelet -f"

# Reset cluster (run on all nodes)
ansible-playbook -i hosts.ini wipe-kube.yml --ask-become-pass
```

## Cluster Details

- Pod subnet: `10.244.0.0/16`
- Service subnet: `10.96.0.0/12`
- CRI socket: `unix:///var/run/crio/crio.sock`

## Support

For on-prem Kubernetes help, reach me at: https://t.me/metantesan
