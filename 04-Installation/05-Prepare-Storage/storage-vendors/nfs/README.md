# NFS Storage

**Type:** Generic NFS  
**CSI Driver:** NFS CSI Driver or NFS Subdir External Provisioner  
**Operator:** Community operator or manual deployment

## Overview

NFS is the simplest way to provide RWX storage on OpenShift. It works on any platform where an NFS server is available. Not recommended for production databases or high-IOPS workloads.

## Access Modes

- RWO, RWX, ROX

## Options

| Option | Description |
|--------|-------------|
| Static PV | Manually create PV pointing to an existing NFS export |
| NFS Subdir External Provisioner | Automatically creates subdirectories on an NFS share per PVC |
| NFS CSI Driver | CSI-based dynamic provisioning |

## Static PV Example

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: nfs-pv
spec:
  capacity:
    storage: 100Gi
  accessModes:
    - ReadWriteMany
  nfs:
    server: 10.0.30.50
    path: /exports/ocp
  persistentVolumeReclaimPolicy: Retain
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: nfs-pvc
spec:
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 100Gi
  volumeName: nfs-pv
```

## NFS Subdir External Provisioner (Dynamic)

```bash
# Install via Helm
helm repo add nfs-subdir-external-provisioner \
  https://kubernetes-sigs.github.io/nfs-subdir-external-provisioner/

helm install nfs-provisioner nfs-subdir-external-provisioner/nfs-subdir-external-provisioner \
  --set nfs.server=10.0.30.50 \
  --set nfs.path=/exports/ocp \
  --set storageClass.name=nfs-dynamic \
  --namespace nfs-provisioner --create-namespace
```

## Prerequisites

- NFS server accessible from all cluster nodes
- NFS export configured with appropriate permissions
- Firewall allows NFS traffic (ports 2049, 111)
- `nfs-utils` available on RHCOS (included by default)

## Limitations

- No volume snapshots
- No volume expansion (provisioner-dependent)
- Performance depends on the NFS server and network
- Not recommended for etcd, databases, or latency-sensitive workloads
