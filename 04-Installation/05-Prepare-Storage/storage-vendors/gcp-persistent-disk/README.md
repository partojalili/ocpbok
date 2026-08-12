# GCP Persistent Disk CSI Driver

**Vendor:** Google Cloud  
**CSI Driver:** GCE PD CSI (built-in with OpenShift on GCP)  
**Operator:** Cluster CSI Driver Operator (managed by OpenShift)

## Overview

The GCE PD CSI driver is automatically installed and managed by OpenShift when the platform is GCP.

## Supported Disk Types

| Disk Type | Use Case | Max IOPS | Max Throughput |
|-----------|----------|----------|----------------|
| pd-ssd (default) | General purpose SSD | 100,000 | 1,200 MB/s |
| pd-balanced | Balanced price/performance | 80,000 | 1,200 MB/s |
| pd-standard | HDD, cold storage | 7,500 | 400 MB/s |
| pd-extreme | Highest performance | 120,000 | 2,400 MB/s |
| hyperdisk-extreme | Next-gen extreme | 350,000 | 5,000 MB/s |

## Access Modes

- RWO (single-zone)
- ROX (multi-attach read-only)
- For RWX, use GCP Filestore (see `gcp-filestore/`)

## Default StorageClass

```bash
oc get sc standard-csi
```

## Custom StorageClass

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: gcp-ssd
provisioner: pd.csi.storage.gke.io
parameters:
  type: pd-ssd
reclaimPolicy: Delete
allowVolumeExpansion: true
volumeBindingMode: WaitForFirstConsumer
```

## Prerequisites

- OpenShift installed on GCP
- Service account with Compute and Storage permissions
- Sufficient persistent disk quota in the target region
