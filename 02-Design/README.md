# Phase 2 — Hardware Architecture and Design

OpenShift installation differs significantly depending on the underlying infrastructure. This phase covers hardware architecture, network design, and storage architecture.

---

## A. Bare Metal

Understand:

- Server hardware requirements
- CPU architecture
- Memory
- Local disks
- RAID/HBA configuration
- NICs
- Firmware
- BIOS settings
- Secure Boot
- PXE
- BMC/IPMI/Redfish
- VLAN configuration
- Network bonding
- NIC naming
- UEFI boot
- Local registry requirements

### Installation Approaches

- IPI Bare Metal
- UPI Bare Metal
- Assisted Installer
- Agent-based installation
- User-provisioned infrastructure

You should understand when each method is appropriate.

---

## B. VMware

Knowledge should include:

- vSphere architecture
- vCenter
- ESXi
- VM sizing
- VM templates
- Networking
- Port groups
- VLANs
- Datastore architecture
- VM anti-affinity
- DRS
- HA
- CSI integration
- Storage classes
- Machine API integration
- IPI vs UPI

Also understand the difference between:

- OpenShift managing the infrastructure
- versus OpenShift running on infrastructure managed by VMware

---

## C. Public Cloud

Create separate knowledge areas for:

- AWS
- Azure
- Google Cloud
- IBM Cloud
- Other supported environments

For each cloud understand:

- Supported regions
- Instance types
- Networking (VPC/VNet)
- Subnets
- Security groups
- Load balancers
- IAM
- DNS
- Cloud storage
- Cloud CSI drivers
- Machine sets
- Autoscaling
- Infrastructure provisioning

For example, on Azure you should understand the relationship between:

```
Azure → VNet → Subnet → Load Balancer → VM → OpenShift node
```

---

## D. HCI (Hyperconverged Infrastructure)

For Hyperconverged Infrastructure, understand:

- Compute
- Storage
- Networking
- Failure domains
- Replication
- Storage policies
- Capacity planning
- Performance impact
- Node failure behavior

Examples include environments based on:

- OpenShift Virtualization
- VMware
- Nutanix
- Other supported HCI platforms

---

## E. Network Design

This is one of the most important parts of the knowledge base.

### Required Network Components

- DNS
- DHCP
- NTP
- Load balancer
- API endpoint
- Ingress
- Egress
- Firewall
- Proxy
- VLANs
- Routing
- MTU

### DNS

Understand:

- API DNS
- API internal DNS
- Apps wildcard DNS
- Reverse DNS
- PTR records

Typical records:

```
api.cluster.example.com
api-int.cluster.example.com
*.apps.cluster.example.com
```

### Load Balancing

Understand:

- API load balancing
- Ingress load balancing
- Internal vs external load balancers
- Health checks
- HAProxy
- F5
- Cloud-native load balancers

### Network Architecture

Understand OpenShift's:

- OVN-Kubernetes
- Cluster network
- Service network
- Machine network
- Pod network
- NetworkPolicy
- Egress traffic
- Multus
- Secondary networks
- SR-IOV

---

## F. Storage Architecture

This should be a standalone body of knowledge.

The first question should not be: *"Which storage product should we use?"*

It should be: **"What storage behavior does the application require?"**

### Storage Types

| Type | Typical Use Cases |
|------|------------------|
| Block | Databases, VM disks, applications requiring low latency, RWO workloads |
| File | Shared filesystem, RWX workloads, applications requiring multiple pods to access the same filesystem |
| Object | Data lakes, backups, large unstructured datasets, S3-compatible applications |

### Storage Decision Framework

For every application, collect:

| Requirement | Question |
|-------------|----------|
| Capacity | How much storage? |
| Performance | What IOPS/throughput? |
| Latency | What latency is acceptable? |
| Access mode | RWO/RWX/RWOP? |
| Protocol | Block/File/Object? |
| Availability | What happens if storage fails? |
| Replication | Who provides replication? |
| Backup | How is data backed up? |
| Encryption | At rest/in transit? |
| Expansion | Can capacity be expanded? |
| Snapshot | Are snapshots required? |
| DR | How is data replicated to another site? |

Then map the requirement to the appropriate storage solution.

### OpenShift Storage Knowledge

Understand:

- CSI
- StorageClass
- PersistentVolume
- PersistentVolumeClaim
- VolumeSnapshot
- VolumeSnapshotClass
- Access modes
- Dynamic provisioning
- Static provisioning
- Reclaim policy
- Expansion
- Topology
- Encryption
- Snapshot
- Backup
- CSI drivers

And, where applicable:

- OpenShift Data Foundation
- External enterprise storage
- Cloud storage
- NFS
- SAN
- Ceph
- S3/object storage

**A critical concept is:** OpenShift does not automatically make every storage system suitable for every workload.
