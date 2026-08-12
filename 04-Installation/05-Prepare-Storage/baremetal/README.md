# Prepare Storage — Bare Metal

## Storage Strategy

| Storage Type | Technology | Use Cases |
|-------------|-----------|-----------|
| Block (RWO) | ODF (Ceph RBD) | Databases, stateful apps, PV for registry |
| File (RWX) | ODF (CephFS) | Shared content, CMS, logging buffers |
| Object (S3) | ODF (Ceph RGW) or NooBaa | Backups (OADP), Loki log storage |
| Ephemeral | EmptyDir / Local | Build caches, temporary processing |

---

## Disk Preparation for ODF

Each worker node must have at least one raw, unpartitioned, unformatted disk for ODF.

**Validation (run on each worker after nodes are up):**

```bash
# SSH to worker (via debug pod)
oc debug node/worker-0.ocp.example.com -- chroot /host lsblk

# Expected output: sdb (or nvme0n1) with no partitions and no filesystem
# NAME   MAJ:MIN RM   SIZE RO TYPE MOUNTPOINT
# sda      8:0    0   250G  0 disk
# ├─sda1   8:1    0     1G  0 part /boot
# ├─sda2   8:2    0   249G  0 part /sysroot
# sdb      8:16   0   500G  0 disk            <-- ODF disk (raw)
```

If disks have residual data:

```bash
# Wipe disk signatures (destructive!)
oc debug node/worker-0.ocp.example.com -- chroot /host wipefs -a /dev/sdb
```

---

## OpenShift Data Foundation (ODF) Installation

### Step 1: Install the ODF Operator

```bash
cat <<EOF | oc apply -f -
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
---
apiVersion: v1
kind: Namespace
metadata:
  name: openshift-storage
  labels:
    openshift.io/cluster-monitoring: "true"
EOF
```

### Step 2: Label Worker Nodes for ODF

```bash
for node in worker-0 worker-1 worker-2; do
  oc label node ${node}.ocp.example.com cluster.ocs.openshift.io/openshift-storage="" --overwrite
done
```

### Step 3: Discover Local Disks

```bash
cat <<EOF | oc apply -f -
apiVersion: local.storage.openshift.io/v1alpha1
kind: LocalVolumeDiscovery
metadata:
  name: auto-discover-devices
  namespace: openshift-storage
spec:
  nodeSelector:
    nodeSelectorTerms:
      - matchExpressions:
          - key: cluster.ocs.openshift.io/openshift-storage
            operator: Exists
EOF
```

### Step 4: Create the StorageCluster

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
      placement: {}
      portable: false
      replica: 3
EOF
```

### Step 5: Verify ODF Health

```bash
# Wait for StorageCluster to become Ready
oc get storagecluster -n openshift-storage -w

# Verify Ceph health
oc rsh -n openshift-storage $(oc get pod -n openshift-storage -l app=rook-ceph-tools -o name) \
  ceph status

# Verify StorageClasses are created
oc get sc | grep ocs
# Expected:
# ocs-storagecluster-ceph-rbd       openshift-storage.rbd.csi.ceph.com    Delete   VolumeBindingImmediate   true
# ocs-storagecluster-cephfs         openshift-storage.cephfs.csi.ceph.com Delete   VolumeBindingImmediate   true
# openshift-storage.noobaa.io       openshift-storage.noobaa.io/obc       Delete   VolumeBindingImmediate   false
```

---

## Set Default StorageClass

```bash
oc patch storageclass ocs-storagecluster-ceph-rbd \
  -p '{"metadata": {"annotations": {"storageclass.kubernetes.io/is-default-class": "true"}}}'
```

## Configure Internal Registry Storage

```bash
oc patch configs.imageregistry.operator.openshift.io cluster --type merge \
  --patch '{"spec":{"storage":{"pvc":{"claim":""}}, "managementState":"Managed", "replicas":2}}'
```
