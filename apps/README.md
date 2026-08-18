# Apps

One folder per app, grouped by category: `apps/<group>/<app>/`. Apps are
installed declaratively through Flux's helm-controller — no `helm` commands.
Install [Flux](../flux/README.md) first, then apply each app's `HelmRepository`
and `HelmRelease`.

| Group | App | Manifests | Namespace |
|-------|-----|-----------|-----------|
| [Ingress](ingress/README.md) | Nginx | `apps/ingress/nginx/` | `ingress-nginx` |
| [Ingress](ingress/README.md) | Traefik | `apps/ingress/traefik/` | `traefik` |
| TLS | Cert-Manager | `apps/cert-manager/` | `cert-manager` |

> **Warning**: do NOT run Traefik and Nginx at the same time — they are both
> Ingress controllers and will fight over the same `Ingress` objects (and the
> 80/443 ports on your LoadBalancer). Pick one, unless you know what you are
> doing. See [`apps/ingress/README.md`](ingress/README.md).

## 1. Install Flux (once)

```bash
ansible-playbook -i hosts.ini flux/install-flux.yml --ask-become-pass
```

## 2. Install the apps you want

```bash
# Ingress: Nginx
kubectl apply -f apps/ingress/nginx/helmrepository.yaml
kubectl apply -f apps/ingress/nginx/helmrelease.yaml

# Ingress: Traefik
kubectl apply -f apps/ingress/traefik/helmrepository.yaml
kubectl apply -f apps/ingress/traefik/helmrelease.yaml

# TLS: Cert-Manager
kubectl apply -f apps/cert-manager/helmrepository.yaml
kubectl apply -f apps/cert-manager/helmrelease.yaml
```

Flux reconciles each `HelmRelease` and installs the chart. Check status:

```bash
kubectl get helmreleases -A
```
