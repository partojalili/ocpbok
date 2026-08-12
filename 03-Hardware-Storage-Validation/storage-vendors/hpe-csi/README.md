# HPE CSI Pre-Installation Validation

**Vendor:** Hewlett Packard Enterprise  
**CSI Driver:** HPE CSI Driver for Kubernetes  
**Supported Arrays:** Alletra, Primera, Nimble, 3PAR

## Storage Array Requirements

| Platform | Minimum Version | Protocol |
|----------|----------------|----------|
| Alletra 9000 / Primera | Primera OS 4.3+ | iSCSI, FC |
| Alletra 6000 / Nimble | NimbleOS 6.0+ | iSCSI, FC |
| Alletra MP | Alletra OS | iSCSI, FC, NFS |
| 3PAR | 3PAR OS 3.3.1+ | iSCSI, FC |

## Validation Checklist

### Array Connectivity

```bash
# Verify management endpoint
oc debug node/<node-name> -- chroot /host curl -sk https://<array-management-ip>/api/v1/storage-systems

# Verify iSCSI data path
oc debug node/<node-name> -- chroot /host iscsiadm -m discovery -t sendtargets -p <iscsi-target-ip>
```

### OpenShift Node Prerequisites

```bash
# Verify iSCSI initiator
oc debug node/<node-name> -- chroot /host systemctl status iscsid
oc debug node/<node-name> -- chroot /host cat /etc/iscsi/initiatorname.iscsi

# Verify multipath
oc debug node/<node-name> -- chroot /host systemctl status multipathd

# For FC: verify HBA ports
oc debug node/<node-name> -- chroot /host ls /sys/class/fc_host/
```

### Network Validation

```bash
# Management API
oc debug node/<node-name> -- chroot /host nc -zv <array-mgmt-ip> 443

# iSCSI
oc debug node/<node-name> -- chroot /host nc -zv <iscsi-ip> 3260

# HPE CSI Info Metrics (optional)
# Port 8080 for CSI driver metrics
```

### HPE CSI Operator Availability

```bash
# Check certified operator
oc get packagemanifest -n openshift-marketplace | grep hpe

# Verify Helm repo (alternative)
helm repo add hpe-storage https://hpe-storage.github.io/co-deployments/
helm search repo hpe-storage
```

### Array Configuration

```bash
# On the array (via CLI or SSMC/Cloud Console):
# - Create a dedicated user for CSI operations
# - Verify storage pool has free capacity
# - Verify iSCSI/FC ports are configured and zoned
# - Verify host group or initiator group exists for cluster nodes
```

## Pre-Installation Summary

| Check | Expected |
|-------|----------|
| Array management reachable | HTTPS 443 accessible |
| iSCSI or FC data path validated | Ports open, zoning complete |
| Array firmware meets minimum version | See table above |
| iscsid/multipathd running | Active on all worker nodes |
| Storage pool identified | Pool name and available capacity |
| CSI admin credentials available | Dedicated user/password |
| HPE CSI operator available | Listed in OperatorHub |
