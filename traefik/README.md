efik Configuration for MicroK8s

This folder provides a clean and modern Traefik setup for MicroK8s using:

* Traefik deployed with Helm (chart v39+)
* Let's Encrypt certificates via OVH DNS challenge
* CrowdSec integration
* Local plugins loaded via Kubernetes PVC (`localPath`)
* Security middlewares (rate limiting, headers, geo filtering, etc.)

This setup avoids deprecated configurations and follows the current Traefik Helm chart best practices.

---

## 📁 Files

* `traefik-values.yaml` → Helm values for Traefik
* `crowdsec-values.yaml` → Helm values for CrowdSec
* `middlewares.yaml` → Security middlewares
* `tls-profile.yaml` → TLS configuration
* `*.template` → editable templates

---

## ⚙️ Prerequisites

* MicroK8s installed and running
* `helm3` addon enabled
* `metallb` (or another LoadBalancer solution)
* A domain managed in OVH (for ACME DNS challenge)
* Local storage (`hostpath-sc` or equivalent)

---

## 🚀 Quick Start

### Enable addons

```bash
microk8s enable helm3 metallb
```

### Create namespaces

```bash
microk8s kubectl create namespace traefik
microk8s kubectl create namespace crowdsec
```

---

## 🔐 OVH Credentials

```bash
microk8s kubectl create secret generic ovh-credentials \
  -n traefik \
  --from-literal=OVH_ENDPOINT=ovh-eu \
  --from-literal=OVH_APPLICATION_KEY=CHANGE_ME \
  --from-literal=OVH_APPLICATION_SECRET=CHANGE_ME \
  --from-literal=OVH_CONSUMER_KEY=CHANGE_ME
```

---

## 🛡️ Install CrowdSec

```bash
microk8s helm3 repo add crowdsec https://crowdsecurity.github.io/helm-charts
microk8s helm3 repo update

microk8s helm3 install crowdsec crowdsec/crowdsec \
  -n crowdsec \
  -f crowdsec-values.yaml
```

---

## 🔌 Local Plugins (Modern Setup)

Plugins are loaded via a **PVC-backed volume** using the `localPath` mechanism.

### Step 1 — Create PVC

```bash
microk8s kubectl apply -n traefik -f - <<EOF
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: traefik-plugins
spec:
  accessModes:
    - ReadWriteOnce
  storageClassName: hostpath-sc
  resources:
    requests:
      storage: 500Mi
EOF
```

---

### Step 2 — Populate Plugins

```bash
microk8s kubectl run plugin-loader \
  --namespace=traefik \
  --image=busybox:latest \
  --restart=Never \
  --overrides='{
    "spec": {
      "volumes": [{
        "name": "plugins",
        "persistentVolumeClaim": {"claimName": "traefik-plugins"}
      }],
      "containers": [{
        "name": "loader",
        "image": "busybox:latest",
        "command": ["sh", "-c", "sleep 3600"],
        "volumeMounts": [{
          "name": "plugins",
          "mountPath": "/plugins"
        }]
      }]
    }
  }'

microk8s kubectl wait pod/plugin-loader -n traefik --for=condition=Ready

microk8s kubectl cp /data/md0/microk8s-config/traefik/src/. \
  traefik/plugin-loader:/plugins/

microk8s kubectl delete pod plugin-loader -n traefik
```

---

## 🔐 Apply Security Resources

```bash
microk8s kubectl apply -f middlewares.yaml
microk8s kubectl apply -f tls-profile.yaml
```

---

## 🌐 Install Traefik

```bash
microk8s helm3 repo add traefik https://traefik.github.io/charts
microk8s helm3 repo update

microk8s helm3 install traefik traefik/traefik \
  -n traefik \
  -f traefik-values.yaml \
  --version 39.0.8
```

---

## 🔄 Upgrade Traefik

```bash
microk8s helm3 upgrade traefik traefik/traefik \
  -n traefik \
  -f traefik-values.yaml \
  --version 39.0.8
```

---

## 📊 Operations

### Check pods

```bash
kubectl -n traefik get pods
```

### Logs

```bash
kubectl -n traefik logs -f deployment/traefik
```

### Check plugins loaded

```bash
kubectl -n traefik logs deployment/traefik | grep plugin
```

---

## ⚠️ Important Notes

* Plugins are loaded from a PVC → **cluster portable**
* No dependency on node filesystem (no `hostPath`)
* Compatible with future Traefik chart versions
* `inlinePlugin` is NOT used (size limits, not suitable for geoblock DB)

---

## 🔧 Customization

Review before production:

* ACME email
* OVH credentials
* storageClass
* plugin sources
* allowed IPs / countries
* CrowdSec API key
* TLS policies

---

## 🧠 Architecture Summary

```
PVC (traefik-plugins)
        ↓
Mounted into Traefik (/plugins-local)
        ↓
Loaded via localPlugins (type: localPath)
        ↓
Used by middlewares (crowdsec, geoblock)
```

