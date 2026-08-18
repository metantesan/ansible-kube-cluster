# Ingress

Ingress controllers available in this cluster, managed via Flux. Both are
installed into their own namespace with a `LoadBalancer` service (IP provided
by your chosen load balancer option in `../../loadbalancer/`).

| App | Manifests | Namespace | Notes |
|-----|-----------|-----------|-------|
| [Nginx](nginx/) | `apps/ingress/nginx/` | `ingress-nginx` | NGINX-based ingress controller |
| [Traefik](traefik/) | `apps/ingress/traefik/` | `traefik` | Go-based ingress controller |

> **Warning**: do NOT run Nginx and Traefik at the same time — they are both
> Ingress controllers and will fight over the same `Ingress` objects and the
> 80/443 ports on your LoadBalancer. Pick one, unless you know what you are
> doing.

## Install one ingress controller

```bash
# Nginx
kubectl apply -f apps/ingress/nginx/helmrepository.yaml
kubectl apply -f apps/ingress/nginx/helmrelease.yaml

# or Traefik
kubectl apply -f apps/ingress/traefik/helmrepository.yaml
kubectl apply -f apps/ingress/traefik/helmrelease.yaml
```
