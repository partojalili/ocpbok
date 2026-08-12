# Phase 1 — POC Discovery and Requirements

Before touching the cluster, collect the requirements.

## Business Requirements

- What problem is OpenShift expected to solve?
- POC objectives and success criteria
- Applications/workloads to be tested
- Expected number of users
- Availability requirements
- Disaster recovery requirements
- Security/compliance requirements
- Expected growth
- Production timeline

## OpenShift Requirements

Determine:

- OpenShift version (4.22)
- Number of clusters
- Cluster type
- Number of control-plane nodes
- Number of worker nodes
- Infrastructure nodes, if required
- Architecture: x86, Power, Z, ARM, etc.
- Connected vs. disconnected
- Single-site vs. multi-site
- Air-gapped requirements
- Proxy requirements
- Identity provider
- DNS
- NTP
- Load balancing
- Registry requirements

## POC Success Criteria

Define measurable criteria before installation.

| Area | Success Criteria |
|------|-----------------|
| Installation | Cluster successfully installed |
| Networking | All required application/network flows work |
| Storage | Required storage classes available |
| Security | Authentication/RBAC requirements validated |
| Applications | Target applications deployed successfully |
| Performance | Workloads meet agreed performance requirements |
| Availability | Node/component failure scenarios tested |
| Upgrade | Upgrade procedure validated, if applicable |
| Operations | Monitoring/logging validated |
| Automation | GitOps/automation validated |
| Documentation | Installation and operational documentation complete |

---

## Proof of Concept Scope — Bare Metal IPI

**Version:** OpenShift 4.22

### In Scope

| Item | Description |
|------|-------------|
| Cluster deployment | 3 control plane + 3 worker nodes on bare metal using IPI |
| Networking | OVN-Kubernetes CNI with a single cluster network |
| Storage | OpenShift Data Foundation (ODF) backed by local NVMe/SSD devices |
| Authentication | Integration with customer LDAP/Active Directory via OAuth |
| Ingress | Default Ingress Controller with customer-provided wildcard certificate |
| Monitoring | Default cluster monitoring stack (Prometheus, Alertmanager, cluster observability dashboards) |
| Logging | Cluster Logging via ClusterLogForwarder (Vector-based collector) forwarding to customer SIEM |
| Registry | Internal image registry backed by ODF PersistentVolume |
| Sample workload | Deploy a sample application to validate the platform end-to-end |
| Backup | OADP (OpenShift API for Data Protection) install and one backup/restore cycle |
| Documentation | Runbook and architecture diagram deliverable |

### Out of Scope

| Item | Rationale |
|------|-----------|
| Multi-cluster management (ACM) | Requires separate engagement; not part of initial platform validation |
| Service mesh (Istio/Sail Operator) | Application-level concern; can be added post-POC |
| Serverless (Knative) | Optional operator; evaluated after core platform is validated |
| GPU/AI/ML workloads | Requires GPU operator and specific hardware; separate POC track |
| Windows container support | Requires WMCO operator and Windows nodes; separate scope |
| Disaster recovery / stretch clusters | Requires multi-site infrastructure; separate design |
| Application migration | Application onboarding is a follow-on phase |
| Production hardening | Security benchmarks (CIS, STIG) applied during production build, not POC |
| Network policy design | Application-specific; customer team defines policies post-deployment |
| Custom Operator development | Customer responsibility; Red Hat provides guidance only |

### Assumptions

- Customer provides physical servers meeting minimum hardware specifications.
- BMC/IPMI access is available and credentials are shared with the installation team.
- DNS and DHCP services are preconfigured per the requirements in this document.
- Network team has provisioned the required VLANs and firewall rules.
- A Red Hat pull secret and valid OpenShift subscription are available.
- A provisioning network (L2, no DHCP) is available for bare metal IPI boot.
- NTP is configured and reachable from all nodes.

---

## Customer Success Criteria — Bare Metal IPI

The POC is considered successful when the following criteria are met and demonstrated.

### Platform Availability

| # | Criterion | Validation Method |
|---|-----------|-------------------|
| 1 | All 3 control plane nodes are Running and Ready | `oc get nodes` shows all control plane nodes Ready |
| 2 | All 3 worker nodes are Running and Ready | `oc get nodes` shows all worker nodes Ready |
| 3 | All ClusterOperators report Available=True, Degraded=False | `oc get co` shows no degraded operators |
| 4 | Cluster version matches target release (4.22.x) | `oc get clusterversion` confirms version |

### Networking

| # | Criterion | Validation Method |
|---|-----------|-------------------|
| 5 | Pod-to-pod communication across nodes | Deploy test pods on different nodes, verify connectivity |
| 6 | Service (ClusterIP and NodePort) routing works | Create a Service, curl from another pod |
| 7 | Ingress/Route exposes application externally | Access application via Route URL from outside the cluster |
| 8 | DNS resolution (internal and external) | `nslookup` for cluster services and external domains from within a pod |

### Storage

| # | Criterion | Validation Method |
|---|-----------|-------------------|
| 9 | ODF cluster is healthy (all OSDs up) | `oc get storagecluster -n openshift-storage` shows Ready |
| 10 | RWO PVC can be provisioned and bound | Create PVC with ocs-storagecluster-ceph-rbd, verify Bound |
| 11 | RWX PVC can be provisioned and bound | Create PVC with ocs-storagecluster-cephfs, verify Bound |
| 12 | Data persists across pod restarts | Write data, delete pod, verify data exists in new pod |

### Authentication and Authorization

| # | Criterion | Validation Method |
|---|-----------|-------------------|
| 13 | LDAP OAuth login works | Log in via `oc login` and web console with LDAP credentials |
| 14 | RBAC groups map correctly | LDAP group members get expected cluster/project roles |
| 15 | kubeadmin is removed after OAuth verification | `oc get secret kubeadmin -n kube-system` returns not found |

### Operational Readiness

| # | Criterion | Validation Method |
|---|-----------|-------------------|
| 16 | Monitoring dashboards are accessible | Open Observe section in the web console |
| 17 | Alerts fire correctly | Trigger a test alert (e.g., pod crash loop), verify in Alertmanager |
| 18 | Logging pipeline delivers logs to SIEM | Verify log entries appear in the customer SIEM |
| 19 | OADP backup and restore completes | Back up a namespace, delete it, restore it, verify all resources return |
| 20 | Sample application runs end-to-end | Multi-tier sample app (frontend + backend + database) is accessible |

### Sign-Off

| Role | Name | Signature | Date |
|------|------|-----------|------|
| Customer Technical Lead | | | |
| Customer Infrastructure Lead | | | |
| Red Hat Consultant | | | |
| Project Manager | | | |
