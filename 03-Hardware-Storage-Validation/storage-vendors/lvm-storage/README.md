# LVM Storage (LVMS) Pre-Installation Validation

**Vendor:** Red Hat  
**Operator:** `lvms-operator` (via OperatorHub)

## Node Requirements

| Component | Minimum |
|-----------|---------|
| Available disks | 1+ unused disk per node (beyond OS disk) |
| Disk state | No existing LVM VG, no filesystem |
| Best for | SNO, compact 3-node clusters, edge |

## Validation Checklist

### Disk Discovery

```bash
# List available disks on each node
oc debug node/<node-name> -- chroot /host lsblk -d -o NAME,SIZE,TYPE,FSTYPE,MOUNTPOINT

# Verify disks are not part of an existing VG
oc debug node/<node-name> -- chroot /host vgs
oc debug node/<node-name> -- chroot /host pvs

# Verify no filesystem on candidate disks
oc debug node/<node-name> -- chroot /host blkid /dev/sdb
```

### Kernel Module Validation

```bash
# LVMS requires LVM and device-mapper kernel modules
oc debug node/<node-name> -- chroot /host modinfo dm_thin_pool
oc debug node/<node-name> -- chroot /host modinfo dm_snapshot
oc debug node/<node-name> -- chroot /host lsmod | grep dm_thin_pool
```

### Node Count Validation

```bash
# LVMS is designed for SNO and compact clusters
# For 3+ worker-node clusters, consider ODF instead
oc get nodes -l node-role.kubernetes.io/worker --no-headers | wc -l

# For SNO, verify single node
oc get nodes --no-headers | wc -l
```

### Operator Availability

```bash
# Verify LVMS operator is available
oc get packagemanifest -n openshift-marketplace | grep lvms

# Verify no conflicting local storage operator
oc get csv -A | grep -E "local-storage|lvms"
```

### Storage Capacity Estimation

```bash
# Calculate total available disk space across nodes
for node in $(oc get nodes -l node-role.kubernetes.io/worker -o name); do
  echo "=== $node ==="
  oc debug $node -- chroot /host lsblk -d -b -o NAME,SIZE,TYPE 2>/dev/null | grep disk
done
```

### Conflict Check

```bash
# LVMS and Local Storage Operator can conflict
# Ensure disks are not claimed by another operator
oc get localvolumediscovery -A
oc get localvolumeset -A
```

## Pre-Installation Summary

| Check | Expected |
|-------|----------|
| Unused disks available | At least 1 per node beyond OS disk |
| No existing LVM VG on candidate disks | `pvs`/`vgs` clean |
| dm_thin_pool module available | Module loaded or loadable |
| LVMS operator in catalog | Listed in OperatorHub |
| No conflicting local storage operator | No resource conflicts |
| Node topology fits LVMS | SNO or compact cluster |
