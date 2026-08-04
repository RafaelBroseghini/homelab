# monitoring

Prometheus + Grafana via [kube-prometheus-stack](https://github.com/prometheus-community/helm-charts/tree/main/charts/kube-prometheus-stack).

## Access

- Grafana: `https://grafana.rafaelbroseghini.com`
- Chart: `kube-prometheus-stack` v80.9.2

## Notable settings

- **Alertmanager disabled** (`alertmanager.enabled: false` in `values.yaml`)
- **Grafana secret drift ignored** — `monitoring.yaml` app tells ArgoCD to ignore `/data` on `monitoring-grafana` so admin password changes in-cluster are not reverted

## ArgoCD + CRDs

Large Prometheus Operator CRDs can cause ArgoCD sync issues. `kustomization.yaml` patches several CRDs with `argocd.argoproj.io/sync-options: Replace=true` to allow clean upgrades.

If a monitoring sync fails after a chart bump, check ArgoCD for CRD replace conflicts before debugging Prometheus itself.

## ServiceMonitors

`monitors/argocd.yaml` scrapes ArgoCD component metrics from the `argocd` namespace. All monitors carry `release: monitoring` so the Prometheus operator picks them up.

To monitor a new workload, add a ServiceMonitor under `monitors/` and include it in `monitors/kustomization.yaml`.

## Upgrading

1. Bump `version` in `kustomization.yaml`
2. Review [chart changelog](https://github.com/prometheus-community/helm-charts/tree/main/charts/kube-prometheus-stack) — CRD and values changes are frequent
3. Run `kustomize build --enable-helm k8s/config/monitoring` locally and inspect diff
4. Sync via ArgoCD; watch for CRD replace operations

## Local build

```bash
kustomize build --enable-helm k8s/config/monitoring
```

The `charts/` directory is gitignored and generated on build.
