# apps

ArgoCD Application manifests — the app-of-apps layer synced by [`root.yaml`](../root.yaml).

Each file points at a path under `k8s/config/` and defines sync policy for that component.

## Applications

| App | Namespace | Sync policy | Notes |
|-----|-----------|-------------|-------|
| `argocd.yaml` | argocd | auto prune + selfHeal | Self-manages ArgoCD from `config/argocd/base` |
| `cert-manager.yaml` | cert-manager | auto, `CreateNamespace` | CRDs may need manual install on first deploy (see cert-manager README) |
| `reflector.yaml` | reflector | auto, `CreateNamespace` | Deploy before workloads that need mirrored TLS secrets |
| `pihole.yaml` | pihole | auto, `CreateNamespace` | Ingress only — Pi-hole runs on bare metal |
| `homepage.yaml` | homepage | **no** auto prune/selfHeal | Manual sync preferred; avoids wiping live config |
| `monitoring.yaml` | monitoring | auto, `CreateNamespace` | Ignores Grafana secret data drift |

## Adding a new app

1. Create manifests under `k8s/config/<name>/`
2. Add `<name>.yaml` here following the existing pattern
3. Commit and let `root` pick it up (or `kubectl apply -f k8s/apps/<name>.yaml`)

## Homepage exception

`homepage.yaml` disables automated prune and selfHeal. Homepage holds a lot of live-tuned `values.yaml` content and references secrets (`argocd-user`) that are not in git. Sync it deliberately after changes.
