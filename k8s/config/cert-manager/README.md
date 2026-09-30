# cert-manager

Issues a wildcard TLS certificate and distributes it to ingress namespaces via [reflector](../reflector/README.md).

## What is deployed

- **Helm chart**: `cert-manager` v1.21.2 from `charts.jetstack.io`
- **CRDs**: v1.21.2 (`cert-manager.crds.yaml` from the matching GitHub release)
- **Certificate**: `rafaelbroseghini` — `*.rafaelbroseghini.com`, secret `rafaelbroseghini-cert`
- Reflector annotations on the Certificate auto-mirror the secret to `argocd`, `pihole`, `homepage`, and `monitoring`

Intentional jump off EOL v1.14.5. The chart still does not install CRDs (`crds.enabled` defaults to false); apply the matching CRD manifest before syncing.

## First-time setup

### CRDs

Helm does not always install CRDs cleanly via ArgoCD on first sync. If cert-manager resources fail with "CRD not found", apply CRDs manually once:

```bash
kubectl apply -f https://github.com/cert-manager/cert-manager/releases/download/v1.21.2/cert-manager.crds.yaml
```

(Also noted as a comment in `kustomization.yaml`.)

### Issuer and cloud credentials

The ACME `ClusterIssuer`/`Issuer` and Azure DNS solver credentials live in **`ignore/issuer.yaml`** (gitignored). You also need the `azuredns-config` secret in the `cert-manager` namespace with the DNS provider client secret.

Do not commit issuer credentials — keep them in `ignore/` or a secrets manager.

## Upgrading

1. Bump `chartVersion` in `kustomization.yaml` and the CRD URL comment to the same release
2. Re-apply CRDs from that release before syncing the chart
3. Verify the Certificate is Ready: `kubectl describe certificate -n cert-manager rafaelbroseghini`
4. Confirm reflector copied `rafaelbroseghini-cert` into `argocd`, `pihole`, `homepage`, and `monitoring`

## Local build

```bash
kustomize build --enable-helm k8s/config/cert-manager
```
