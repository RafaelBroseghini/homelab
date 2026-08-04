# k8s

GitOps manifests for the homelab cluster. ArgoCD is the source of truth once bootstrapped.

## Layout

| Path | Purpose |
|------|---------|
| [`root.yaml`](./root.yaml) | Bootstrap Application — apply this once to start the app-of-apps loop |
| [`apps/`](./apps/) | ArgoCD Application CRs (one per component) |
| [`config/`](./config/) | Kustomize/Helm source for each component |

## Bootstrap

After k3s and ArgoCD are running on the cluster:

```bash
kubectl apply -f k8s/root.yaml
```

`root` syncs `k8s/apps/`, which in turn deploys every component under `k8s/config/`.

## Local validation

Kustomize with Helm support is required for most configs:

```bash
kustomize build --enable-helm k8s/config/<component>
```

## Dependency order

Components have implicit ordering — if something fails on first sync, wait for upstream deps:

1. **reflector** — must exist before TLS certs can be mirrored
2. **cert-manager** — issues the wildcard cert that reflector copies
3. Everything else (argocd self-manages, pihole/homepage/monitoring depend on cert + ingress)

See [`config/README.md`](./config/README.md) for per-component notes.
