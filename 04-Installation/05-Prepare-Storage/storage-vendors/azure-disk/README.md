# Azure Disk CSI Driver

**Vendor:** Microsoft Azure  
**CSI Driver:** Azure Disk CSI (built-in with OpenShift on Azure)  
**Operator:** Cluster CSI Driver Operator (managed by OpenShift)

## Overview

The Azure Disk CSI driver is automatically installed and managed by OpenShift when the platform is Azure.

## Supported Disk Types

| Disk Type | Use Case | Max IOPS | Max Throughput |
|-----------|----------|----------|----------------|
| Premium SSD v2 | High-performance databases | 80,000 | 1,200 MB/s |
| Premium SSD (P-series) | Production workloads | 20,000 | 900 MB/s |
| Standard SSD (E-series) | Dev/test, light workloads | 6,000 | 750 MB/s |
| Standard HDD (S-series) | Backup, archival | 2,000 | 500 MB/s |
| Ultra Disk | Extreme IOPS requirements | 160,000 | 4,000 MB/s |

## Access Modes

- RWO only (Azure Disks are single-attach)
- For RWX, use Azure File (see `azure-file/`)

## Default StorageClass

OpenShift creates `managed-csi` as the default StorageClass on Azure:

```bash
oc get sc managed-csi
```

## Custom StorageClass

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: azure-premium-v2
provisioner: disk.csi.azure.com
parameters:
  skuName: PremiumV2_LRS
  DiskIOPSReadWrite: "5000"
  DiskMBpsReadWrite: "200"
reclaimPolicy: Delete
allowVolumeExpansion: true
volumeBindingMode: WaitForFirstConsumer
```

## Prerequisites

- OpenShift installed on Azure
- Service principal or managed identity with Disk permissions
- Sufficient managed disk quota in the target region
