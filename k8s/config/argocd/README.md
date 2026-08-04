# argocd

ArgoCD deployment via Kustomize, vendored from upstream manifests with local patches.

## Access

- URL: `https://argocd.rafaelbroseghini.com/argocd`
- Served behind Traefik at subpath `/argocd` (see `base/config/argocd-cmd-params-cm.yaml`)
- `server.insecure: "true"` — TLS terminates at Traefik, not the ArgoCD server pod

## Notable customizations

| File | What it does |
|------|--------------|
| `base/kustomization.yaml` | Pins image to `v3.5.0-rc1`; dex and notifications controllers are commented out |
| `base/config/argocd-cm.yaml` | Enables `--enable-helm` for Kustomize builds; defines `accounts.homepage` API key account |
| `base/config/argocd-cmd-params-cm.yaml` | Root path `/argocd`, insecure server mode |
| `base/ingress/traefik.yaml` | gRPC route (priority 11) + HTTP route for the UI |

## Homepage integration

`argocd-cm` defines `accounts.homepage` with `apiKey` login. Create the corresponding API token in the ArgoCD UI and store it in the `argocd-user` secret in the `homepage` namespace (see homepage README). RBAC for that account is configured in `argocd-rbac-cm`.

## Upgrading

1. Clone upstream at the target version:

```bash
git clone https://github.com/argoproj/argo-cd.git /tmp/argo-cd
cd /tmp/argo-cd && git checkout <version>
```

2. Copy manifests and re-apply local customizations:

```bash
cp -r /tmp/argo-cd/manifests/base/* /path/to/homelab/k8s/config/argocd/base/
```

3. Re-apply your patches: image tag in `kustomization.yaml`, `argocd-cm.yaml`, `argocd-cmd-params-cm.yaml`, ingress, and any commented-out components (dex, notification).

4. Diff before committing — upstream layout changes between major versions.

## Local build

```bash
kustomize build k8s/config/argocd/base
```
