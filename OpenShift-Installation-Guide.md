# Red Hat OpenShift Container Platform Installation Guide

## Bare Metal IPI (Installer-Provisioned Infrastructure)

**Version:** OpenShift 4.16  
**Date:** August 2026  
**Author:** Platform Engineering Team

---

## Table of Contents

1. [Installation Types Overview](#1-installation-types-overview)
2. [Proof of Concept Scope](#2-proof-of-concept-scope)
3. [Customer Success Criteria](#3-customer-success-criteria)
4. [Pre-Installation Validation](#4-pre-installation-validation)
5. [Network Configuration](#5-network-configuration)
6. [Storage Configuration](#6-storage-configuration)
7. [Installation Procedure](#7-installation-procedure)
8. [Post-Installation Validation](#8-post-installation-validation)
9. [Troubleshooting](#9-troubleshooting)

---

## 1. Installation Types Overview

Red Hat OpenShift Container Platform supports multiple installation topologies. The choice depends on infrastructure ownership, automation requirements, and operational model.

### 1.1 Installer-Provisioned Infrastructure (IPI)

- The OpenShift installer provisions and manages the underlying infrastructure automatically.
- Supported platforms: AWS, Azure, GCP, VMware vSphere, Bare Metal, OpenStack.
- The installer handles machine provisioning, OS installation, and cluster bootstrapping.
- Best suited for environments where full automation of the infrastructure lifecycle is desired.
- Bare metal IPI uses the Ironic provisioning service to manage server lifecycle via IPMI/BMC.

### 1.2 User-Provisioned Infrastructure (UPI)

- The customer provisions infrastructure manually or with their own tooling before running the installer.
- The installer generates Ignition configs; the customer is responsible for booting machines with those configs.
- Required when the target platform is not natively supported by IPI or when custom infrastructure constraints exist.
- Provides the most flexibility but requires deeper operational knowledge.

### 1.3 Assisted Installer (AI)

- A SaaS-based or on-premises service (via Assisted Installer Operator) that provides a guided, wizard-driven installation workflow.
- Performs host discovery, validation checks, and generates ISOs for booting target nodes.
- Supports bare metal, vSphere, and Nutanix environments.
- Well suited for smaller deployments, edge, and disconnected environments when paired with the on-premises variant.

### 1.4 Agent-Based Installer

- Embeds the Assisted Installer logic into a bootable ISO, removing the need for an external service.
- Fully disconnected-capable from the start.
- Configuration is defined in YAML manifests (install-config.yaml and agent-config.yaml).
- Ideal for air-gapped environments and edge deployments.

### 1.5 Hosted Control Planes (HCP) / HyperShift

- Control plane components run as pods on a management cluster rather than on dedicated machines.
- Worker nodes are provisioned separately and join the hosted cluster.
- Reduces infrastructure cost for the control plane and enables rapid cluster provisioning.
- Currently supported on AWS, bare metal (with Agent), and KubeVirt.

### 1.6 Single Node OpenShift (SNO)

- Runs the full OpenShift stack (control plane + worker) on a single node.
- Designed for edge, retail, and remote/branch office use cases.
- Can be installed via the Assisted Installer, Agent-based Installer, or IPI.

### 1.7 Comparison Matrix

| Criteria                  | IPI            | UPI            | Assisted       | Agent-Based    | HCP            | SNO            |
|---------------------------|----------------|----------------|----------------|----------------|----------------|----------------|
| Infrastructure automation | Full           | None           | Partial        | Partial        | Full (CP)      | Varies         |
| Disconnected support      | Limited        | Yes            | Yes (on-prem)  | Yes            | Limited        | Yes            |
| Minimum nodes             | 6 (3CP + 3W)  | 6 (3CP + 3W)   | 3 (compact)    | 3 (compact)    | 2 (mgmt + 1W)  | 1              |
| Complexity                | Low            | High           | Low            | Medium         | Medium         | Low            |
| Day-2 machine management  | Automated      | Manual         | Manual         | Manual         | Automated (W)  | N/A            |
| Best for                  | Standard DC    | Custom infra   | Quick start    | Air-gapped     | Multi-tenancy  | Edge           |

**Selected approach for this guide:** Bare Metal IPI, which provides full lifecycle automation using BMC/IPMI and the Metal3 provisioning stack.

---

## 2. Proof of Concept Scope

### 2.1 In Scope

| Item | Description |
|------|-------------|
| Cluster deployment | 3 control plane + 3 worker nodes on bare metal using IPI |
| Networking | OVN-Kubernetes CNI with a single cluster network |
| Storage | OpenShift Data Foundation (ODF) backed by local NVMe/SSD devices |
| Authentication | Integration with customer LDAP/Active Directory via OAuth |
| Ingress | Default Ingress Controller with customer-provided wildcard certificate |
| Monitoring | Default cluster monitoring stack (Prometheus, Alertmanager, Grafana) |
| Logging | OpenShift Logging (Loki-based) forwarding to customer SIEM |
| Registry | Internal image registry backed by ODF PersistentVolume |
| Sample workload | Deploy a sample application to validate the platform end-to-end |
| Backup | OADP (OpenShift API for Data Protection) install and one backup/restore cycle |
| Documentation | Runbook and architecture diagram deliverable |

### 2.2 Out of Scope

| Item | Rationale |
|------|-----------|
| Multi-cluster management (ACM) | Requires separate engagement; not part of initial platform validation |
| Service mesh (Istio/Envoy) | Application-level concern; can be added post-POC |
| Serverless (Knative) | Optional operator; evaluated after core platform is validated |
| GPU/AI/ML workloads | Requires GPU operator and specific hardware; separate POC track |
| Windows container support | Requires WMCO operator and Windows nodes; separate scope |
| Disaster recovery / stretch clusters | Requires multi-site infrastructure; separate design |
| Application migration | Application onboarding is a follow-on phase |
| Production hardening | Security benchmarks (CIS, STIG) applied during production build, not POC |
| Network policy design | Application-specific; customer team defines policies post-deployment |
| Custom Operator development | Customer responsibility; Red Hat provides guidance only |

### 2.3 Assumptions

- Customer provides physical servers meeting minimum hardware specifications.
- BMC/IPMI access is available and credentials are shared with the installation team.
- DNS and DHCP services are preconfigured per the requirements in this document.
- Network team has provisioned the required VLANs and firewall rules.
- A Red Hat pull secret and valid OpenShift subscription are available.
- A provisioning network (L2, no DHCP) is available for bare metal IPI boot.
- NTP is configured and reachable from all nodes.

---

## 3. Customer Success Criteria

The POC is considered successful when the following criteria are met and demonstrated:

### 3.1 Platform Availability

| # | Criterion | Validation Method |
|---|-----------|-------------------|
| 1 | All 3 control plane nodes are Running and Ready | `oc get nodes` shows all control plane nodes Ready |
| 2 | All 3 worker nodes are Running and Ready | `oc get nodes` shows all worker nodes Ready |
| 3 | All ClusterOperators report Available=True, Degraded=False | `oc get co` shows no degraded operators |
| 4 | Cluster version matches target release (4.16.x) | `oc get clusterversion` confirms version |

### 3.2 Networking

| # | Criterion | Validation Method |
|---|-----------|-------------------|
| 5 | Pod-to-pod communication across nodes | Deploy test pods on different nodes, verify connectivity |
| 6 | Service (ClusterIP and NodePort) routing works | Create a Service, curl from another pod |
| 7 | Ingress/Route exposes application externally | Access application via Route URL from outside the cluster |
| 8 | DNS resolution (internal and external) | `nslookup` for cluster services and external domains from within a pod |

### 3.3 Storage

| # | Criterion | Validation Method |
|---|-----------|-------------------|
| 9 | ODF cluster is healthy (all OSDs up) | `oc get storagecluster -n openshift-storage` shows Ready |
| 10 | RWO PVC can be provisioned and bound | Create PVC with ocs-storagecluster-ceph-rbd, verify Bound |
| 11 | RWX PVC can be provisioned and bound | Create PVC with ocs-storagecluster-cephfs, verify Bound |
| 12 | Data persists across pod restarts | Write data, delete pod, verify data exists in new pod |

### 3.4 Authentication and Authorization

| # | Criterion | Validation Method |
|---|-----------|-------------------|
| 13 | LDAP OAuth login works | Log in via `oc login` and web console with LDAP credentials |
| 14 | RBAC groups map correctly | LDAP group members get expected cluster/project roles |
| 15 | kubeadmin is removed after OAuth verification | `oc get secret kubeadmin -n kube-system` returns not found |

### 3.5 Operational Readiness

| # | Criterion | Validation Method |
|---|-----------|-------------------|
| 16 | Monitoring dashboards are accessible | Open Observe section in the web console |
| 17 | Alerts fire correctly | Trigger a test alert (e.g., pod crash loop), verify in Alertmanager |
| 18 | Logging pipeline delivers logs to SIEM | Verify log entries appear in the customer SIEM |
| 19 | OADP backup and restore completes | Back up a namespace, delete it, restore it, verify all resources return |
| 20 | Sample application runs end-to-end | Multi-tier sample app (frontend + backend + database) is accessible |

### 3.6 Sign-Off

| Role | Name | Signature | Date |
|------|------|-----------|------|
| Customer Technical Lead | | | |
| Customer Infrastructure Lead | | | |
| Red Hat Consultant | | | |
| Project Manager | | | |

---

## 4. Pre-Installation Validation

### 4.1 Hardware Requirements

#### Control Plane Nodes (minimum 3)

| Resource | Minimum | Recommended |
|----------|---------|-------------|
| CPU | 8 vCPU / physical cores | 16 cores |
| RAM | 32 GB | 64 GB |
| Boot disk | 120 GB SSD | 250 GB SSD |
| NIC | 2x 10 GbE (provisioning + baremetal) | 2x 25 GbE |
| BMC | IPMI 2.0 / Redfish | Redfish preferred |

#### Worker Nodes (minimum 3)

| Resource | Minimum | Recommended |
|----------|---------|-------------|
| CPU | 8 vCPU / physical cores | 32 cores |
| RAM | 32 GB | 128 GB |
| Boot disk | 120 GB SSD | 250 GB SSD |
| ODF disks | 1x 500 GB NVMe/SSD (raw, unpartitioned) | 2x 1 TB NVMe |
| NIC | 2x 10 GbE | 2x 25 GbE |
| BMC | IPMI 2.0 / Redfish | Redfish preferred |

### 4.2 BMC/IPMI Validation

Run these checks on every node before starting installation:

```bash
# Test IPMI connectivity (replace with actual BMC IPs and credentials)
for BMC_IP in 10.0.10.11 10.0.10.12 10.0.10.13 10.0.10.14 10.0.10.15 10.0.10.16; do
  echo "--- Testing $BMC_IP ---"
  ipmitool -I lanplus -H $BMC_IP -U admin -P <password> chassis status
  ipmitool -I lanplus -H $BMC_IP -U admin -P <password> mc info
done
```

Verify the following for each node:

- [ ] BMC is reachable from the provisioning host.
- [ ] IPMI credentials are valid and have administrator-level access.
- [ ] Boot order can be set to PXE/network first.
- [ ] Virtual media (if using Redfish) is functional.
- [ ] BMC firmware is at a supported version (check vendor compatibility matrix).

### 4.3 DNS Requirements

All DNS records must be resolvable before installation begins. Replace `ocp.example.com` with the actual cluster and base domain.

| Record | Type | Value | Purpose |
|--------|------|-------|---------|
| `api.ocp.example.com` | A | VIP (e.g., 10.0.20.100) | Kubernetes API endpoint |
| `api-int.ocp.example.com` | A | VIP (e.g., 10.0.20.100) | Internal API endpoint |
| `*.apps.ocp.example.com` | A | VIP (e.g., 10.0.20.101) | Wildcard ingress for Routes |
| `master-0.ocp.example.com` | A | 10.0.20.10 | Control plane node 0 |
| `master-1.ocp.example.com` | A | 10.0.20.11 | Control plane node 1 |
| `master-2.ocp.example.com` | A | 10.0.20.12 | Control plane node 2 |
| `worker-0.ocp.example.com` | A | 10.0.20.20 | Worker node 0 |
| `worker-1.ocp.example.com` | A | 10.0.20.21 | Worker node 1 |
| `worker-2.ocp.example.com` | A | 10.0.20.22 | Worker node 2 |

Reverse DNS (PTR) records must also be configured for all node IPs.

**Validation:**

```bash
# Forward lookups
for RECORD in api.ocp.example.com api-int.ocp.example.com test.apps.ocp.example.com \
  master-0.ocp.example.com master-1.ocp.example.com master-2.ocp.example.com \
  worker-0.ocp.example.com worker-1.ocp.example.com worker-2.ocp.example.com; do
  echo "$RECORD -> $(dig +short $RECORD)"
done

# Reverse lookups
for IP in 10.0.20.100 10.0.20.101 10.0.20.10 10.0.20.11 10.0.20.12 \
  10.0.20.20 10.0.20.21 10.0.20.22; do
  echo "$IP -> $(dig +short -x $IP)"
done
```

### 4.4 DHCP Requirements (Provisioning Network)

The provisioning network must **not** have an external DHCP server. The Metal3 provisioning service (Ironic) manages DHCP on this network.

The baremetal network requires either:
- **Option A (Recommended):** Static IPs defined in `install-config.yaml` (no DHCP needed).
- **Option B:** An external DHCP server with reservations for every node MAC address.

### 4.5 NTP Validation

```bash
# Verify NTP is reachable from the provisioning host
chronyc sources -v
# Confirm time is synchronized
chronyc tracking
```

All nodes must be able to reach the NTP server(s) after booting. If the environment is air-gapped, an internal NTP source is mandatory.

### 4.6 Pull Secret and Subscription

- [ ] Download pull secret from [console.redhat.com](https://console.redhat.com/openshift/install/pull-secret).
- [ ] Verify the pull secret is valid JSON: `cat pull-secret.json | jq .`
- [ ] Confirm the OpenShift subscription covers the target node count and support level.

### 4.7 Provisioning Host Requirements

The provisioning host is the machine from which you run the installer. It must have:

| Resource | Requirement |
|----------|-------------|
| OS | RHEL 9.x or Fedora 38+ |
| CPU | 4 cores |
| RAM | 16 GB |
| Disk | 100 GB free |
| NICs | 2 (one on provisioning network, one on baremetal network) |
| Packages | `podman`, `firewalld`, `ipmitool`, `nmcli` |

```bash
# Install required packages
sudo dnf install -y podman firewalld ipmitool NetworkManager

# Download the installer
OCP_VERSION="4.16.10"
curl -L "https://mirror.openshift.com/pub/openshift-v4/clients/ocp/${OCP_VERSION}/openshift-install-linux.tar.gz" \
  -o openshift-install-linux.tar.gz
tar xvf openshift-install-linux.tar.gz
sudo mv openshift-install /usr/local/bin/

# Download oc client
curl -L "https://mirror.openshift.com/pub/openshift-v4/clients/ocp/${OCP_VERSION}/openshift-client-linux.tar.gz" \
  -o openshift-client-linux.tar.gz
tar xvf openshift-client-linux.tar.gz
sudo mv oc kubectl /usr/local/bin/

# Verify
openshift-install version
oc version --client
```

### 4.8 Firewall and Connectivity

The following ports must be open between all cluster nodes and from the provisioning host:

| Source | Destination | Port | Protocol | Purpose |
|--------|-------------|------|----------|---------|
| All nodes | All nodes | 6443 | TCP | Kubernetes API |
| All nodes | All nodes | 22623 | TCP | Machine Config Server |
| All nodes | All nodes | 2379-2380 | TCP | etcd |
| All nodes | All nodes | 9000-9999 | TCP | Host-level services |
| All nodes | All nodes | 10250-10259 | TCP | Kubelet, kube-controller |
| All nodes | All nodes | 30000-32767 | TCP | NodePort services |
| All nodes | All nodes | 4789 | UDP | VXLAN (OVN) |
| All nodes | All nodes | 6081 | UDP | Geneve (OVN) |
| All nodes | All nodes | 9000-9999 | UDP | Host-level services |
| All nodes | All nodes | 500 | UDP | IPsec IKE |
| All nodes | All nodes | 4500 | UDP | IPsec NAT-T |
| Provisioning host | BMC network | 623 | UDP | IPMI |
| Provisioning host | BMC network | 443 | TCP | Redfish |
| All nodes | Internet / Mirror | 443 | TCP | Container image pull |

**Validation:**

```bash
# Test API VIP port readiness (should fail before install, confirms no port conflict)
nc -zv 10.0.20.100 6443

# Test outbound connectivity (if connected)
curl -s https://quay.io/v2/ && echo "Quay reachable" || echo "Quay NOT reachable"
curl -s https://registry.redhat.io/v2/ && echo "Registry reachable" || echo "Registry NOT reachable"
```

---

## 5. Network Configuration

### 5.1 Network Architecture

```
                        +-----------------------+
                        |    External Network   |
                        |    (Corporate LAN)    |
                        +-----------+-----------+
                                    |
                              [ Firewall ]
                                    |
                        +-----------+-----------+
                        |  Baremetal Network     |
                        |  VLAN 20: 10.0.20.0/24|
                        +-----------+-----------+
                                    |
          +------------+------------+------------+------------+
          |            |            |            |            |
     +----+----+  +----+----+  +----+----+  +----+----+  +----+----+
     |Master-0 |  |Master-1 |  |Master-2 |  |Worker-0 |  |Worker-1 | ...
     |.10      |  |.11      |  |.12      |  |.20      |  |.21      |
     +----+----+  +----+----+  +----+----+  +----+----+  +----+----+
          |            |            |            |            |
          +------------+------------+------------+------------+
                                    |
                        +-----------+-----------+
                        | Provisioning Network  |
                        | VLAN 10: 172.22.0.0/24|
                        | (L2 only, no DHCP)    |
                        +-----------------------+

     API VIP:     10.0.20.100
     Ingress VIP: 10.0.20.101
```

### 5.2 Network Definitions

| Network | VLAN | Subnet | Gateway | Purpose |
|---------|------|--------|---------|---------|
| Provisioning | 10 | 172.22.0.0/24 | None | PXE boot, RHCOS image delivery |
| Baremetal | 20 | 10.0.20.0/24 | 10.0.20.1 | Cluster traffic, API, Ingress |
| BMC/IPMI | 30 | 10.0.10.0/24 | 10.0.10.1 | Out-of-band management |
| Storage (optional) | 40 | 10.0.40.0/24 | None | Dedicated ODF replication traffic |

### 5.3 Provisioning Host Network Configuration

```bash
# Configure provisioning NIC (no IP needed; bridge will be created by installer)
nmcli connection add type bridge con-name provisioning ifname provisioning
nmcli connection add type ethernet con-name eno1-prov ifname eno1 master provisioning
nmcli connection modify provisioning ipv4.addresses 172.22.0.1/24 ipv4.method manual
nmcli connection modify provisioning ipv4.dns "" ipv4.gateway ""
nmcli connection up provisioning

# Configure baremetal NIC
nmcli connection add type bridge con-name baremetal ifname baremetal
nmcli connection add type ethernet con-name eno2-bm ifname eno2 master baremetal
nmcli connection modify baremetal ipv4.addresses 10.0.20.2/24 ipv4.method manual \
  ipv4.gateway 10.0.20.1 ipv4.dns "10.0.20.1"
nmcli connection up baremetal
```

### 5.4 OVN-Kubernetes CNI Configuration

OVN-Kubernetes is the default CNI for OpenShift 4.16. The following parameters are set in `install-config.yaml`:

| Parameter | Value |
|-----------|-------|
| Cluster network CIDR | 10.128.0.0/14 |
| Host prefix | /23 (512 pods per node) |
| Service network CIDR | 172.30.0.0/16 |
| Network type | OVNKubernetes |

### 5.5 Load Balancer Considerations

Bare metal IPI uses keepalived and HAProxy to manage the API and Ingress VIPs internally. No external load balancer is required for the POC. For production, consider:

- An external load balancer (F5, HAProxy, MetalLB) for Ingress traffic.
- MetalLB Operator for LoadBalancer-type Services.

---

## 6. Storage Configuration

### 6.1 Storage Strategy

| Storage Type | Technology | Use Cases |
|-------------|-----------|-----------|
| Block (RWO) | ODF (Ceph RBD) | Databases, stateful apps, PV for registry |
| File (RWX) | ODF (CephFS) | Shared content, CMS, logging buffers |
| Object (S3) | ODF (Ceph RGW) or NooBaa | Backups (OADP), Loki log storage |
| Ephemeral | EmptyDir / Local | Build caches, temporary processing |

### 6.2 Disk Preparation for ODF

Each worker node must have at least one raw, unpartitioned, unformatted disk for ODF.

**Validation (run on each worker after nodes are up):**

```bash
# SSH to worker (via debug pod)
oc debug node/worker-0.ocp.example.com -- chroot /host lsblk

# Expected output: sdb (or nvme0n1) with no partitions and no filesystem
# NAME   MAJ:MIN RM   SIZE RO TYPE MOUNTPOINT
# sda      8:0    0   250G  0 disk
# ├─sda1   8:1    0     1G  0 part /boot
# ├─sda2   8:2    0   249G  0 part /sysroot
# sdb      8:16   0   500G  0 disk            <-- ODF disk (raw)
```

If disks have residual data:

```bash
# Wipe disk signatures (destructive!)
oc debug node/worker-0.ocp.example.com -- chroot /host wipefs -a /dev/sdb
```

### 6.3 OpenShift Data Foundation (ODF) Installation

#### Step 1: Install the ODF Operator

```bash
cat <<EOF | oc apply -f -
apiVersion: operators.coreos.com/v1alpha1
kind: Subscription
metadata:
  name: odf-operator
  namespace: openshift-storage
spec:
  channel: stable-4.16
  installPlanApproval: Automatic
  name: odf-operator
  source: redhat-operators
  sourceNamespace: openshift-marketplace
---
apiVersion: v1
kind: Namespace
metadata:
  name: openshift-storage
  labels:
    openshift.io/cluster-monitoring: "true"
EOF
```

#### Step 2: Label Worker Nodes for ODF

```bash
for node in worker-0 worker-1 worker-2; do
  oc label node ${node}.ocp.example.com cluster.ocs.openshift.io/openshift-storage="" --overwrite
done
```

#### Step 3: Discover Local Disks

```bash
cat <<EOF | oc apply -f -
apiVersion: local.storage.openshift.io/v1alpha1
kind: LocalVolumeDiscovery
metadata:
  name: auto-discover-devices
  namespace: openshift-storage
spec:
  nodeSelector:
    nodeSelectorTerms:
      - matchExpressions:
          - key: cluster.ocs.openshift.io/openshift-storage
            operator: Exists
EOF
```

#### Step 4: Create the StorageCluster

```bash
cat <<EOF | oc apply -f -
apiVersion: ocs.openshift.io/v1
kind: StorageCluster
metadata:
  name: ocs-storagecluster
  namespace: openshift-storage
spec:
  manageNodes: false
  monDataDirHostPath: /var/lib/rook
  storageDeviceSets:
    - name: ocs-deviceset
      count: 1
      dataPVCTemplate:
        spec:
          accessModes:
            - ReadWriteOnce
          resources:
            requests:
              storage: 500Gi
          storageClassName: localblock
          volumeMode: Block
      placement: {}
      portable: false
      replica: 3
EOF
```

#### Step 5: Verify ODF Health

```bash
# Wait for StorageCluster to become Ready
oc get storagecluster -n openshift-storage -w

# Verify Ceph health
oc rsh -n openshift-storage $(oc get pod -n openshift-storage -l app=rook-ceph-tools -o name) \
  ceph status

# Verify StorageClasses are created
oc get sc | grep ocs
# Expected:
# ocs-storagecluster-ceph-rbd       openshift-storage.rbd.csi.ceph.com    Delete   VolumeBindingImmediate   true
# ocs-storagecluster-cephfs         openshift-storage.cephfs.csi.ceph.com Delete   VolumeBindingImmediate   true
# openshift-storage.noobaa.io       openshift-storage.noobaa.io/obc       Delete   VolumeBindingImmediate   false
```

### 6.4 Set Default StorageClass

```bash
# Set Ceph RBD as the default StorageClass
oc patch storageclass ocs-storagecluster-ceph-rbd \
  -p '{"metadata": {"annotations": {"storageclass.kubernetes.io/is-default-class": "true"}}}'
```

### 6.5 Configure Internal Registry Storage

```bash
oc patch configs.imageregistry.operator.openshift.io cluster --type merge \
  --patch '{"spec":{"storage":{"pvc":{"claim":""}}, "managementState":"Managed", "replicas":2}}'
```

---

## 7. Installation Procedure

### 7.1 Generate SSH Key

```bash
ssh-keygen -t ed25519 -N '' -f ~/.ssh/ocp-key
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/ocp-key
```

### 7.2 Create install-config.yaml

```yaml
apiVersion: v1
baseDomain: example.com
metadata:
  name: ocp
networking:
  networkType: OVNKubernetes
  clusterNetwork:
    - cidr: 10.128.0.0/14
      hostPrefix: 23
  serviceNetwork:
    - 172.30.0.0/16
  machineNetwork:
    - cidr: 10.0.20.0/24
compute:
  - name: worker
    replicas: 3
controlPlane:
  name: master
  replicas: 3
platform:
  baremetal:
    apiVIPs:
      - 10.0.20.100
    ingressVIPs:
      - 10.0.20.101
    provisioningNetworkInterface: eno1
    provisioningNetworkCIDR: 172.22.0.0/24
    hosts:
      - name: master-0
        role: master
        bmc:
          address: ipmi://10.0.10.11
          username: admin
          password: <REDACTED>
        bootMACAddress: AA:BB:CC:DD:EE:01
        rootDeviceHints:
          deviceName: /dev/sda
      - name: master-1
        role: master
        bmc:
          address: ipmi://10.0.10.12
          username: admin
          password: <REDACTED>
        bootMACAddress: AA:BB:CC:DD:EE:02
        rootDeviceHints:
          deviceName: /dev/sda
      - name: master-2
        role: master
        bmc:
          address: ipmi://10.0.10.13
          username: admin
          password: <REDACTED>
        bootMACAddress: AA:BB:CC:DD:EE:03
        rootDeviceHints:
          deviceName: /dev/sda
      - name: worker-0
        role: worker
        bmc:
          address: ipmi://10.0.10.14
          username: admin
          password: <REDACTED>
        bootMACAddress: AA:BB:CC:DD:EE:04
        rootDeviceHints:
          deviceName: /dev/sda
      - name: worker-1
        role: worker
        bmc:
          address: ipmi://10.0.10.15
          username: admin
          password: <REDACTED>
        bootMACAddress: AA:BB:CC:DD:EE:05
        rootDeviceHints:
          deviceName: /dev/sda
      - name: worker-2
        role: worker
        bmc:
          address: ipmi://10.0.10.16
          username: admin
          password: <REDACTED>
        bootMACAddress: AA:BB:CC:DD:EE:06
        rootDeviceHints:
          deviceName: /dev/sda
pullSecret: '<PASTE_PULL_SECRET_JSON_HERE>'
sshKey: '<PASTE_SSH_PUBLIC_KEY_HERE>'
```

### 7.3 Back Up the Configuration

```bash
mkdir -p ~/ocp-install-backup
cp install-config.yaml ~/ocp-install-backup/install-config.yaml.bak
```

The installer consumes and deletes `install-config.yaml` during the process. Always keep a backup.

### 7.4 Create the Cluster

```bash
mkdir -p ~/ocp-install
cp install-config.yaml ~/ocp-install/

# Create the manifests (optional: review before proceeding)
openshift-install create manifests --dir=~/ocp-install

# Create the cluster
openshift-install create cluster --dir=~/ocp-install --log-level=info
```

Expected timeline:
- Bootstrap: ~20-30 minutes
- Control plane: ~30-40 minutes
- Workers join: ~15-20 minutes
- Operators converge: ~20-30 minutes
- **Total: ~90-120 minutes**

### 7.5 Monitor Installation Progress

In a separate terminal:

```bash
# Watch bootstrap progress
openshift-install wait-for bootstrap-complete --dir=~/ocp-install --log-level=info

# After bootstrap completes, watch installation complete
openshift-install wait-for install-complete --dir=~/ocp-install --log-level=info
```

Additional monitoring:

```bash
# Export kubeconfig
export KUBECONFIG=~/ocp-install/auth/kubeconfig

# Watch nodes
oc get nodes -w

# Watch ClusterOperators
oc get co

# Watch CSRs (workers may need CSR approval)
oc get csr
oc get csr -o go-template='{{range .items}}{{if not .status}}{{.metadata.name}}{{"\n"}}{{end}}{{end}}' \
  | xargs -r oc adm certificate approve
```

### 7.6 Access the Cluster

```bash
# Console URL and credentials are printed at install completion
# Credentials are also stored in:
cat ~/ocp-install/auth/kubeadmin-password

# Web Console
echo "https://console-openshift-console.apps.ocp.example.com"

# API
export KUBECONFIG=~/ocp-install/auth/kubeconfig
oc whoami
oc get nodes
```

---

## 8. Post-Installation Validation

### 8.1 Cluster Health Check Script

```bash
#!/bin/bash
echo "=== Cluster Version ==="
oc get clusterversion

echo ""
echo "=== Nodes ==="
oc get nodes -o wide

echo ""
echo "=== ClusterOperators (Degraded/Unavailable) ==="
oc get co | grep -E 'True.*True|True.*False.*True|False'

echo ""
echo "=== Pending CSRs ==="
oc get csr --no-headers | grep -c Pending

echo ""
echo "=== Pod Health (non-Running pods) ==="
oc get pods -A --field-selector=status.phase!=Running,status.phase!=Succeeded --no-headers | wc -l

echo ""
echo "=== Storage Classes ==="
oc get sc

echo ""
echo "=== PVCs (non-Bound) ==="
oc get pvc -A | grep -v Bound | grep -v NAME

echo ""
echo "=== etcd Health ==="
oc get etcd -o=jsonpath='{range .items[0].status.conditions[?(@.type=="EtcdMembersAvailable")]}{.message}{"\n"}{end}'

echo ""
echo "=== Certificate Expiry (next 30 days) ==="
oc get co kube-apiserver -o json | jq -r '.status.conditions[] | select(.type=="NodeInstallerProgressing") | .message'
```

### 8.2 Smoke Test Deployment

```bash
# Create test project
oc new-project smoke-test

# Deploy a test application
oc new-app --name=hello \
  --image=registry.access.redhat.com/ubi9/httpd-24:latest

# Expose the application
oc expose svc/hello

# Wait for rollout
oc rollout status deployment/hello

# Test access
ROUTE=$(oc get route hello -o jsonpath='{.spec.host}')
curl -s -o /dev/null -w "%{http_code}" http://$ROUTE
# Expected: 200 or 403 (default httpd page)

# Test storage
cat <<EOF | oc apply -f -
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: test-pvc
  namespace: smoke-test
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 5Gi
EOF

oc get pvc test-pvc -w
# Expected: STATUS = Bound

# Cleanup
oc delete project smoke-test
```

---

## 9. Troubleshooting

### 9.1 Common Issues

| Symptom | Likely Cause | Resolution |
|---------|-------------|------------|
| Bootstrap VM does not power on | BMC credentials incorrect or BMC unreachable | Verify IPMI connectivity and credentials |
| Nodes boot but do not get an IP on provisioning network | Provisioning NIC is wrong or external DHCP conflict | Confirm `provisioningNetworkInterface` in install-config; ensure no rogue DHCP |
| Bootstrap completes but workers do not join | CSRs pending approval | Run `oc get csr` and approve pending CSRs |
| ClusterOperator `ingress` is Degraded | DNS wildcard `*.apps` does not resolve | Verify `*.apps.ocp.example.com` DNS record |
| etcd is unhealthy | Clock skew between control plane nodes | Check NTP synchronization on all masters |
| Image pull failures | Pull secret is invalid or network blocks registry access | Validate pull secret; test connectivity to quay.io and registry.redhat.io |
| ODF StorageCluster stuck in Progressing | Raw disks not available or have residual data | Run `wipefs -a` on ODF target disks; verify with `lsblk` |
| Console not accessible | Ingress VIP not reachable or Route DNS misconfigured | Check keepalived pods; verify DNS and firewall rules |

### 9.2 Log Collection

```bash
# Installer logs
cat ~/ocp-install/.openshift_install.log

# Bootstrap logs (if bootstrap has not been torn down)
ssh core@<bootstrap-ip> journalctl -b -f -u release-image.service -u bootkube.service

# Must-gather (comprehensive cluster diagnostic bundle)
oc adm must-gather --dest-dir=./must-gather-$(date +%Y%m%d)

# Must-gather for ODF
oc adm must-gather --image=registry.redhat.io/odf4/ocs-must-gather-rhel9:latest \
  --dest-dir=./must-gather-odf-$(date +%Y%m%d)

# Node-level logs
oc adm node-logs <node-name> -u kubelet
oc adm node-logs <node-name> -u crio
```

### 9.3 Support Resources

| Resource | Location |
|----------|----------|
| Red Hat Customer Portal | https://access.redhat.com |
| OpenShift Documentation | https://docs.openshift.com |
| Red Hat Support Cases | https://access.redhat.com/support/cases |
| Must-gather upload | https://access.redhat.com/articles/sosreport |
| Knowledge Base | https://access.redhat.com/search |

---

**End of Document**
