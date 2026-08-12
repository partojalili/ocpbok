# Dell CSI Pre-Installation Validation

**Vendor:** Dell Technologies  
**CSI Drivers:** PowerStore, PowerScale, PowerFlex, PowerMax, Unity XT

## Storage Array Requirements

| Platform | Minimum Version | Protocol |
|----------|----------------|----------|
| PowerStore | PowerStoreOS 3.0+ | iSCSI, FC, NFS, NVMe/TCP |
| PowerScale (Isilon) | OneFS 9.4+ | NFS |
| PowerFlex | PowerFlex 3.6+ | SDC (block) |
| PowerMax | PowerMaxOS 10+ | iSCSI, FC |
| Unity XT | Unity OE 5.2+ | iSCSI, FC, NFS |

## Validation Checklist

### Array Connectivity

```bash
# Verify management endpoint reachable
oc debug node/<node-name> -- chroot /host curl -sk https://<array-management-ip>/api/version

# Verify data path — iSCSI
oc debug node/<node-name> -- chroot /host iscsiadm -m discovery -t sendtargets -p <iscsi-target-ip>

# Verify data path — NFS (PowerScale/PowerStore)
oc debug node/<node-name> -- chroot /host showmount -e <nfs-endpoint>
```

### PowerFlex-Specific Validation

```bash
# Verify SDC kernel module compatibility with RHCOS kernel
oc debug node/<node-name> -- chroot /host uname -r

# Verify MDM connectivity
oc debug node/<node-name> -- chroot /host nc -zv <mdm-ip> 6611
oc debug node/<node-name> -- chroot /host nc -zv <mdm-ip> 9011
```

### OpenShift Node Prerequisites

```bash
# For iSCSI backends: verify iscsid
oc debug node/<node-name> -- chroot /host systemctl status iscsid

# For FC backends: verify HBA
oc debug node/<node-name> -- chroot /host ls /sys/class/fc_host/

# For multipathing
oc debug node/<node-name> -- chroot /host multipathd show status
```

### Network Validation

```bash
# Management API (443)
oc debug node/<node-name> -- chroot /host nc -zv <array-mgmt-ip> 443

# iSCSI data (3260)
oc debug node/<node-name> -- chroot /host nc -zv <iscsi-ip> 3260

# NFS data (2049)
oc debug node/<node-name> -- chroot /host nc -zv <nfs-ip> 2049
```

### CSI Operator Availability

```bash
# Check Dell CSI operator in OperatorHub (community or certified)
oc get packagemanifest -n openshift-marketplace | grep dell

# Or verify Helm/manual installation prerequisites
helm version
```

### Credentials and Permissions

```bash
# Verify storage admin credentials
# Create the secret ahead of time to validate
oc create secret generic dell-creds \
  --from-literal=username=<admin-user> \
  --from-literal=password=<admin-password> \
  -n dell-csi-driver --dry-run=client -o yaml
```

## Pre-Installation Summary

| Check | Expected |
|-------|----------|
| Array management endpoint reachable | HTTPS 443 accessible |
| Data path (iSCSI/NFS/FC) validated | Protocol ports open |
| Array firmware meets minimum version | See table above |
| Storage pool/tier identified | Pool name for StorageClass |
| Admin credentials available | Username/password or API token |
| CSI operator/driver available | OperatorHub or Helm ready |
