# LVM Storage (LVMS)

**Vendor:** Red Hat  
**Operator:** `lvms-operator` (via OperatorHub)  
**Previously known as:** Logical Volume Manager Storage / TopoLVM

## Overview

LVM Storage uses Linux LVM to dynamically provision local storage volumes with thin provisioning and snapshot support. Designed for Single Node OpenShift (SNO) and compact clusters where ODF is too heavy but dynamic provisioning is required.

## Access Modes

- RWO only (local to a single node)

## Features

- Thin provisioning — overcommit disk capacity
- Volume snapshots (CSI VolumeSnapshot)
- Volume cloning
- Volume expansion
- Automatic device discovery
- TopoLVM CSI driver under the hood

## Installation

```bash
cat <<EOF | oc apply -f -
apiVersion: v1
kind: Namespace
metadata:
  name: openshift-storage
---
apiVersion: operators.coreos.com/v1alpha1
kind: Subscription
metadata:
  name: lvms-operator
  namespace: openshift-storage
spec:
  channel: stable-4.22
  installPlanApproval: Automatic
  name: lvms-operator
  source: redhat-operators
  sourceNamespace: openshift-marketplace
EOF
```

## LVMCluster CR

```yaml
apiVersion: lvm.topolvm.io/v1alpha1
kind: LVMCluster
metadata:
  name: lvmcluster
  namespace: openshift-storage
spec:
  storage:
    deviceClasses:
      - name: vg1
        default: true
        thinPoolConfig:
          name: thin-pool-1
          sizePercent: 90
          overprovisionRatio: 10
        nodeSelector:
          nodeSelectorTerms:
            - matchExpressions:
                - key: kubernetes.io/hostname
                  operator: Exists
```

## StorageClass (Auto-Created)

```bash
oc get sc lvms-vg1
```

The operator creates a StorageClass `lvms-<deviceClassName>` automatically.

## VolumeSnapshotClass (Auto-Created)

```bash
oc get volumesnapshotclass lvms-vg1
```

## Best Use Cases

- Single Node OpenShift (SNO)
- Compact 3-node clusters
- Edge deployments
- Workloads requiring dynamic local provisioning without ODF overhead

## Prerequisites

- Available disks on nodes (no filesystem, no LVM VG membership)
- For SNO: at least one unused disk beyond the OS disk
- Nodes where LVMS should provision volumes must match the `nodeSelector`
