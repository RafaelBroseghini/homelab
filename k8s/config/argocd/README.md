# argocd

ArgoCD deployment via Kustomize, vendored from upstream manifests with local patches.

## Access

- URL: `https://argocd.rafaelbroseghini.com` (root path on dedicated host)
- `argocd-cm` `url` must match the public URL for redirects and links
- `server.insecure: "true"` — TLS terminates at Traefik, not the ArgoCD server pod
- Traefik priorities in `base/ingress/traefik.yaml` must stay above the dashboard `PathPrefix(/api)` rule: HTTP `100`, gRPC `110` (gRPC stays higher). Helm’s default Traefik dashboard IngressRoute often matches ``PathPrefix(`/dashboard`) || PathPrefix(`/api`)`` with no Host, so its effective priority is the rule length (~48). Lower priorities let that rule steal `/api/v1/...`: the shell loads and the UI hangs. The gRPC match is ``HeaderRegexp(`Content-Type`, `^application/grpc`)`` so grpc-web variants still hit the h2c service.
- The Traefik dashboard itself is still not GitOps’d. Proper fix later: scope that route with ``Host(`…`) && (PathPrefix(`/dashboard`) || PathPrefix(`/api`))`` instead of a bare path match.

## Notable customizations

| File | What it does |
|------|--------------|
| `base/kustomization.yaml` | Pins image to `v3.6.0-rc1`; dex and notifications controllers are commented out |
| `base/config/argocd-cm.yaml` | Enables `--enable-helm` for Kustomize builds; defines `accounts.homepage` API key account |
| `base/config/argocd-cmd-params-cm.yaml` | Insecure server mode (TLS at Traefik) |
| `base/ingress/traefik.yaml` | Host-based Traefik v3 routes (HTTP priority 100, gRPC/h2c priority 110) |

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
