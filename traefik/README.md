# Traefik Configuration For MicroK8s

This folder contains an example Traefik setup for MicroK8s.

It combines ingress, TLS, and security-related middleware in one place. The sample configuration is built around:

- Traefik deployed with Helm
- Let's Encrypt certificates with the OVH DNS challenge
- CrowdSec integration
- Geo-blocking, IP allow lists, rate limiting, compression, and security headers

## Files

- `traefik-values.yaml`: Helm values for the Traefik deployment
- `crowdsec-values.yaml`: Helm values for the CrowdSec deployment
- `middlewares.yaml`: Traefik middleware examples
- `tls-profile.yaml`: TLS option example
- `*.template`: template files to copy and adapt

## Prerequisites

- MicroK8s is installed and running
- the `helm3` addon is enabled
- MetalLB or another `LoadBalancer` solution is available
- a domain name is managed in OVH if you want to use the included ACME DNS challenge example
- local storage is available for persistent data such as `acme.json` and CrowdSec data

## What This Setup Covers

- automatic HTTPS with a DNS challenge
- persistent Traefik data storage
- middleware examples for hardening and access control
- local plugin loading for CrowdSec and geoblocking

This folder is an example baseline, not a one-click production installer.

## Quick Start

Enable the required addons:

```bash
microk8s enable helm3 metallb
```

Create the namespaces:

```bash
microk8s kubectl create namespace traefik
microk8s kubectl create namespace crowdsec
```

Create the OVH credentials secret used by Traefik:

```bash
microk8s kubectl create secret generic ovh-credentials \
  --namespace=traefik \
  --from-literal=OVH_ENDPOINT=ovh-eu \
  --from-literal=OVH_APPLICATION_KEY=CHANGE_ME \
  --from-literal=OVH_APPLICATION_SECRET=CHANGE_ME \
  --from-literal=OVH_CONSUMER_KEY=CHANGE_ME
```

Install CrowdSec:

```bash
microk8s helm3 repo add crowdsec https://crowdsecurity.github.io/helm-charts
microk8s helm3 repo update
microk8s helm3 install crowdsec crowdsec/crowdsec \
  -n crowdsec \
  -f crowdsec-values.yaml
```

Apply the middleware and TLS resources:

```bash
microk8s kubectl apply -f middlewares.yaml
microk8s kubectl apply -f tls-profile.yaml
```

Install Traefik:

```bash
microk8s helm3 repo add traefik https://traefik.github.io/charts
microk8s helm3 repo update
microk8s helm3 install traefik traefik/traefik \
  -n traefik \
  -f traefik-values.yaml
```

## Important Customization Points

Review these values before deployment:

- email address used for ACME
- OVH credentials
- storage class name
- local plugin paths
- allowed IP ranges
- allowed or blocked countries
- CrowdSec LAPI key
- timezone

## Operations

Upgrade Traefik after editing the values file:

```bash
microk8s helm3 upgrade traefik traefik/traefik \
  -n traefik \
  -f traefik-values.yaml
```

View Traefik logs:

```bash
microk8s kubectl logs -n traefik -f deployment/traefik
```

## Notes

- The current example loads Traefik plugins from local host paths. Make sure those paths exist on the node.
- `middlewares.yaml` contains example values that must be replaced before production use.
- If you do not use OVH, you will need to adapt the ACME DNS resolver configuration.
