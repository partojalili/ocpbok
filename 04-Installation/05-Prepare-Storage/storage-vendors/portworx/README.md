# Portworx

**Vendor:** Pure Storage (Portworx)  
**CSI Driver:** Portworx CSI  
**Operator:** `portworx-certified` (via OperatorHub)

## Overview

Portworx is a software-defined storage platform that runs on the OpenShift nodes themselves, aggregating local or cloud disks into a shared storage pool. It provides block, file, and object storage with built-in replication, encryption, and DR.

## Supported Access Modes

| StorageClass | Access Modes |
|-------------|-------------|
| portworx-sc (replicated block) | RWO |
| portworx-shared-sc (shared volumes) | RWX |
| portworx-db-sc (database-optimized) | RWO |

## Supported Platforms

- Bare Metal
- VMware vSphere
- AWS
- Azure
- GCP

## Supported Features

- Software-defined storage (no external array needed)
- Synchronous replication (RF 1/2/3)
- Encryption at rest
- Snapshots and clones
- Cloud snapshots (to S3/Azure Blob/GCS)
- Application-consistent backup (PX-Backup)
- Disaster recovery (PX-DR / async replication)
- Autopilot (automated capacity management)
- Hyper-convergence with OpenShift

## Installation

```bash
cat <<EOF | oc apply -f -
apiVersion: operators.coreos.com/v1alpha1
kind: Subscription
metadata:
  name: portworx-certified
  namespace: kube-system
spec:
  channel: stable
  installPlanApproval: Automatic
  name: portworx-certified
  source: certified-operators
  sourceNamespace: openshift-marketplace
EOF
```

## StorageCluster Example

```yaml
apiVersion: core.libopenstorage.org/v1
kind: StorageCluster
metadata:
  name: px-cluster
  namespace: kube-system
spec:
  image: portworx/oci-monitor:3.1.0
  kvdb:
    internal: true
  storage:
    useAll: true
    journalDevice: auto
  network:
    dataInterface: eth0
    mgmtInterface: eth0
```

## Prerequisites

- Minimum 3 worker nodes with raw disks (for Portworx storage pool)
- 8 GB RAM and 4 CPU cores per Portworx node (for PX processes)
- Unique disk per node (not shared across nodes)
- Portworx license (Essentials is free for up to 5 nodes / 5 TB)
- Network connectivity between all Portworx nodes (port 9001-9022)
