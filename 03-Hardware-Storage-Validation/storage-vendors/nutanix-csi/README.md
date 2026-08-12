# Nutanix CSI Pre-Installation Validation

**Vendor:** Nutanix  
**CSI Driver:** Nutanix CSI Volume Driver  
**Supported Backends:** Nutanix Volumes (ABS), Nutanix Files (AFS)

## Requirements

| Component | Requirement |
|-----------|-------------|
| AOS | 6.5+ |
| Prism Central | pc.2022.6+ |
| Prism Element | Registered with Prism Central |
| Nutanix Files | 4.2+ (for NFS/RWX) |

## Validation Checklist

### Prism Central Connectivity

```bash
# Verify Prism Central API reachable
oc debug node/<node-name> -- chroot /host curl -sk https://<prism-central-ip>:9440/api/nutanix/v3/clusters/list \
  -X POST -H "Content-Type: application/json" -d '{"kind":"cluster"}'

# Verify Prism Element API
oc debug node/<node-name> -- chroot /host curl -sk https://<prism-element-ip>:9440/api/nutanix/v2.0/cluster/
```

### Nutanix Volumes (ABS) Validation

```bash
# Verify iSCSI data services IP is configured
# On Prism Element: Settings > Cluster Details > iSCSI Data Services IP
# Must be reachable from OpenShift nodes

# Test iSCSI connectivity
oc debug node/<node-name> -- chroot /host iscsiadm -m discovery -t sendtargets -p <iscsi-data-services-ip>:3260

# Verify iscsid is running
oc debug node/<node-name> -- chroot /host systemctl status iscsid
```

### Nutanix Files (AFS) Validation

```bash
# Verify Files server is deployed and healthy
# On Prism Element: File Server > check status

# Verify NFS export reachable
oc debug node/<node-name> -- chroot /host showmount -e <files-server-ip>

# Test NFS port
oc debug node/<node-name> -- chroot /host nc -zv <files-server-ip> 2049
```

### Network Validation

```bash
# Prism Central API (9440)
oc debug node/<node-name> -- chroot /host nc -zv <prism-central-ip> 9440

# Prism Element API (9440)
oc debug node/<node-name> -- chroot /host nc -zv <prism-element-ip> 9440

# iSCSI Data Services (3260)
oc debug node/<node-name> -- chroot /host nc -zv <iscsi-data-services-ip> 3260

# NFS (2049) — if using Nutanix Files
oc debug node/<node-name> -- chroot /host nc -zv <files-server-ip> 2049
```

### Storage Container Validation

```bash
# Verify storage container exists and has capacity
# On Prism Element: Storage > Storage Containers
# Note the container name — used in StorageClass
```

### Credentials

```bash
# Create Prism Central credentials secret (dry-run test)
oc create secret generic ntnx-secret \
  --from-literal=key='<prism-central-ip>:9440:<username>:<password>' \
  -n ntnx-system --dry-run=client -o yaml
```

### Operator Availability

```bash
# Check Nutanix CSI operator
oc get packagemanifest -n openshift-marketplace | grep nutanix
```

## Pre-Installation Summary

| Check | Expected |
|-------|----------|
| Prism Central reachable (port 9440) | API responds |
| AOS version 6.5+ | Verified in Prism |
| iSCSI Data Services IP configured | IP set and reachable |
| iscsid running on worker nodes | Active |
| Storage container identified | Name and available capacity |
| Nutanix Files healthy (if RWX needed) | File server operational |
| Credentials verified | API access confirmed |
| Nutanix CSI operator available | Listed in OperatorHub |
