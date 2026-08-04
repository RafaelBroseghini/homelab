# pihole

Traefik ingress for Pi-hole running on **bare metal** (Raspberry Pi), not as a Kubernetes workload.

## Architecture

```
Browser → Traefik (k3s) → ExternalName Service → Pi-hole (LAN host)
```

Only the IngressRoute in this repo is synced by ArgoCD. The backend `Service` and redirect `Middleware` objects live in **`ignore/svc.yaml`** because they contain the LAN IP and non-standard ports.

## Ports

Pi-hole admin UI uses non-default ports on the Pi:

| Traefik middleware | Redirects to |
|--------------------|--------------|
| `redirectscheme` (HTTP) | `http://<host>:8880` |
| `redirectscheme-https` (HTTPS) | `https://<host>:4443` |

Homepage widgets also point at port `8880` for the HTTP admin interface.

## Setup checklist

1. Ensure `ignore/svc.yaml` is applied (or recreate the ExternalName service pointing at the Pi's LAN IP)
2. Confirm reflector has copied `rafaelbroseghini-cert` into the `pihole` namespace
3. DNS: `pihole.rafaelbroseghini.com` should resolve to the k3s/Traefik host, not the Pi directly

## Local build

```bash
kustomize build k8s/config/pihole
```

Only renders the IngressRoute — the service in `ignore/` must be applied separately.
