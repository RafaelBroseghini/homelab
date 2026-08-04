# homepage

[gethomepage](https://gethomepage.dev/) dashboard — homelab landing page with service widgets, k8s resource stats, and links.

## Access

- URL: `https://homepage.rafaelbroseghini.com`
- Chart: `homepage` v2.0.1 from `jameswynn.github.io/helm-charts`
- Image pinned to `v0.10.9` in `values.yaml`

## ArgoCD sync policy

Automated **prune and selfHeal are disabled** in [`../../apps/homepage.yaml`](../../apps/homepage.yaml). Sync homepage manually after changes to avoid unexpected rollbacks of live-tuned config.

## Secrets (not in git)

| Secret | Namespace | Purpose |
|--------|-----------|---------|
| `argocd-user` | homepage | ArgoCD API key for the widget (`HOMEPAGE_VAR_ARGOCD_KEY`) |

Create an API token for the `homepage` account in ArgoCD, then:

```bash
kubectl create secret generic argocd-user -n homepage --from-literal=key=<token>
```

A template may exist in `ignore/secret.yaml` — that directory is gitignored.

## Feed proxy

`feed/` exposes a Traefik route at `feed.rafaelbroseghini.com` → ExternalName service on the Pi (port 8000). Same pattern as pihole: the LAN backend is not a pod.

## Customization

Most of the dashboard lives in `values.yaml` — services, widgets, layout, and k8s discovery settings. This file is large and intentionally hand-edited; treat it as the source of truth for dashboard content.

RBAC is enabled (`enableRbac: true`) with a dedicated `homepage` service account for in-cluster discovery.

## Local build

```bash
kustomize build --enable-helm k8s/config/homepage
```
