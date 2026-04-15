# NVIDIA GPU Configuration For MicroK8s

This folder contains example files to configure GPU sharing on MicroK8s with NVIDIA time slicing.

It is useful when one physical GPU needs to be shared by several workloads.

## Files

- `time-slicing-config-all.yaml`: ConfigMap used by the NVIDIA device plugin
- `testing.yaml.testing`: test Pod that runs `nvidia-smi`

## Prerequisites

- an NVIDIA GPU is available on the host
- NVIDIA drivers are installed on the host
- NVIDIA Container Toolkit is installed on the host
- MicroK8s is installed and running
- the `nvidia` addon can be enabled successfully

## Quick Start

Enable NVIDIA support in MicroK8s:

```bash
microk8s enable nvidia
```

Watch the GPU operator resources:

```bash
microk8s kubectl get pods -n gpu-operator-resources -w
```

Check that the node exposes a GPU resource:

```bash
microk8s kubectl describe node | grep nvidia.com/gpu
```

Apply the time-slicing configuration:

```bash
microk8s kubectl apply -n gpu-operator-resources -f time-slicing-config-all.yaml
```

Patch the cluster policy to use that config:

```bash
microk8s kubectl patch clusterpolicies.nvidia.com/cluster-policy \
  -n gpu-operator-resources \
  --type merge \
  -p '{"spec":{"devicePlugin":{"config":{"name":"time-slicing-config-all","default":"any"}}}}'
```

## Test

Run the validation pod:

```bash
mv testing.yaml.testing testing.yaml
microk8s kubectl apply -f testing.yaml
microk8s kubectl get pods -w
microk8s kubectl logs -f nvidia-smi-test
```

Delete the test pod after validation:

```bash
microk8s kubectl delete -f testing.yaml
```

## Notes

- The example config sets `replicas: 8` for `nvidia.com/gpu`. Adjust this value to match your GPU sharing policy.
- The test pod requests `2` GPU slices to confirm that time slicing is active.
- Time slicing improves sharing, but it does not provide full isolation between workloads.
