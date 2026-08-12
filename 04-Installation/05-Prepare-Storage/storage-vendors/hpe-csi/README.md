# HPE CSI Driver

**Vendor:** Hewlett Packard Enterprise  
**CSI Driver:** HPE CSI Driver for Kubernetes  
**Operator:** `hpe-csi-operator` (via OperatorHub)

## Supported Storage Arrays

| Product | Protocol | Access Modes |
|---------|----------|-------------|
| Alletra 5000/6000 (Nimble) | iSCSI, FC | RWO |
| Alletra 9000 (Primera/3PAR) | iSCSI, FC | RWO |
| Alletra MP | iSCSI, FC, NFS | RWO, RWX (NFS) |
| Primera | iSCSI, FC | RWO |
| Nimble Storage | iSCSI, FC | RWO |

## Supported Features

- Dynamic provisioning
- Volume expansion
- Snapshots and clones
- Raw block volumes
- Volume groups
- Volume encryption (array-level)
- Peer persistence (replication)

## Installation via Operator

```bash
cat <<EOF | oc apply -f -
apiVersion: operators.coreos.com/v1alpha1
kind: Subscription
metadata:
  name: hpe-csi-operator
  namespace: hpe-storage
spec:
  channel: stable
  installPlanApproval: Automatic
  name: hpe-csi-operator
  source: certified-operators
  sourceNamespace: openshift-marketplace
EOF
```

## Backend Configuration Example

```yaml
apiVersion: storage.hpe.com/v1
kind: HPECSIDriver
metadata:
  name: csi-driver
  namespace: hpe-storage
spec:
  backendType: nimble
  secret:
    name: hpe-backend
    namespace: hpe-storage
```

## Prerequisites

- HPE storage array with REST API enabled
- Network connectivity from all nodes to management and data interfaces
- iSCSI initiator configured on nodes (for iSCSI backends)
- Multipath configured (recommended for production)
