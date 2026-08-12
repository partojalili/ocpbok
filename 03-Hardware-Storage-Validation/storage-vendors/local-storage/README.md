# Local Storage Operator Pre-Installation Validation

**Vendor:** Red Hat  
**Operator:** `local-storage-operator` (via OperatorHub)

## Node Requirements

| Component | Requirement |
|-----------|-------------|
| Raw disks | At least 1 per node |
| Disk state | No partitions, no filesystem, no LVM |
| Disk type | SSD/NVMe recommended for production |

## Validation Checklist

### Disk Discovery

```bash
# List all block devices on target nodes
oc debug node/<node-name> -- chroot /host lsblk -d -o NAME,SIZE,TYPE,FSTYPE,MOUNTPOINT,MODEL

# Verify disks have no filesystem signatures
oc debug node/<node-name> -- chroot /host wipefs -n /dev/sdb

# Check for existing LVM physical volumes
oc debug node/<node-name> -- chroot /host pvs

# Check for existing RAID configurations
oc debug node/<node-name> -- chroot /host cat /proc/mdstat
```

### Disk Health Validation

```bash
# Check SMART status (if available)
oc debug node/<node-name> -- chroot /host smartctl -H /dev/sdb

# Verify disk is writable (quick test)
oc debug node/<node-name> -- chroot /host dd if=/dev/zero of=/dev/sdb bs=1M count=1 oflag=direct 2>&1 | tail -1
```

### Node Label Validation

```bash
# Identify nodes with local disks available
for node in $(oc get nodes -l node-role.kubernetes.io/worker -o name); do
  echo "=== $node ==="
  oc debug $node -- chroot /host lsblk -d -o NAME,SIZE,TYPE,FSTYPE 2>/dev/null | grep -v loop
done

# Label nodes for Local Storage Operator discovery
oc label node <node-name> cluster.ocs.openshift.io/openshift-storage=""
```

### Operator Availability

```bash
# Verify Local Storage Operator is available
oc get packagemanifest -n openshift-marketplace | grep local-storage

# Verify no conflicting operator is installed
oc get csv -n openshift-local-storage
```

### Symlink Validation (for specific device paths)

```bash
# Discover persistent device paths (by-id or by-path)
oc debug node/<node-name> -- chroot /host ls -la /dev/disk/by-id/
oc debug node/<node-name> -- chroot /host ls -la /dev/disk/by-path/
```

## Pre-Installation Summary

| Check | Expected |
|-------|----------|
| Raw disks available on target nodes | No FS, no partitions |
| Disk size adequate | Meets workload requirements |
| No LVM/RAID conflicts | Clean disk state |
| Persistent device paths available | /dev/disk/by-id/ entries |
| Nodes labeled | Storage label applied |
| Local Storage Operator in catalog | Listed in OperatorHub |
