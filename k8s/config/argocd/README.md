# argocd

ArgoCD deployment via kustomize.

- Ingress served via Traefik at `/argocd`

## Upgrading

```bash
git clone git@github.com:argoproj/argo-cd.git
git clone git@github.com:rafaelbroseghini/homelab.git
cd homelab/k8s/config/argocd && git checkout <version>
cp -r manifests/base/ ../homelab/k8s/config/argocd/base/
```
