# OpenShift Data Foundation (ODF)

**Vendor:** Red Hat  
**CSI Driver:** Ceph RBD, CephFS, NooBaa (S3)  
**Operator:** `odf-operator` (via OperatorHub)

## Supported Access Modes

| StorageClass | Protocol | Access Modes |
|-------------|----------|-------------|
| ocs-storagecluster-ceph-rbd | Block (RBD) | RWO, RWOP |
| ocs-storagecluster-cephfs | File (CephFS) | RWO, RWX |
| openshift-storage.noobaa.io | Object (S3) | N/A (OBC) |

## Supported Platforms

- Bare Metal (local disks or SAN)
- VMware vSphere (virtual disks)
- AWS (EBS-backed)
- Azure (Managed Disk-backed)
- GCP (PD-backed)

## Installation

### Step 1: Create Namespace and Install Operator

```bash
cat <<EOF | oc apply -f -
apiVersion: v1
kind: Namespace
metadata:
  name: openshift-storage
  labels:
    openshift.io/cluster-monitoring: "true"
---
apiVersion: operators.coreos.com/v1alpha1
kind: Subscription
metadata:
  name: odf-operator
  namespace: openshift-storage
spec:
  channel: stable-4.22
  installPlanApproval: Automatic
  name: odf-operator
  source: redhat-operators
  sourceNamespace: openshift-marketplace
EOF
```

### Step 2: Label Storage Nodes

```bash
for node in worker-0 worker-1 worker-2; do
  oc label node ${node}.ocp.example.com cluster.ocs.openshift.io/openshift-storage="" --overwrite
done
```

### Step 3: Create StorageCluster

```bash
cat <<EOF | oc apply -f -
apiVersion: ocs.openshift.io/v1
kind: StorageCluster
metadata:
  name: ocs-storagecluster
  namespace: openshift-storage
spec:
  manageNodes: false
  monDataDirHostPath: /var/lib/rook
  storageDeviceSets:
    - name: ocs-deviceset
      count: 1
      dataPVCTemplate:
        spec:
          accessModes:
            - ReadWriteOnce
          resources:
            requests:
              storage: 500Gi
          storageClassName: localblock
          volumeMode: Block
      portable: false
      replica: 3
EOF
```

### Step 4: Verify

```bash
oc get storagecluster -n openshift-storage -w
oc get sc | grep ocs
```

## Minimum Requirements

- 3 storage nodes with raw disks (minimum 500 GB each)
- 16 GB RAM per storage node (for ODF pods)
- 8 CPU cores per storage node
