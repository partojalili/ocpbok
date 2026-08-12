# Pure Storage CSI Driver

**Vendor:** Pure Storage  
**CSI Driver:** Pure Storage CSI Driver  
**Operator:** `pure-csi-operator` (via OperatorHub)

## Supported Products

| Product | Protocol | Access Modes |
|---------|----------|-------------|
| FlashArray | iSCSI, FC, NVMe-oF | RWO |
| FlashBlade | NFS, S3 | RWO, RWX |

## Supported Features

- Dynamic provisioning
- Volume expansion
- Snapshots and clones
- Raw block volumes
- Topology-aware provisioning
- Volume import
- Async replication (ActiveCluster)

## Installation

```bash
cat <<EOF | oc apply -f -
apiVersion: operators.coreos.com/v1alpha1
kind: Subscription
metadata:
  name: pure-csi-operator
  namespace: pure-csi
spec:
  channel: stable
  installPlanApproval: Automatic
  name: pure-csi-operator
  source: certified-operators
  sourceNamespace: openshift-marketplace
EOF
```

## Backend Configuration

```yaml
apiVersion: pure-csi.purestorage.com/v1
kind: PureCSIDriver
metadata:
  name: pure-csi
  namespace: pure-csi
spec:
  arrays:
    FlashArrays:
      - MgmtEndPoint: 10.0.30.20
        APIToken: <api-token>
    FlashBlades:
      - MgmtEndPoint: 10.0.30.21
        APIToken: <api-token>
        NFSEndPoint: 10.0.30.22
```

## Prerequisites

- Pure Storage FlashArray with Purity 5.3+ or FlashBlade with Purity//FB 3.1+
- API token generated for CSI access
- Network connectivity from all nodes to management and data interfaces
- iSCSI initiator or FC HBA configured on nodes (for FlashArray)
