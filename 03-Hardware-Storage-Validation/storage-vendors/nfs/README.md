# NFS Pre-Installation Validation

**Type:** Generic NFS  
**CSI Driver:** NFS CSI Driver or NFS Subdir External Provisioner

## Requirements

| Component | Requirement |
|-----------|-------------|
| NFS Server | NFSv3 or NFSv4 |
| Export | Configured with appropriate permissions |
| Network | Port 2049 accessible from all cluster nodes |

## Validation Checklist

### NFS Server Connectivity

```bash
# Test NFS port from cluster nodes
oc debug node/<node-name> -- chroot /host nc -zv <nfs-server-ip> 2049

# Test RPC portmapper (NFSv3)
oc debug node/<node-name> -- chroot /host nc -zv <nfs-server-ip> 111

# List exports
oc debug node/<node-name> -- chroot /host showmount -e <nfs-server-ip>
```

### NFS Export Permissions

```bash
# On the NFS server, verify export configuration
# Expected: export allows cluster node subnet with rw,no_root_squash or appropriate mapping
cat /etc/exports

# Example expected export:
# /exports/ocp  10.0.0.0/16(rw,sync,no_subtree_check,no_root_squash)

# Verify export is active
exportfs -v
```

### Mount Test

```bash
# Test mount from a cluster node
oc debug node/<node-name> -- chroot /host mount -t nfs <nfs-server-ip>:/exports/ocp /mnt

# Verify write access
oc debug node/<node-name> -- chroot /host touch /mnt/test-write && echo "Write OK"

# Clean up
oc debug node/<node-name> -- chroot /host rm /mnt/test-write
oc debug node/<node-name> -- chroot /host umount /mnt
```

### NFS Utils Validation

```bash
# Verify nfs-utils is available on RHCOS (included by default)
oc debug node/<node-name> -- chroot /host rpm -q nfs-utils
```

### Performance Baseline

```bash
# Quick throughput test (optional)
oc debug node/<node-name> -- chroot /host bash -c \
  "mount -t nfs <nfs-server-ip>:/exports/ocp /mnt && \
   dd if=/dev/zero of=/mnt/testfile bs=1M count=100 oflag=direct 2>&1 | tail -1 && \
   rm /mnt/testfile && umount /mnt"
```

### Firewall Validation

```bash
# Required ports
# NFSv4: TCP 2049
# NFSv3: TCP/UDP 2049, 111 (portmapper), dynamic RPC ports

# From cluster nodes
oc debug node/<node-name> -- chroot /host nc -zv <nfs-server-ip> 2049
oc debug node/<node-name> -- chroot /host nc -zv <nfs-server-ip> 111
```

### NFS Subdir Provisioner Availability (for dynamic provisioning)

```bash
# Verify Helm is available (used for provisioner installation)
helm version

# Add repo
helm repo add nfs-subdir-external-provisioner \
  https://kubernetes-sigs.github.io/nfs-subdir-external-provisioner/
helm search repo nfs-subdir-external-provisioner
```

## Pre-Installation Summary

| Check | Expected |
|-------|----------|
| NFS server reachable (port 2049) | TCP connection succeeds |
| NFS export configured | Export visible via showmount |
| Export permissions allow cluster subnet | rw access granted |
| Mount test successful | Write test passed |
| nfs-utils on RHCOS | Package installed |
| Firewall ports open | 2049, 111 (if NFSv3) |
