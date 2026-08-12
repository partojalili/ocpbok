# Proof of Concept Scope — Bare Metal IPI

**Version:** OpenShift 4.22

## In Scope

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

## Out of Scope

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

## Assumptions

- Customer provides physical servers meeting minimum hardware specifications.
- BMC/IPMI access is available and credentials are shared with the installation team.
- DNS and DHCP services are preconfigured per the requirements in this document.
- Network team has provisioned the required VLANs and firewall rules.
- A Red Hat pull secret and valid OpenShift subscription are available.
- A provisioning network (L2, no DHCP) is available for bare metal IPI boot.
- NTP is configured and reachable from all nodes.
