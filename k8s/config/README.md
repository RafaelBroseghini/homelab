# config

Kustomize (and Helm-in-Kustomize) manifests for each homelab component. All paths here are referenced by ArgoCD Applications in [`../apps/`](../apps/).

## Components

| Directory | Description | README |
|-----------|-------------|--------|
| `argocd/` | ArgoCD install (vendored upstream manifests + customizations) | [README](./argocd/README.md) |
| `cert-manager/` | TLS issuance (Let's Encrypt via DNS-01) | [README](./cert-manager/README.md) |
| `reflector/` | Mirrors TLS secrets across namespaces | [README](./reflector/README.md) |
| `pihole/` | Traefik ingress to bare-metal Pi-hole | [README](./pihole/README.md) |
| `homepage/` | Dashboard (gethomepage) + feed proxy | [README](./homepage/README.md) |
| `monitoring/` | kube-prometheus-stack + ServiceMonitors | [README](./monitoring/README.md) |

## Conventions

### `ignore/` directories

Gitignored. Holds secrets, one-off resources, and things that should not be committed (ACME issuers with cloud credentials, ExternalName services with LAN IPs, etc.). Check each component's README for what belongs there.

### `charts/` directories

Gitignored. Helm chart tarballs expanded locally by `kustomize build --enable-helm`. Safe to delete and regenerate.

### TLS

Wildcard cert `*.rafaelbroseghini.com` is issued in `cert-manager` and auto-reflected to `argocd`, `pihole`, `homepage`, and `monitoring` via reflector annotations on the Certificate. IngressRoutes reference `rafaelbroseghini-cert`.

### Ingress

All HTTP ingress uses Traefik `IngressRoute` CRDs (`traefik.io/v1alpha1`), not standard Kubernetes Ingress.
