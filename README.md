# k8s-homelab

This repository holds configuration for ArgoCD to administer a Kubernetes homelab.

## Quickstart

TODO: ArgoCD setup

## Ingress

Gateway API is used for this repository.

Key dependencies are:
- Traefik
- cert-manager
- metallb

## Storage

The default storage class for this cluster is `nfs-subdir-external-provisioner`, which allows the use of NFS for provisioning PVCs.

## OIDC/SSO

AUthentik is used for OIDC/SSO.

## Services

Currently supported services include:
- ray
- valkey
- flink
- openwebui
