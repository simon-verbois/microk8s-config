# MicroK8s — Image Garbage Collection (kubelet)

## Objective

Automatically remove unused container images to prevent disk exhaustion on cluster nodes.

---

## Configuration

Configure kubelet image garbage collection thresholds by updating the kubelet arguments:

```bash
sudo tee -a /var/snap/microk8s/current/args/kubelet >/dev/null <<'EOF'
--image-gc-high-threshold=75
--image-gc-low-threshold=50
EOF
```

### Explanation

* `--image-gc-high-threshold=75`
  Triggers image garbage collection when disk usage reaches 75%.

* `--image-gc-low-threshold=50`
  Removes unused images until disk usage drops back to 50%.

---

## Apply Configuration

Restart MicroK8s to apply the changes:

```bash
sudo snap restart microk8s
```

---

## Verification

Check that the configuration is applied:

```bash
cat /var/snap/microk8s/current/args/kubelet
```

---

## Best Practices

* Apply this configuration on **every node** in the cluster
* Adjust thresholds based on disk capacity and workload characteristics:

  * `75 / 50` → balanced (recommended)
  * `75 / 10` → aggressive (may increase image re-pulls)
