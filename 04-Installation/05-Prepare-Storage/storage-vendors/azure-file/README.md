# Azure File CSI Driver

**Vendor:** Microsoft Azure  
**CSI Driver:** Azure File CSI (built-in with OpenShift on Azure)  
**Operator:** Cluster CSI Driver Operator (managed by OpenShift)

## Overview

Azure File provides managed SMB and NFS file shares with RWX support.

## Access Modes

- RWO, RWX, ROX

## Supported Tiers

| Tier | Protocol | Use Case |
|------|----------|----------|
| Premium (SSD) | SMB, NFS | Production RWX workloads |
| Transaction optimized | SMB | General purpose |
| Hot | SMB | Frequently accessed data |
| Cool | SMB | Archival |

## Default StorageClass

OpenShift creates `azurefile-csi` on Azure:

```bash
oc get sc azurefile-csi
```

## Custom StorageClass (NFS)

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: azure-file-nfs
provisioner: file.csi.azure.com
parameters:
  protocol: nfs
  skuName: Premium_LRS
reclaimPolicy: Delete
allowVolumeExpansion: true
volumeBindingMode: Immediate
mountOptions:
  - nconnect=4
```

## Prerequisites

- OpenShift installed on Azure
- Storage account created (or dynamic creation via CSI)
- For NFS: Premium tier required, VNet integration configured
- Network rules allow access from cluster nodes to the storage account
