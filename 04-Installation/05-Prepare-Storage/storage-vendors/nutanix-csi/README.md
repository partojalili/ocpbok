# Nutanix CSI Driver

**Vendor:** Nutanix  
**CSI Driver:** Nutanix CSI Volume Driver  
**Operator:** `nutanix-csi-operator` (via OperatorHub)

## Supported Products

| Product | Protocol | Access Modes |
|---------|----------|-------------|
| Nutanix Volumes (ABS) | iSCSI | RWO |
| Nutanix Files (AFS) | NFS | RWO, RWX |
| Nutanix Objects | S3 | N/A (application-level) |

## Supported Features

- Dynamic provisioning
- Volume expansion
- Snapshots and clones
- Raw block volumes
- Volume metrics
- Topology-aware provisioning

## Installation

```bash
cat <<EOF | oc apply -f -
apiVersion: operators.coreos.com/v1alpha1
kind: Subscription
metadata:
  name: nutanix-csi-operator
  namespace: openshift-cluster-csi-drivers
spec:
  channel: stable
  installPlanApproval: Automatic
  name: nutanix-csi-operator
  source: certified-operators
  sourceNamespace: openshift-marketplace
EOF
```

## Secret Configuration

```bash
oc create secret generic nutanix-csi-credentials \
  -n openshift-cluster-csi-drivers \
  --from-literal=key='<prism-element-ip>:<prism-port>:<username>:<password>'
```

## StorageClass Example (Volumes)

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: nutanix-volumes
provisioner: csi.nutanix.com
parameters:
  storageType: NutanixVolumes
  csi.storage.k8s.io/provisioner-secret-name: nutanix-csi-credentials
  csi.storage.k8s.io/provisioner-secret-namespace: openshift-cluster-csi-drivers
  storageContainer: default-container
reclaimPolicy: Delete
allowVolumeExpansion: true
```

## StorageClass Example (Files)

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: nutanix-files
provisioner: csi.nutanix.com
parameters:
  storageType: NutanixFiles
  nfsServerName: nutanix-files-server
  csi.storage.k8s.io/provisioner-secret-name: nutanix-csi-credentials
  csi.storage.k8s.io/provisioner-secret-namespace: openshift-cluster-csi-drivers
reclaimPolicy: Delete
allowVolumeExpansion: true
```

## Prerequisites

- Nutanix AHV cluster with Prism Element and Prism Central
- Nutanix Volumes (ABS) enabled for block storage
- Nutanix Files (AFS) deployed for file storage
- iSCSI data services IP configured on Prism Element
- Network connectivity from all nodes to Prism Element and data services IP
