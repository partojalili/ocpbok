# VMware vSphere CSI Driver

**Vendor:** VMware  
**CSI Driver:** vSphere CSI (built-in with OpenShift on vSphere)  
**Operator:** Cluster CSI Driver Operator (managed by OpenShift)

## Overview

The vSphere CSI driver is automatically installed and managed by OpenShift when the platform is vSphere. No manual operator installation is needed.

## Supported Backends

| Backend | Protocol | Access Modes |
|---------|----------|-------------|
| VMFS Datastore | Block (VMDK) | RWO |
| vSAN Datastore | Block (VMDK) | RWO |
| NFS Datastore | Block (VMDK) | RWO |
| vSAN File Services | File (NFS) | RWX |

## Supported Features

- Dynamic provisioning
- Volume expansion
- Snapshots (vSphere 7.0 U3+)
- Topology-aware provisioning (zone/region)
- Storage policy-based management (SPBM)
- Encryption via vSAN or VM encryption policies

## Default StorageClass

OpenShift creates a default StorageClass `thin-csi` automatically on vSphere:

```bash
oc get sc thin-csi
```

## Custom StorageClass with Storage Policy

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: vsphere-gold
provisioner: csi.vsphere.vmware.com
parameters:
  storagePolicyName: "Gold-Storage-Policy"
reclaimPolicy: Delete
allowVolumeExpansion: true
volumeBindingMode: WaitForFirstConsumer
```

## Prerequisites

- vSphere 7.0 U3+ or 8.0+ (for full CSI feature support)
- vCenter credentials configured during OpenShift installation
- Datastore accessible from all ESXi hosts running OpenShift nodes
- Storage policies defined in vCenter (optional but recommended)
- For RWX: vSAN File Services enabled
