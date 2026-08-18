# CNI Options

Install **exactly one** CNI plugin. They are mutually exclusive — do not run
both on the same cluster.

| Option | Playbook | Notes |
|--------|----------|-------|
| [Cilium](#option-1-cilium) | `cni/cilium/install-cilium.yml` | eBPF-based; replaces kube-proxy; L2 announcements for bare-metal LoadBalancer |
| [Flannel](#option-2-flannel) | `cni/flannel/install-flannel.yml` | Simple VXLAN overlay; works with the default kube-proxy |

## Option 1: Cilium

```bash
ansible-playbook -i hosts.ini cni/cilium/install-cilium.yml --ask-become-pass
```

Deploys Cilium to `kube-system` with `kubeProxyReplacement=true` and
`l2announcements.enabled=true`.

## Option 2: Flannel

```bash
ansible-playbook -i hosts.ini cni/flannel/install-flannel.yml --ask-become-pass
```

Deploys Flannel to `kube-flannel` using the `10.244.0.0/16` pod CIDR, matching
the kubeadm `podSubnet` from `02-setup-control-plane.yml`. If you change the pod
CIDR there, set `pod_cidr` in the playbook accordingly.

> **Note**: Do NOT run both. Cilium with `kubeProxyReplacement=true` and Flannel
> cannot coexist in the same cluster.
