# cert-manager

Issues a wildcard TLS certificate and distributes it to ingress namespaces via [reflector](../reflector/README.md).

## What is deployed

- **Helm chart**: `cert-manager` v1.14.5 from `charts.jetstack.io`
- **Certificate**: `rafaelbroseghini` — `*.rafaelbroseghini.com`, secret `rafaelbroseghini-cert`
- Reflector annotations on the Certificate auto-mirror the secret to `argocd`, `pihole`, `homepage`, and `monitoring`

## First-time setup

### CRDs

Helm does not always install CRDs cleanly via ArgoCD on first sync. If cert-manager resources fail with "CRD not found", apply CRDs manually once:

```bash
kubectl apply -f https://github.com/cert-manager/cert-manager/releases/download/v1.14.5/cert-manager.crds.yaml
```

(Also noted as a comment in `kustomization.yaml`.)

### Issuer and cloud credentials

The ACME `ClusterIssuer`/`Issuer` and Azure DNS solver credentials live in **`ignore/issuer.yaml`** (gitignored). You also need the `azuredns-config` secret in the `cert-manager` namespace with the DNS provider client secret.

Do not commit issuer credentials — keep them in `ignore/` or a secrets manager.

## Upgrading

1. Bump `chartVersion` in `kustomization.yaml`
2. Check upstream release notes for CRD changes — you may need to re-apply CRDs
3. Verify the Certificate renews: `kubectl describe certificate -n cert-manager rafaelbroseghini`

## Local build

```bash
kustomize build --enable-helm k8s/config/cert-manager
```
