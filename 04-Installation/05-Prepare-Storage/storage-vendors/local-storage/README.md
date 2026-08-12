# Local Storage Operator

**Vendor:** Red Hat  
**Operator:** `local-storage-operator` (via OperatorHub)

## Overview

The Local Storage Operator manages local disks on nodes and makes them available as PersistentVolumes. Used as a backend for ODF, or directly for workloads that require low-latency local storage.

## Access Modes

- RWO only (local to a single node)

## Important Characteristics

- **Not replicated** — data is lost if the node fails
- **Not portable** — PVs are bound to a specific node
- Best used as a building block for ODF or for workloads that handle their own replication (e.g., Kafka, Elasticsearch, Cassandra)

## Installation

```bash
cat <<EOF | oc apply -f -
apiVersion: v1
kind: Namespace
metadata:
  name: openshift-local-storage
---
apiVersion: operators.coreos.com/v1alpha1
kind: Subscription
metadata:
  name: local-storage-operator
  namespace: openshift-local-storage
spec:
  channel: stable
  installPlanApproval: Automatic
  name: local-storage-operator
  source: redhat-operators
  sourceNamespace: openshift-marketplace
EOF
```

## LocalVolumeDiscovery

```yaml
apiVersion: local.storage.openshift.io/v1alpha1
kind: LocalVolumeDiscovery
metadata:
  name: auto-discover-devices
  namespace: openshift-local-storage
spec:
  nodeSelector:
    nodeSelectorTerms:
      - matchExpressions:
          - key: kubernetes.io/hostname
            operator: Exists
```

## LocalVolumeSet (Automatic)

```yaml
apiVersion: local.storage.openshift.io/v1alpha1
kind: LocalVolumeSet
metadata:
  name: local-disks
  namespace: openshift-local-storage
spec:
  storageClassName: localblock
  volumeMode: Block
  deviceInclusionSpec:
    deviceTypes:
      - disk
    minSize: 100Gi
  nodeSelector:
    nodeSelectorTerms:
      - matchExpressions:
          - key: cluster.ocs.openshift.io/openshift-storage
            operator: Exists
```

## Prerequisites

- Raw, unpartitioned, unformatted disks available on target nodes
- Disks must not have filesystem signatures (run `wipefs -a` if needed)
- Nodes labeled appropriately for device discovery
