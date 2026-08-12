# IBM Storage Pre-Installation Validation

**Vendor:** IBM  
**CSI Driver:** IBM Block CSI Driver  
**Supported Arrays:** FlashSystem, SAN Volume Controller (SVC), Spectrum Scale (GPFS)

## Storage Array Requirements

| Platform | Minimum Version | Protocol |
|----------|----------------|----------|
| FlashSystem 5200/7300/9500 | 8.5.0+ | iSCSI, FC, NVMe/FC |
| SAN Volume Controller (SVC) | 8.5.0+ | iSCSI, FC |
| Spectrum Scale (GPFS) | 5.1.4+ | Native POSIX |

## Validation Checklist

### Block Storage Connectivity (FlashSystem / SVC)

```bash
# Verify management endpoint
oc debug node/<node-name> -- chroot /host curl -sk https://<array-mgmt-ip>/

# Verify iSCSI data path
oc debug node/<node-name> -- chroot /host iscsiadm -m discovery -t sendtargets -p <iscsi-target-ip>

# Verify FC (if applicable)
oc debug node/<node-name> -- chroot /host ls /sys/class/fc_host/
```

### Spectrum Scale Connectivity

```bash
# Verify GUI endpoint
oc debug node/<node-name> -- chroot /host curl -sk https://<scale-gui-ip>/scalemgmt/v2/cluster

# Verify GPFS filesystem is mounted on Scale nodes
# (Run on Scale node, not OpenShift node)
mmlsfs all -T
mmhealth cluster show
```

### OpenShift Node Prerequisites

```bash
# For iSCSI
oc debug node/<node-name> -- chroot /host systemctl status iscsid
oc debug node/<node-name> -- chroot /host systemctl status multipathd

# Verify multipath config
oc debug node/<node-name> -- chroot /host multipath -ll

# For FC: verify HBA
oc debug node/<node-name> -- chroot /host cat /sys/class/fc_host/host*/port_name
```

### Network Validation

```bash
# Management API (443)
oc debug node/<node-name> -- chroot /host nc -zv <array-mgmt-ip> 443

# iSCSI (3260)
oc debug node/<node-name> -- chroot /host nc -zv <iscsi-ip> 3260

# Spectrum Scale GUI API (443)
oc debug node/<node-name> -- chroot /host nc -zv <scale-gui-ip> 443
```

### Credentials

```bash
# IBM Block CSI requires a management user with these roles:
# FlashSystem/SVC: CopyOperator, Administrator (for thin provisioning)
# Verify credentials work
curl -sk -u <user>:<password> https://<array-mgmt-ip>/restapi/v1/lssystem
```

### Operator Availability

```bash
# Check IBM operator in OperatorHub
oc get packagemanifest -n openshift-marketplace | grep ibm-block

# For Spectrum Scale
oc get packagemanifest -n openshift-marketplace | grep ibm-spectrum-scale
```

## Pre-Installation Summary

| Check | Expected |
|-------|----------|
| Array management reachable | HTTPS 443 accessible |
| iSCSI/FC data path validated | Protocol ports open, zoning complete |
| Array firmware meets minimum | See table above |
| iscsid/multipathd running | Active on worker nodes |
| Storage pool identified | Pool name and capacity confirmed |
| Admin credentials verified | API access confirmed |
| IBM CSI operator available | Listed in OperatorHub |
