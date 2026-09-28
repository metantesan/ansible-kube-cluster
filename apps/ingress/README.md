# Ingress

Ingress controllers available in this cluster, managed via Flux. Both are
installed into their own namespace with a `LoadBalancer` service (IP provided
by your chosen load balancer option in `../../loadbalancer/`).

| App | Manifests | Namespace | Notes |
|-----|-----------|-----------|-------|
| [Nginx](nginx/) | `apps/ingress/nginx/` | `ingress-nginx` | NGINX-based ingress controller |
| [Traefik](traefik/) | `apps/ingress/traefik/` | `traefik` | Go-based ingress controller |

> **Warning**: install only one controller for a given public 80/443 address.
> Nginx and Traefik each expose traffic through a `LoadBalancer` service.

## Install one ingress controller

```bash
# Nginx
kubectl apply -f apps/ingress/nginx/helmrepository.yaml
kubectl apply -f apps/ingress/nginx/helmrelease.yaml

# or Traefik
kubectl apply -f apps/ingress/traefik/helmrepository.yaml
kubectl apply -f apps/ingress/traefik/helmrelease.yaml
```
