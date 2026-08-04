# reflector

[Emberstack Reflector](https://github.com/emberstack/kubernetes-reflector) — copies secrets (and configmaps) across namespaces based on annotations.

## Why it exists

cert-manager issues the wildcard TLS cert once in `cert-manager`. Reflector mirrors `rafaelbroseghini-cert` into every namespace that serves HTTPS ingress, so each IngressRoute can reference the same `secretName` locally.

Allowed target namespaces are declared on the Certificate in [`../cert-manager/certificate.yaml`](../cert-manager/certificate.yaml):

- `argocd`
- `pihole`
- `homepage`
- `monitoring`

## Adding a new namespace

1. Add the namespace to `reflection-allowed-namespaces` on the Certificate
2. Re-sync cert-manager
3. Reflector will auto-create `rafaelbroseghini-cert` in the new namespace

## Chart

- `reflector` v9.0.313 from `emberstack.github.io/helm-charts`

## Local build

```bash
kustomize build --enable-helm k8s/config/reflector
```
