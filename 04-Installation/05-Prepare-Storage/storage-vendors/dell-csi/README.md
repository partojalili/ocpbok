# Dell CSI Drivers

**Vendor:** Dell Technologies  
**Operator:** `dell-csi-operator` (via OperatorHub)

## Supported Storage Arrays

| Product | CSI Driver | Protocol | Access Modes |
|---------|-----------|----------|-------------|
| PowerStore | csi-powerstore | iSCSI, FC, NFS | RWO, RWX (NFS) |
| PowerScale (Isilon) | csi-powerscale | NFS | RWO, RWX |
| PowerFlex (VxFlex) | csi-powerflex | Block (SDC) | RWO |
| PowerMax | csi-powermax | iSCSI, FC | RWO |
| Unity XT | csi-unity | iSCSI, FC, NFS | RWO, RWX (NFS) |

## Supported Features

- Dynamic provisioning
- Volume expansion
- Snapshots and clones
- Topology-aware provisioning
- Replication (array-based)
- Volume health monitoring

## Installation via Dell CSI Operator

```bash
# Install the Dell CSI Operator from OperatorHub
cat <<EOF | oc apply -f -
apiVersion: operators.coreos.com/v1alpha1
kind: Subscription
metadata:
  name: dell-csi-operator
  namespace: dell-csi-operator
spec:
  channel: stable
  installPlanApproval: Automatic
  name: dell-csi-operator
  source: certified-operators
  sourceNamespace: openshift-marketplace
EOF
```

## PowerStore Example

### Secret

```bash
oc create secret generic powerstore-creds -n dell-csi-operator \
  --from-literal=username=admin \
  --from-literal=password=<password>
```

### CSI Driver CR

```yaml
apiVersion: storage.dell.com/v1
kind: CSIPowerStore
metadata:
  name: powerstore
  namespace: dell-csi-operator
spec:
  driver:
    configVersion: v2.10.0
    authSecret: powerstore-creds
    common:
      image: dellemc/csi-powerstore:v2.10.0
    storageCapacity:
      enabled: true
```

## Prerequisites

- Dell storage array with REST API enabled
- Network connectivity from all nodes to array management and data interfaces
- iSCSI initiator or multipath configured on nodes (for block)
- NFS client packages on nodes (for file)
