# Pure Storage Pre-Installation Validation

**Vendor:** Pure Storage  
**CSI Driver:** Pure CSI Driver  
**Supported Arrays:** FlashArray, FlashBlade

## Storage Array Requirements

| Platform | Minimum Version | Protocol |
|----------|----------------|----------|
| FlashArray | Purity 5.3+ | iSCSI, FC, NVMe-oF |
| FlashBlade | Purity//FB 3.3+ | NFS, S3 |

## Validation Checklist

### FlashArray Connectivity

```bash
# Verify management endpoint
oc debug node/<node-name> -- chroot /host curl -sk https://<flasharray-mgmt-ip>/api/api_version

# Verify iSCSI data path
oc debug node/<node-name> -- chroot /host iscsiadm -m discovery -t sendtargets -p <flasharray-iscsi-ip>
```

### FlashBlade Connectivity

```bash
# Verify management endpoint
oc debug node/<node-name> -- chroot /host curl -sk https://<flashblade-mgmt-ip>/api/api_version

# Verify NFS data path
oc debug node/<node-name> -- chroot /host showmount -e <flashblade-data-vip>
```

### OpenShift Node Prerequisites

```bash
# For iSCSI (FlashArray)
oc debug node/<node-name> -- chroot /host systemctl status iscsid
oc debug node/<node-name> -- chroot /host systemctl status multipathd

# For FC (FlashArray)
oc debug node/<node-name> -- chroot /host ls /sys/class/fc_host/

# For NFS (FlashBlade) — nfs-utils included in RHCOS
oc debug node/<node-name> -- chroot /host rpm -q nfs-utils
```

### Network Validation

```bash
# Management API (443)
oc debug node/<node-name> -- chroot /host nc -zv <array-mgmt-ip> 443

# iSCSI (3260)
oc debug node/<node-name> -- chroot /host nc -zv <iscsi-ip> 3260

# NFS (2049)
oc debug node/<node-name> -- chroot /host nc -zv <nfs-data-vip> 2049
```

### API Token Validation

```bash
# Generate API token on the array:
# FlashArray: System > Users > Create API Token
# FlashBlade: Settings > API Clients

# Verify API access with token
curl -sk -H "Authorization: Bearer <api-token>" \
  https://<flasharray-mgmt-ip>/api/2.0/arrays
```

### Pure CSI Operator Availability

```bash
# Check certified operator
oc get packagemanifest -n openshift-marketplace | grep pure

# Or Helm
helm repo add pure https://purestorage.github.io/pso-csi/
helm search repo pure/pure-pso
```

## Pre-Installation Summary

| Check | Expected |
|-------|----------|
| FlashArray management reachable | HTTPS 443 accessible |
| FlashBlade management reachable | HTTPS 443 accessible |
| iSCSI/FC/NFS data path validated | Protocol ports open |
| Purity firmware meets minimum | See table above |
| API tokens generated | One per array |
| iscsid/multipathd running (iSCSI) | Active on worker nodes |
| Pure CSI operator available | Listed in OperatorHub |
