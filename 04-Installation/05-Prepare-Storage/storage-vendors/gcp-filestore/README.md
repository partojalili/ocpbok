# GCP Filestore CSI Driver

**Vendor:** Google Cloud  
**CSI Driver:** GCP Filestore CSI  
**Operator:** Manual deployment or Marketplace

## Overview

GCP Filestore provides managed NFS storage with RWX support.

## Supported Tiers

| Tier | Capacity | Use Case |
|------|----------|----------|
| Basic HDD | 1-63.9 TB | Dev/test, general file sharing |
| Basic SSD | 2.5-63.9 TB | Production file workloads |
| Zonal (High Scale SSD) | 10-100 TB | High-performance RWX |
| Enterprise | 1-10 TB | Mission-critical, regional availability |

## Access Modes

- RWO, RWX, ROX

## StorageClass Example

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: gcp-filestore
provisioner: filestore.csi.storage.gke.io
parameters:
  tier: standard
  network: default
reclaimPolicy: Delete
allowVolumeExpansion: true
volumeBindingMode: Immediate
```

## Prerequisites

- Filestore API enabled on the GCP project
- Service account with Filestore permissions
- VPC network accessible from cluster nodes
- Firewall rules allow NFS (port 2049) from cluster nodes to Filestore
