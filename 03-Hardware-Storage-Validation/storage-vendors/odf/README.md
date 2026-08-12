# ODF Pre-Installation Validation

**Vendor:** Red Hat OpenShift Data Foundation  
**Components:** Ceph (RBD, CephFS), NooBaa (MCG)

## Node Requirements

| Component | Minimum | Recommended |
|-----------|---------|-------------|
| ODF worker nodes | 3 | 3+ (dedicated) |
| CPU per node | 16 vCPU | 24+ vCPU |
| RAM per node | 64 GB | 96+ GB |
| OSD disks per node | 1 | 2+ |
| OSD disk size | 100 GB | 500 GB+ |
| Disk type | SSD | NVMe |

## Validation Checklist

### Node Readiness

```bash
# Verify at least 3 worker nodes for ODF
oc get nodes -l node-role.kubernetes.io/worker --no-headers | wc -l

# Check node resources
oc adm top nodes

# Label ODF nodes
oc label node <node1> cluster.ocs.openshift.io/openshift-storage=""
oc label node <node2> cluster.ocs.openshift.io/openshift-storage=""
oc label node <node3> cluster.ocs.openshift.io/openshift-storage=""

# Verify labels
oc get nodes -l cluster.ocs.openshift.io/openshift-storage
```

### Disk Validation

```bash
# List available disks on each ODF node (SSH or debug pod)
oc debug node/<node-name> -- chroot /host lsblk -d -o NAME,SIZE,TYPE,MOUNTPOINT

# Verify disks are clean (no partitions, no filesystem)
oc debug node/<node-name> -- chroot /host wipefs -n /dev/sdb

# Check for existing LVM signatures
oc debug node/<node-name> -- chroot /host pvs
```

### Network Validation

```bash
# ODF requires high-bandwidth, low-latency between storage nodes
# Verify MTU (9000 recommended for storage network)
oc debug node/<node-name> -- chroot /host ip link show

# Test inter-node bandwidth (iperf3 between ODF nodes)
# Minimum: 10 Gbps recommended
```

### OperatorHub Access

```bash
# Verify ODF operator is available
oc get packagemanifest -n openshift-marketplace | grep odf

# Check catalog source health
oc get catalogsource -n openshift-marketplace
```

### Resource Quota Validation

```bash
# Ensure openshift-storage namespace has no restrictive quotas
oc get resourcequota -n openshift-storage
```

## Pre-Installation Summary

| Check | Command | Expected |
|-------|---------|----------|
| 3+ labeled ODF nodes | `oc get nodes -l cluster.ocs.openshift.io/openshift-storage` | 3+ nodes |
| Raw disks available | `lsblk` on each node | Unformatted disks |
| 10 Gbps+ network | `iperf3` between ODF nodes | ≥10 Gbps |
| ODF operator available | `oc get packagemanifest \| grep odf` | odf-operator listed |
| Sufficient CPU/RAM | `oc adm top nodes` | Meets minimums |
