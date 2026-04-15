# Local Storage For MicroK8s

This folder contains a simple custom `StorageClass` for MicroK8s hostpath storage.

It is useful when you want MicroK8s volumes to be created in a specific local directory instead of the default path.

## Files

- `hostpath-sc.yaml`: custom `StorageClass` named `hostpath-sc`
- `hostpath-sc.yaml.template`: template version to adapt before use
- `testing.yaml.testing`: sample PVC and Pod to validate the storage class

## What It Does

The storage class uses the MicroK8s hostpath provisioner and stores volumes under:

`/data/md0/microk8s-storage`

You can change this path in the YAML file before applying it.

## Prerequisites

- MicroK8s is installed and running
- the `hostpath-storage` addon is enabled
- the target directory exists on the node and has the correct permissions

## Quick Start

Enable the addon:

```bash
microk8s enable hostpath-storage
```

Create the local directory:

```bash
mkdir -p /data/md0/microk8s-storage
chmod 775 /data/md0/microk8s-storage
```

Apply the storage class:

```bash
microk8s kubectl apply -f hostpath-sc.yaml
```

If you want to make it the default storage class:

```bash
microk8s kubectl patch storageclass microk8s-hostpath \
  -p '{"metadata":{"annotations":{"storageclass.kubernetes.io/is-default-class":"false"}}}'

microk8s kubectl patch storageclass hostpath-sc \
  -p '{"metadata":{"annotations":{"storageclass.kubernetes.io/is-default-class":"true"}}}'
```

## Test

Run the sample workload:

```bash
mv testing.yaml.testing testing.yaml
microk8s kubectl apply -f testing.yaml.testing
microk8s kubectl get pods -w
```

Check the created persistent volume:

```bash
microk8s kubectl get pvc test-pvc
microk8s kubectl get pv
```

Remove the test resources:

```bash
microk8s kubectl delete -f testing.yaml.testing
```

## Notes

- This setup is intended for local or single-node style storage needs.
- For multi-node production storage, use a storage backend designed for distributed persistence.
