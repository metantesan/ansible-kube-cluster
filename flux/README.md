# Flux (GitOps / helm-controller)

[Flux](https://fluxcd.io) provides a declarative **helm-controller**: instead of
running `helm` commands by hand, you create `HelmRelease` (and `HelmRepository`)
custom resources and Flux installs/upgrades the charts for you.

> Flux itself is installed with `helm install` (no Git bootstrap). This repo does
> not use Flux's Git sync — only its Helm capabilities.

## Install Flux

```bash
ansible-playbook -i hosts.ini flux/install-flux.yml --ask-become-pass
```

This runs on the `kube_admin` node and does `helm upgrade --install flux
oci://ghcr.io/fluxcd-community/charts/flux2 --namespace flux-system
--create-namespace`, deploying Flux's controllers (including helm-controller)
into the `flux-system` namespace.

## What to install via Flux vs playbooks

| Component | How it's installed | Why |
|-----------|-------------------|-----|
| CNI (`cni/`) | Ansible playbooks (helm CLI) | Must be up first, before Flux can schedule pods |
| Everything else (apps, etc.) | Flux `HelmRelease` | Managed declaratively by Flux's helm-controller |

## Install apps through Flux

Each app in the [`apps/`](../apps/README.md) folder ships a `HelmRepository`
and a `HelmRelease`. After Flux is installed:

```bash
kubectl apply -f apps/cert-manager/helmrepository.yaml
kubectl apply -f apps/cert-manager/helmrelease.yaml
kubectl apply -f apps/ingress/nginx/helmrepository.yaml
kubectl apply -f apps/ingress/nginx/helmrelease.yaml
```

Flux's helm-controller reconciles the `HelmRelease` and installs the chart into
the app's namespace. Check status with:

```bash
kubectl get helmreleases -A
```

## Managing other charts with Flux

Any Helm chart can be managed the same way. Create a `HelmRepository` for its
repo, then a `HelmRelease` referencing it, e.g. for Longhorn or MetalLB:

```yaml
apiVersion: source.toolkit.fluxcd.io/v1
kind: HelmRepository
metadata:
  name: longhorn
  namespace: flux-system
spec:
  interval: 24h
  url: https://charts.longhorn.io
---
apiVersion: helm.toolkit.fluxcd.io/v2
kind: HelmRelease
metadata:
  name: longhorn
  namespace: flux-system
spec:
  interval: 30m
  chart:
    spec:
      chart: longhorn
      version: 1.11.0
      sourceRef:
        kind: HelmRepository
        name: longhorn
  targetNamespace: longhorn-system
  createNamespace: true
```

Docs: https://fluxcd.io/flux/components/helm/helmreleases/
