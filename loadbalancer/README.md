# Load Balancer Options

Give your cluster `LoadBalancer`-type services real IPs. Pick a controller for
your setup — **choose one**, they are not meant to run side by side.

| Option | Playbooks | How it works |
|--------|-----------|--------------|
| [MetalLB](#option-1-metallb) | `loadbalancer/metallb/install-metallb.yml` | ARP/L2 (or BGP) announces service IPs; works with any CNI |
| [Cilium LB-IPAM](#option-2-cilium-lb-ipam) | `loadbalancer/cilium/cilium-ip-pool.yaml` | Uses Cilium's own IPAM + L2 announcements (requires Cilium CNI) |

## Option 1: MetalLB

Installs MetalLB via Helm and applies an `IPAddressPool` + `L2Advertisement`.

```bash
# Edit the address range in metallb/metallb-ip-pool.yaml first
ansible-playbook -i hosts.ini loadbalancer/metallb/install-metallb.yml --ask-become-pass
```

- Creates namespace `metallb-system`.
- The playbook applies `loadbalancer/metallb/metallb-ip-pool.yaml` (edit the
  `192.168.1.x` range to match your LAN before running).

## Option 2: Cilium LB-IPAM

If you use the Cilium CNI option, you can give LoadBalancer services real IPs
with Cilium's own LB IP pool + L2 announcements. The Cilium install in
`cni/cilium/install-cilium.yml` already enables `l2announcements`.

```bash
# Edit the CIDR / interfaces in cilium/cilium-ip-pool.yaml first
kubectl apply -f loadbalancer/cilium/cilium-ip-pool.yaml
```

- Creates `CiliumLoadBalancerIPPool` (the IP range) and
  `CiliumL2AnnouncementPolicy` (which interface announces them).

> **Note**: do NOT run MetalLB and Cilium LB-IPAM at the same time.

