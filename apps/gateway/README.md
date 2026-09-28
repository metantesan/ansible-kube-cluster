# Gateway API

Gateway API implementations are managed through Flux. The Envoy Gateway
release installs the Gateway API CRDs, the Envoy Gateway controller, and its
managed `LoadBalancer` proxy services.

## Envoy Gateway

```bash
kubectl apply -f apps/gateway/envoy-gateway/ocirepository.yaml
kubectl apply -f apps/gateway/envoy-gateway/helmrelease.yaml
kubectl apply -f apps/gateway/envoy-gateway/gatewayclass.yaml
```

The `eg` `GatewayClass` is used by application `Gateway` resources. cert-manager
is configured to watch Gateway API resources. To issue TLS automatically,
annotate a Gateway with an existing `ClusterIssuer` and reference a Secret from
an HTTPS listener:

```yaml
apiVersion: gateway.networking.k8s.io/v1
kind: Gateway
metadata:
  name: public
  namespace: default
  annotations:
    cert-manager.io/cluster-issuer: letsencrypt-prod
spec:
  gatewayClassName: eg
  listeners:
    - name: https
      hostname: app.example.com
      port: 443
      protocol: HTTPS
      tls:
        mode: Terminate
        certificateRefs:
          - name: app-example-com-tls
      allowedRoutes:
        namespaces:
          from: All
```

Replace the example hostname and `ClusterIssuer` name with values from your
cluster. The issuer must already exist.
