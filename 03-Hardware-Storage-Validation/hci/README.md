# Hardware and Storage Validation — HCI (Hyperconverged Infrastructure)

## Overview

HCI platforms combine compute, storage, and networking into a single converged stack. OpenShift runs as VMs on the HCI layer, and storage is provided by the HCI platform itself (vSAN, Nutanix Volumes, etc.) or by ODF on top.

## Supported HCI Platforms

| Platform | OpenShift Integration | Installation Methods |
|----------|----------------------|---------------------|
| Nutanix AHV | IPI (GA in 4.22), Assisted Installer | Native Nutanix CSI |
| VMware vSAN | IPI, UPI | vSphere CSI |
| OpenShift Virtualization (KubeVirt) | Hosted Control Planes | ODF / hostpath |

---

## General HCI Validation

### Capacity Planning

HCI nodes share resources between the hypervisor and VMs. Account for overhead:

| Component | Overhead |
|-----------|----------|
| Hypervisor / HCI management | 10-15% CPU, 16-32 GB RAM per host |
| Storage replication (RF2/RF3) | 2x or 3x raw storage capacity |
| HCI network traffic | Dedicated storage network recommended |

- [ ] Total cluster capacity exceeds OpenShift VM requirements + HCI overhead
- [ ] Failure domain planning accounts for HCI node loss (RF2 = survive 1 node, RF3 = survive 2)

### Failure Domains

- [ ] HCI cluster has enough nodes to survive a node failure without impacting OpenShift quorum
- [ ] VM anti-affinity rules configured so control plane VMs run on separate HCI nodes

---

## Nutanix AHV Validation

### Prism Central

- [ ] Prism Central is accessible and running (version compatible with OCP 4.22)
- [ ] Prism Central credentials available with admin permissions

```bash
# Test Prism Central API
curl -sk -u admin:<password> https://<prism-central>:9440/api/nutanix/v3/clusters/list \
  -X POST -H "Content-Type: application/json" -d '{}' | jq '.entities[].spec.name'
```

### Compute

- [ ] Nutanix cluster has sufficient CPU and RAM for all OpenShift VMs
- [ ] AHV hosts are running compatible firmware

### Storage

- [ ] Storage container exists with sufficient capacity
- [ ] Nutanix Volumes CSI driver compatible with OCP 4.22
- [ ] Storage container supports thin provisioning

### Network

- [ ] AHV subnet exists for the OpenShift VM network
- [ ] IPAM or DHCP configured on the subnet
- [ ] DNS and NTP reachable from the subnet

---

## VMware vSAN Validation

Same as VMware vSphere validation (see `vmware/`), plus:

### vSAN-Specific Checks

- [ ] vSAN health is green (no disk or host failures)
- [ ] vSAN storage policy exists for OpenShift VMs
- [ ] vSAN capacity accounts for RF2/RF3 replication overhead
- [ ] Deduplication/compression impact on performance evaluated
- [ ] vSAN network (vmkernel) is healthy

```bash
# Check vSAN health via govc
govc object.collect -s /datacenter/host/cluster vSanHealthSummary
```

---

## Storage Considerations for HCI

| Approach | Description | When to Use |
|----------|-------------|-------------|
| HCI-native storage (Nutanix Volumes, vSAN) | Use the HCI platform's storage via CSI | Default choice — simplest, leverages existing investment |
| ODF on top of HCI | Deploy ODF inside OpenShift using virtual disks from HCI | When you need RWX (CephFS), S3 (RGW), or storage features HCI doesn't provide |
| Mixed | HCI for boot/RWO, ODF for RWX/object | Common in production for flexibility |

---

## Pull Secret Validation

Same as bare metal (see `baremetal/ipi/`).
