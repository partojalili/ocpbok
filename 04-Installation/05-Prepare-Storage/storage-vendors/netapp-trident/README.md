# NetApp Trident

**Vendor:** NetApp  
**CSI Driver:** Trident CSI  
**Operator:** `trident-operator` (via OperatorHub or Helm)

## Supported Backends

| Backend | Protocol | Access Modes |
|---------|----------|-------------|
| ONTAP NAS | NFS | RWO, RWX |
| ONTAP SAN | iSCSI, FC | RWO |
| ONTAP SAN Economy | iSCSI | RWO |
| SolidFire | iSCSI | RWO |
| Azure NetApp Files | NFS, SMB | RWO, RWX |
| Cloud Volumes ONTAP | NFS, iSCSI | RWO, RWX |
| Google Cloud NetApp Volumes | NFS | RWO, RWX |

## Supported Features

- Dynamic provisioning
- Volume expansion
- Snapshots and clones
- Import existing volumes
- Topology-aware provisioning
- Encryption (ONTAP-level)

## Installation via Operator

```bash
cat <<EOF | oc apply -f -
apiVersion: operators.coreos.com/v1alpha1
kind: Subscription
metadata:
  name: trident-operator
  namespace: trident
spec:
  channel: stable
  installPlanApproval: Automatic
  name: trident-operator
  source: certified-operators
  sourceNamespace: openshift-marketplace
EOF
```

## Backend Configuration Example (ONTAP NAS)

```bash
cat <<EOF | oc apply -f -
apiVersion: trident.netapp.io/v1
kind: TridentBackendConfig
metadata:
  name: ontap-nas
  namespace: trident
spec:
  version: 1
  storageDriverName: ontap-nas
  managementLIF: 10.0.30.10
  dataLIF: 10.0.30.11
  svm: ocp_svm
  credentials:
    name: ontap-credentials
EOF
```

## StorageClass Example

```bash
cat <<EOF | oc apply -f -
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: netapp-nas
provisioner: csi.trident.netapp.io
parameters:
  backendType: "ontap-nas"
reclaimPolicy: Delete
allowVolumeExpansion: true
volumeBindingMode: Immediate
EOF
```

## Prerequisites

- NetApp ONTAP 9.8+ (or compatible backend)
- Network connectivity from all nodes to management and data LIFs
- SVM configured with NFS or iSCSI enabled
- Trident-compatible ONTAP user credentials
