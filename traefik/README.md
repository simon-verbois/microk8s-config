# Traefik Configuration for MicroK8s

Traefik setup for MicroK8s with:
- Helm-based deployment
- OVH DNS challenge for Let's Encrypt
- CrowdSec bouncer and appsec
- `geoblock`, `whitelist`, and `public-secure-chain` middlewares

## Files

- `traefik-values.yaml`: Traefik Helm values
- `crowdsec-values.yaml`: CrowdSec Helm values
- `middlewares.yaml`: shared security middlewares
- `tls-profile.yaml`: TLS options
- `*.template`: reusable sanitized templates

## Quick Start

```bash
microk8s enable helm3 metallb
microk8s kubectl create namespace traefik
microk8s kubectl create namespace crowdsec
```

Create the OVH secret:

```bash
microk8s kubectl create secret generic ovh-credentials \
  -n traefik \
  --from-literal=OVH_ENDPOINT=ovh-eu \
  --from-literal=OVH_APPLICATION_KEY=CHANGE_ME \
  --from-literal=OVH_APPLICATION_SECRET=CHANGE_ME \
  --from-literal=OVH_CONSUMER_KEY=CHANGE_ME
```

Install or upgrade CrowdSec:

```bash
microk8s helm3 repo add crowdsec https://crowdsecurity.github.io/helm-charts
microk8s helm3 repo update
microk8s helm3 upgrade --install crowdsec crowdsec/crowdsec \
  -n crowdsec \
  -f crowdsec-values.yaml
```

Apply shared Traefik resources:

```bash
microk8s kubectl apply -f middlewares.yaml
microk8s kubectl apply -f tls-profile.yaml
```

Install or upgrade Traefik:

```bash
microk8s helm3 repo add traefik https://traefik.github.io/charts
microk8s helm3 repo update
microk8s helm3 upgrade --install traefik traefik/traefik \
  -n traefik \
  -f traefik-values.yaml \
  --version 39.0.8
```

## Notes

- Review OVH credentials, ACME email, storage class, allowed IPs, and allowed countries before production use.
- `public-secure-chain` is the default middleware chain for public routes.
- Templates should stay aligned with the local `.yaml` files when the live config changes.
