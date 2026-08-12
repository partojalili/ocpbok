# IBM Storage CSI Drivers

**Vendor:** IBM  
**Operator:** `ibm-block-csi-operator` / `ibm-spectrum-scale-csi` (via OperatorHub)

## Supported Products

| Product | CSI Driver | Protocol | Access Modes |
|---------|-----------|----------|-------------|
| FlashSystem 5000/7000/9000 | IBM Block CSI | iSCSI, FC | RWO |
| SAN Volume Controller (SVC) | IBM Block CSI | iSCSI, FC | RWO |
| Spectrum Virtualize | IBM Block CSI | iSCSI, FC | RWO |
| Spectrum Scale (GPFS) | IBM Spectrum Scale CSI | GPFS / NFS | RWO, RWX |
| Spectrum Fusion | IBM Fusion CSI | Block, File | RWO, RWX |

## Supported Features

- Dynamic provisioning
- Volume expansion
- Snapshots
- Volume cloning
- Raw block volumes
- Topology-aware provisioning
- Replication (array-based)

## Installation (IBM Block CSI)

```bash
cat <<EOF | oc apply -f -
apiVersion: operators.coreos.com/v1alpha1
kind: Subscription
metadata:
  name: ibm-block-csi-operator
  namespace: openshift-operators
spec:
  channel: stable
  installPlanApproval: Automatic
  name: ibm-block-csi-operator
  source: certified-operators
  sourceNamespace: openshift-marketplace
EOF
```

## Secret Configuration

```bash
oc create secret generic flashsystem-creds -n openshift-operators \
  --from-literal=management_address=10.0.30.30 \
  --from-literal=username=admin \
  --from-literal=password=<password>
```

## StorageClass Example

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: ibm-block-gold
provisioner: block.csi.ibm.com
parameters:
  SpaceEfficiency: thin
  pool: pool0
  csi.storage.k8s.io/provisioner-secret-name: flashsystem-creds
  csi.storage.k8s.io/provisioner-secret-namespace: openshift-operators
reclaimPolicy: Delete
allowVolumeExpansion: true
```

## Prerequisites

- IBM storage array with REST API enabled
- Network connectivity from all nodes to array management and data ports
- iSCSI initiator or FC HBA configured on nodes
- Multipath configured (required for production)
