# Phase 2 — Hardware Architecture and Design

OpenShift installation differs significantly depending on the underlying infrastructure. This phase covers installation types, hardware architecture, network design, and storage architecture.

---

## A. Installation Types — OpenShift 4.22

Red Hat OpenShift Container Platform supports multiple installation topologies. The choice depends on infrastructure ownership, automation requirements, and operational model.

### Installer-Provisioned Infrastructure (IPI)

- The OpenShift installer provisions and manages the underlying infrastructure automatically.
- Supported platforms: AWS, Azure, GCP, VMware vSphere, Bare Metal, OpenStack, Nutanix.
- The installer handles machine provisioning, OS installation, and cluster bootstrapping.
- Best suited for environments where full automation of the infrastructure lifecycle is desired.
- Bare metal IPI uses the Ironic provisioning service to manage server lifecycle via BMC (Redfish/IPMI).
- As of OpenShift 4.22, Redfish is the preferred and default BMC protocol; IPMI remains supported but is considered legacy.

### User-Provisioned Infrastructure (UPI)

- The customer provisions infrastructure manually or with their own tooling before running the installer.
- The installer generates Ignition configs; the customer is responsible for booting machines with those configs.
- Required when the target platform is not natively supported by IPI or when custom infrastructure constraints exist.
- Provides the most flexibility but requires deeper operational knowledge.

### Assisted Installer (AI)

- A SaaS-based or on-premises service (via Assisted Installer Operator / Infrastructure Operator) that provides a guided, wizard-driven installation workflow.
- Performs host discovery, validation checks, and generates ISOs for booting target nodes.
- Supports bare metal, vSphere, Nutanix, and external platform (platform: none) environments.
- Well suited for smaller deployments, edge, and disconnected environments when paired with the on-premises variant.
- In OpenShift 4.22, the Assisted Installer supports dual-stack networking, OVN-Kubernetes with IPsec, and static IP configurations natively.

### Agent-Based Installer

- Embeds the Assisted Installer logic into a bootable ISO, removing the need for an external service.
- Fully disconnected-capable from the start.
- Configuration is defined in YAML manifests (install-config.yaml and agent-config.yaml).
- Ideal for air-gapped environments and edge deployments.

### Hosted Control Planes (HCP)

- Control plane components run as pods on a management cluster rather than on dedicated machines.
- Worker nodes are provisioned separately and join the hosted cluster.
- Reduces infrastructure cost for the control plane and enables rapid cluster provisioning (sub-10-minute control planes).
- GA on AWS, bare metal (with Agent provider), KubeVirt, and Azure.
- In OpenShift 4.22, HCP is fully integrated into the MCE/ACM operator and supports automated node pool scaling, disconnected deployments, and custom machine CIDR configurations.

### Single Node OpenShift (SNO)

- Runs the full OpenShift stack (control plane + worker) on a single node.
- Designed for edge, retail, and remote/branch office use cases.
- Can be installed via the Assisted Installer, Agent-based Installer, or IPI.

### Installation Types Comparison Matrix

| Criteria                  | IPI            | UPI            | Assisted       | Agent-Based    | HCP            | SNO            |
|---------------------------|----------------|----------------|----------------|----------------|----------------|----------------|
| Infrastructure automation | Full           | None           | Partial        | Partial        | Full (CP)      | Varies         |
| Disconnected support      | Limited        | Yes            | Yes (on-prem)  | Yes            | Limited        | Yes            |
| Minimum nodes             | 6 (3CP + 3W)  | 6 (3CP + 3W)   | 3 (compact)    | 3 (compact)    | 2 (mgmt + 1W)  | 1              |
| Complexity                | Low            | High           | Low            | Medium         | Medium         | Low            |
| Day-2 machine management  | Automated      | Manual         | Manual         | Manual         | Automated (W)  | N/A            |
| Best for                  | Standard DC    | Custom infra   | Quick start    | Air-gapped     | Multi-tenancy  | Edge           |

---

## B. Bare Metal

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

---

## C. VMware

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

## D. Public Cloud

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

## E. HCI (Hyperconverged Infrastructure)

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

## F. Network Design

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

## G. Storage Architecture

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
