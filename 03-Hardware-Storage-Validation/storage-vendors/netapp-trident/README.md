# NetApp Trident Pre-Installation Validation

**Vendor:** NetApp  
**CSI Driver:** Trident

## Storage Backend Requirements

| Backend | Protocol | Minimum ONTAP Version |
|---------|----------|-----------------------|
| ONTAP NAS | NFS | ONTAP 9.8+ |
| ONTAP SAN | iSCSI, FC, NVMe/TCP | ONTAP 9.8+ |
| Element (SolidFire) | iSCSI | Element 12+ |
| Azure NetApp Files | NFS, SMB | N/A |
| Cloud Volumes ONTAP | NFS, iSCSI | ONTAP 9.8+ |

## Validation Checklist

### ONTAP Backend Connectivity

```bash
# Verify management LIF is reachable from cluster nodes
oc debug node/<node-name> -- chroot /host curl -sk https://<ontap-management-lif>/api/cluster

# Verify data LIF is reachable (NFS)
oc debug node/<node-name> -- chroot /host showmount -e <ontap-data-lif>

# Verify iSCSI connectivity (if using SAN)
oc debug node/<node-name> -- chroot /host iscsiadm -m discovery -t sendtargets -p <ontap-iscsi-lif>
```

### ONTAP Configuration

```bash
# On the ONTAP cluster (via SSH or System Manager):
# Verify SVM exists
vserver show -vserver <svm-name>

# Verify NFS is enabled on SVM
vserver nfs show -vserver <svm-name>

# Verify export policy allows cluster nodes
export-policy rule show -vserver <svm-name> -policyname default

# Verify aggregate has free space
aggr show -fields availsize

# Verify user credentials
security login show -vserver <svm-name>
```

### OpenShift Node Validation

```bash
# Verify NFS utils available (included in RHCOS)
oc debug node/<node-name> -- chroot /host rpm -q nfs-utils

# For iSCSI: verify iscsid is running
oc debug node/<node-name> -- chroot /host systemctl status iscsid

# For iSCSI: enable multipathing via MachineConfig if needed
oc get machineconfig | grep multipath
```

### Network Validation

```bash
# Test NFS port (2049) from cluster nodes
oc debug node/<node-name> -- chroot /host nc -zv <ontap-data-lif> 2049

# Test iSCSI port (3260) if applicable
oc debug node/<node-name> -- chroot /host nc -zv <ontap-iscsi-lif> 3260

# Test management API port (443)
oc debug node/<node-name> -- chroot /host nc -zv <ontap-management-lif> 443
```

### Trident Operator Availability

```bash
# Verify Trident operator is available in OperatorHub
oc get packagemanifest -n openshift-marketplace | grep trident
```

## Pre-Installation Summary

| Check | Expected |
|-------|----------|
| ONTAP management LIF reachable | HTTPS 443 accessible |
| Data LIF reachable | NFS 2049 or iSCSI 3260 accessible |
| SVM configured with NFS/iSCSI | Protocol enabled |
| Export policy allows cluster subnet | Rules configured |
| Aggregate free space | Sufficient for expected PV capacity |
| Trident operator in catalog | Listed in OperatorHub |
