# Hardware and Storage Validation — VMware vSphere

## vSphere Infrastructure Validation

### vCenter

- [ ] vCenter Server is accessible and running (version 7.0 U3+ or 8.0+)
- [ ] vCenter credentials with required permissions available
- [ ] vCenter API accessible from the provisioning host

```bash
# Test vCenter API connectivity
curl -sk https://<vcenter-fqdn>/api
```

### ESXi Hosts

- [ ] Minimum 3 ESXi hosts for control plane HA
- [ ] ESXi version compatible with vCenter version
- [ ] ESXi hosts have sufficient capacity

| Resource | Per Control Plane VM | Per Worker VM |
|----------|---------------------|---------------|
| vCPU | 8 | 8 |
| RAM | 32 GB | 32 GB |
| Disk | 120 GB | 120 GB + ODF disks |

### Cluster Configuration

- [ ] DRS enabled (required for IPI)
- [ ] HA enabled (recommended)
- [ ] VM anti-affinity rules can be created
- [ ] Resource pool created (optional but recommended)

---

## Storage Validation

### Datastores

- [ ] Datastore accessible by all ESXi hosts
- [ ] Sufficient free space for all VMs (control plane + workers + ODF)
- [ ] Datastore type validated (VMFS, vSAN, NFS)

```
Minimum storage calculation:
  3 CP × 120 GB = 360 GB
  3 Workers × 120 GB = 360 GB
  3 Workers × 500 GB ODF = 1,500 GB (if using ODF with virtual disks)
  Total: ~2.2 TB minimum
```

### vSphere CSI Driver

- [ ] vSphere CSI driver is compatible with OCP 4.22
- [ ] Storage policy exists for dynamic provisioning
- [ ] Thin provisioning enabled (recommended)

---

## Network Validation

### Port Groups

- [ ] VM network port group exists
- [ ] VLAN configured correctly on the port group
- [ ] Port group accessible by all ESXi hosts

### IP Allocation

- [ ] DHCP available on the VM network (for IPI), or static IPs planned (for UPI)
- [ ] API VIP and Ingress VIP are unused
- [ ] DNS records created (api, api-int, *.apps, all nodes)

### Firewall

- [ ] vCenter API (443) accessible from provisioning host
- [ ] ESXi hosts accessible on port 443 and 902
- [ ] Cluster nodes can reach DNS, NTP, and image registries

---

## Permissions Validation

The vCenter user account requires these permissions:

- Virtual Machine: all
- Network: assign
- Datastore: allocate space, browse, low-level file operations
- Resource: assign VM to resource pool
- Folder: create, delete
- vApp: import
- Profile-driven storage: all (for CSI)

```bash
# Verify vCenter credentials and permissions
govc about -u "user:pass@vcenter.example.com"
govc ls /datacenter/vm/
```

---

## Pull Secret Validation

Same as bare metal (see `baremetal/ipi/`).
