# Portworx Pre-Installation Validation

**Vendor:** Pure Storage (Portworx)  
**CSI Driver:** Portworx CSI  
**Operator:** Portworx Enterprise Operator

## Node Requirements

| Component | Minimum | Recommended |
|-----------|---------|-------------|
| Worker nodes | 3 | 3+ |
| CPU per node | 4 vCPU | 8+ vCPU |
| RAM per node | 8 GB | 16+ GB |
| Disks per node | 1 raw disk | 2+ SSD/NVMe |
| Disk size | 64 GB | 256 GB+ |
| Network | 1 Gbps | 10 Gbps+ |

## Validation Checklist

### Disk Validation

```bash
# List available disks (must be raw — no partitions, no filesystem)
oc debug node/<node-name> -- chroot /host lsblk -d -o NAME,SIZE,TYPE,FSTYPE,MOUNTPOINT

# Verify disks are clean
oc debug node/<node-name> -- chroot /host wipefs -n /dev/sdb
```

### Network Validation

```bash
# Portworx requires two networks:
# - Data network: inter-node replication
# - Management network: cluster coordination

# Verify inter-node connectivity
oc debug node/<node-name> -- chroot /host nc -zv <other-node-ip> 9001
oc debug node/<node-name> -- chroot /host nc -zv <other-node-ip> 9002
oc debug node/<node-name> -- chroot /host nc -zv <other-node-ip> 9010
oc debug node/<node-name> -- chroot /host nc -zv <other-node-ip> 9012
oc debug node/<node-name> -- chroot /host nc -zv <other-node-ip> 9014
```

### Portworx Required Ports

| Port | Protocol | Purpose |
|------|----------|---------|
| 9001 | TCP | Portworx API |
| 9002 | TCP | Portworx SDK |
| 9010 | TCP | Cluster management |
| 9012 | TCP | Gossip |
| 9014 | TCP | Internal REST gateway |
| 9019 | TCP | Cloud drive operations |
| 9020-9021 | TCP | Object store |

### Kernel Module Validation

```bash
# Portworx requires specific kernel modules
oc debug node/<node-name> -- chroot /host modinfo dm_thin_pool
oc debug node/<node-name> -- chroot /host modinfo dm_mirror
```

### Portworx License and Activation

```bash
# Verify Portworx Essentials (free) or Enterprise license
# Obtain activation ID from Pure Storage portal

# For air-gapped: download Portworx images and mirror them
```

### Operator Availability

```bash
# Check certified operator
oc get packagemanifest -n openshift-marketplace | grep portworx

# Verify CatalogSource
oc get catalogsource -n openshift-marketplace
```

## Pre-Installation Summary

| Check | Expected |
|-------|----------|
| 3+ worker nodes with raw disks | Unformatted disks visible |
| Inter-node ports 9001-9021 open | All ports reachable |
| 10 Gbps network between nodes | Low-latency path verified |
| Kernel modules available | dm_thin_pool, dm_mirror loaded |
| Portworx license obtained | Activation ID available |
| Operator available in catalog | portworx-certified listed |
