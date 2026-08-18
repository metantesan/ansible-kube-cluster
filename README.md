# Kubernetes Cluster Setup with Ansible

Bootstrap a Kubernetes cluster with **CRI-O** and **kubeadm** using Ansible.
Works on **Arch Linux** (pacman), **RHEL/Fedora/Rocky** (RPM) and
**Debian/Ubuntu** (DEB). CRI-O and Kubernetes packages are installed from the
official repositories listed on [cri-o.io](https://cri-o.io).

## Layout

| Folder | Purpose |
|--------|---------|
| [`cni/`](cni/README.md) | CNI plugin options: Cilium or Flannel |
| [`storage/`](storage/README.md) | Storage options: Local Path Provisioner or Longhorn |
| [`loadbalancer/`](loadbalancer/README.md) | LoadBalancer options: MetalLB or Cilium LB-IPAM |
| [`flux/`](flux/README.md) | Flux GitOps helm-controller (installed via helm) |
| [`apps/`](apps/README.md) | Apps managed declaratively through Flux (`apps/<group>/<app>/`) |

## Prerequisites

- SSH access to all nodes with root privileges
- SSH key configured (`~/.ssh/id_ed25519` or update `hosts.ini`)
- Ansible installed locally
- Copy `hosts.ini.sample` to `hosts.ini` and configure your nodes
- Supported OS: Arch, RHEL/Fedora/Rocky, Debian/Ubuntu

## Quick Start

```bash
# 1. Configure hosts
cp hosts.ini.sample hosts.ini
vim hosts.ini

# 2. Install dependencies on all nodes (CRI-O + kubelet/kubeadm/kubectl)
ansible-playbook -i hosts.ini 01-install-deps.yml --ask-become-pass

# 3. Initialize control plane (runs on the [kube_admin] node)
ansible-playbook -i hosts.ini 02-setup-control-plane.yml --ask-become-pass

# 4. Join worker nodes
ansible-playbook -i hosts.ini 03-join-workers.yml --ask-become-pass

# 5. Install CNI (choose ONE option in the cni/ folder)
ansible-playbook -i hosts.ini cni/cilium/install-cilium.yml --ask-become-pass
# or
ansible-playbook -i hosts.ini cni/flannel/install-flannel.yml --ask-become-pass

# 6. Install Flux (GitOps helm-controller) - installed with helm install
ansible-playbook -i hosts.ini flux/install-flux.yml --ask-become-pass

# 7. Install apps through Flux (see apps/README.md, ingress options in apps/ingress/README.md)
# Ingress: Nginx (or Traefik - pick ONE, not both)
kubectl apply -f apps/ingress/nginx/helmrepository.yaml
kubectl apply -f apps/ingress/nginx/helmrelease.yaml
# WARNING: do NOT install both Nginx and Traefik (both are Ingress controllers)
# unless you know what you are doing.
#
#   or for Traefik:
#   kubectl apply -f apps/ingress/traefik/helmrepository.yaml
#   kubectl apply -f apps/ingress/traefik/helmrelease.yaml

# TLS: Cert-Manager
kubectl apply -f apps/cert-manager/helmrepository.yaml
kubectl apply -f apps/cert-manager/helmrelease.yaml

# 8. Install storage (choose ONE option in the storage/ folder)
ansible-playbook -i hosts.ini storage/local-path/install-local-path-provisioner.yml --ask-become-pass
# or
ansible-playbook -i hosts.ini storage/longhorn/install-longhorn-deps.yml --ask-become-pass
ansible-playbook -i hosts.ini storage/longhorn/install-longhorn.yml --ask-become-pass

# 9. Remove NoSchedule taint from control plane (allows scheduling pods on control plane)
ansible-playbook -i hosts.ini 06-remove-noschedule.yml --ask-become-pass
```

## Playbooks

| Order | Playbook | Description |
|-------|----------|-------------|
| 1 | `01-install-deps.yml` | Install CRI-O, kubeadm, kubectl, kubelet on all nodes |
| 2 | `02-setup-control-plane.yml` | Initialize first control plane node |
| 3 | `03-join-workers.yml` | Join worker nodes to cluster |
| 3b | `03b-join-control-plane.yml` | Join additional control plane nodes (HA) |
| 6 | `06-remove-noschedule.yml` | Remove NoSchedule taint from control plane |

## CNI Options

Install **one** CNI plugin from the [`cni/`](cni/README.md) folder. They are
mutually exclusive:

| Option | Playbook | Notes |
|--------|----------|-------|
| [Cilium](cni/README.md) | `cni/cilium/install-cilium.yml` | eBPF, kube-proxy replacement, L2 announcements |
| [Flannel](cni/README.md) | `cni/flannel/install-flannel.yml` | Simple VXLAN overlay, uses `10.244.0.0/16` pod CIDR |

## Apps & Flux

Flux's helm-controller ([`flux/`](flux/README.md)) installs apps declaratively
from `HelmRelease` resources — no `helm` commands. Flux itself is installed
with `helm install` (see `flux/install-flux.yml`). CNI ([`cni/`](cni/README.md))
is the exception and is installed first via playbook.

| Group | App | Manifests | Namespace |
|-------|-----|-----------|-----------|
| Ingress | Nginx | `apps/ingress/nginx/` | `ingress-nginx` |
| Ingress | Traefik | `apps/ingress/traefik/` | `traefik` |
| TLS | Cert-Manager | `apps/cert-manager/` | `cert-manager` |

> **Warning**: do NOT run Traefik and Nginx at the same time (both are Ingress
> controllers) unless you know what you are doing.

```bash
# Install Flux once
ansible-playbook -i hosts.ini flux/install-flux.yml --ask-become-pass

# Then apply the apps you want (kubectl from the kube_admin node)
kubectl apply -f apps/ingress/nginx/helmrepository.yaml
kubectl apply -f apps/ingress/nginx/helmrelease.yaml
```

## Storage Options

Storage is optional and lives in the [`storage/`](storage/README.md) folder.
Pick **one** option:

| Option | Playbooks | StorageClass | Notes |
|--------|-----------|--------------|-------|
| [Local Path Provisioner](storage/README.md) | `storage/local-path/install-local-path-provisioner.yml` | `local-path` | Simple, hostPath-based, no deps, not replicated |
| [Longhorn](storage/README.md) | `storage/longhorn/install-longhorn-deps.yml` + `storage/longhorn/install-longhorn.yml` | `longhorn` | Distributed, replicated; requires iSCSI + NFS |

## Load Balancer Options

Give `LoadBalancer`-type services real IPs. Pick **one** from the
[`loadbalancer/`](loadbalancer/README.md) folder:

| Option | Playbooks | Notes |
|--------|-----------|-------|
| [MetalLB](loadbalancer/README.md) | `loadbalancer/metallb/install-metallb.yml` | ARP/L2 (or BGP) announcements; works with any CNI |
| [Cilium LB-IPAM](loadbalancer/README.md) | `loadbalancer/cilium/cilium-ip-pool.yaml` | Cilium's own LB IP pool + L2 announcements (Cilium CNI only) |

## HA Multi-Master Setup

```bash
# After 02-setup-control-plane.yml completes (runs on the [kube_admin] node):
# Joins ALL additional control planes (main minus the primary) in one run
ansible-playbook -i hosts.ini 03b-join-control-plane.yml --ask-become-pass
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

[main]           # Control plane nodes (all, for HA)
kube1.kube.internal
kube2.kube.internal
kube3.kube.internal

[kube_admin]     # The ONE operator node (kubeadm init + all kubectl/helm runs)
kube1.kube.internal

[workers]        # Worker nodes (optional group)
kube2.kube.internal
kube3.kube.internal

[kube:vars]
ansible_user=root
ansible_ssh_private_key_file=~/.ssh/id_ed25519
```

`[kube_admin]` must contain exactly one node: the first control plane that holds
the admin kubeconfig. It is the only node that runs `kubeadm init` and all
`kubectl`/`helm` installs (playbooks 02, 06, `cni/`, `storage/`, `loadbalancer/`,
`flux/`). Even in a multi-master HA setup, those never run on every control
plane. `[main]` lists all control planes and is used for the HA join (`03b`).

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
