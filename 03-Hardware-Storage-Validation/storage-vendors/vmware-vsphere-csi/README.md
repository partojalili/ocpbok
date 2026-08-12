# VMware vSphere CSI Pre-Installation Validation

**Vendor:** VMware (Broadcom)  
**CSI Driver:** vSphere CSI (built-in with OpenShift on vSphere)  
**Operator:** Cluster CSI Driver Operator (managed by OpenShift)

## Requirements

| Component | Minimum Version |
|-----------|----------------|
| vCenter Server | 7.0 U3+ |
| ESXi | 7.0 U3+ |
| VM Hardware Version | 15+ |
| VMware Tools | 11.0.5+ (open-vm-tools in RHCOS) |

## Validation Checklist

### vCenter Connectivity

```bash
# Verify vCenter API reachable from cluster nodes
oc debug node/<node-name> -- chroot /host curl -sk https://<vcenter-fqdn>/sdk

# Using govc (from a jump host with govc installed)
export GOVC_URL=https://<vcenter-fqdn>/sdk
export GOVC_USERNAME=<user>@vsphere.local
export GOVC_PASSWORD=<password>
export GOVC_INSECURE=true

govc about
```

### Datastore Validation

```bash
# List datastores accessible to the cluster
govc datastore.info <datastore-name>

# Verify free space
govc datastore.info -json <datastore-name> | jq '.datastores[].summary.freeSpace'

# Verify all ESXi hosts can access the datastore
govc host.info -json | jq '.hostSystems[].name'
```

### Storage Policy Validation (SPBM)

```bash
# List storage policies
govc storage.policy.ls

# Verify default policy exists
govc storage.policy.info "vSAN Default Storage Policy"
```

### vSphere Permissions

```bash
# The CSI driver user needs these permissions:
# - Datastore: Allocate space, Browse datastore, Low level file operations
# - VirtualMachine: Config (Add/Remove disk)
# - CNS: Searchable, Manageable

# Verify user permissions
govc permissions.ls -principal '<user>@vsphere.local'
```

### Network Validation

```bash
# vCenter API (443)
oc debug node/<node-name> -- chroot /host nc -zv <vcenter-fqdn> 443

# ESXi management (443) — used by CSI for disk operations
oc debug node/<node-name> -- chroot /host nc -zv <esxi-host-ip> 443
```

### Cloud Provider Configuration

```bash
# Verify vSphere cloud provider is configured
oc get infrastructure cluster -o jsonpath='{.status.platform}'
# Expected: VSphere

# Verify CSI driver pods are running
oc get pods -n openshift-cluster-csi-drivers -l app=vmware-vsphere-csi-driver
```

### VM Hardware Compatibility

```bash
# Verify VM hardware version (via govc)
govc vm.info -json <vm-name> | jq '.virtualMachines[].config.version'
# Expected: vmx-15 or higher

# Verify disk.enableUUID is set
govc vm.info -e <vm-name> | grep disk.enableUUID
# Expected: disk.enableUUID = TRUE
```

## Pre-Installation Summary

| Check | Expected |
|-------|----------|
| vCenter reachable (port 443) | API responds |
| vCenter version 7.0 U3+ | Verified |
| Datastore accessible from all ESXi hosts | Shared storage confirmed |
| Datastore has sufficient free space | Capacity for expected PVs |
| SPBM policies configured | Default policy exists |
| CSI user permissions set | Required permissions granted |
| VM hardware version 15+ | vmx-15 or higher |
| disk.enableUUID=TRUE | Set on all cluster VMs |
