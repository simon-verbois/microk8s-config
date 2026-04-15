# MicroK8s Config

This repository contains practical configuration examples for a self-hosted MicroK8s cluster.

The goal is simple:

- keep useful MicroK8s customizations in small, reusable folders
- version the YAML files used to operate the cluster
- make it easier to reproduce the same setup on another machine

Each subdirectory focuses on one topic and can be used on its own.

## Repository Concept

This is not a full platform deployment.

It is a collection of focused building blocks for common MicroK8s needs:

- custom local storage
- NVIDIA GPU sharing with time slicing
- Traefik ingress with security-related middleware

The repository is designed as a toolbox:

- each folder contains manifests and values files for one feature
- templates can be copied and adapted to your environment
- the README in each folder explains the expected usage

## Repository Layout

- `local-storage`: custom hostpath storage class for local persistent volumes
- `nvidia`: NVIDIA GPU operator configuration with time slicing
- `traefik`: Traefik ingress setup with CrowdSec, TLS options, and middleware examples

## How To Use This Repository

1. Pick the subdirectory that matches the feature you want to configure.
2. Read its local `README.md`.
3. Review the YAML files and replace the example values with your own settings.
4. Apply the manifests or Helm values to your MicroK8s cluster.

You do not need to use every folder. Each module is independent.

## Notes

- These examples are intended to be adjusted before production use.
- Paths, email addresses, IP ranges, domains, and credentials in the sample files must be reviewed.
- Some modules depend on MicroK8s addons such as `hostpath-storage`, `helm3`, `metallb`, or `nvidia`.

## License

This repository is distributed under the license included in [LICENSE](/data/md0/microk8s-config/LICENSE).
