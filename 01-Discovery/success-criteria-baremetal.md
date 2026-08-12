# Customer Success Criteria — Bare Metal IPI

The POC is considered successful when the following criteria are met and demonstrated.

## Platform Availability

| # | Criterion | Validation Method |
|---|-----------|-------------------|
| 1 | All 3 control plane nodes are Running and Ready | `oc get nodes` shows all control plane nodes Ready |
| 2 | All 3 worker nodes are Running and Ready | `oc get nodes` shows all worker nodes Ready |
| 3 | All ClusterOperators report Available=True, Degraded=False | `oc get co` shows no degraded operators |
| 4 | Cluster version matches target release (4.22.x) | `oc get clusterversion` confirms version |

## Networking

| # | Criterion | Validation Method |
|---|-----------|-------------------|
| 5 | Pod-to-pod communication across nodes | Deploy test pods on different nodes, verify connectivity |
| 6 | Service (ClusterIP and NodePort) routing works | Create a Service, curl from another pod |
| 7 | Ingress/Route exposes application externally | Access application via Route URL from outside the cluster |
| 8 | DNS resolution (internal and external) | `nslookup` for cluster services and external domains from within a pod |

## Storage

| # | Criterion | Validation Method |
|---|-----------|-------------------|
| 9 | ODF cluster is healthy (all OSDs up) | `oc get storagecluster -n openshift-storage` shows Ready |
| 10 | RWO PVC can be provisioned and bound | Create PVC with ocs-storagecluster-ceph-rbd, verify Bound |
| 11 | RWX PVC can be provisioned and bound | Create PVC with ocs-storagecluster-cephfs, verify Bound |
| 12 | Data persists across pod restarts | Write data, delete pod, verify data exists in new pod |

## Authentication and Authorization

| # | Criterion | Validation Method |
|---|-----------|-------------------|
| 13 | LDAP OAuth login works | Log in via `oc login` and web console with LDAP credentials |
| 14 | RBAC groups map correctly | LDAP group members get expected cluster/project roles |
| 15 | kubeadmin is removed after OAuth verification | `oc get secret kubeadmin -n kube-system` returns not found |

## Operational Readiness

| # | Criterion | Validation Method |
|---|-----------|-------------------|
| 16 | Monitoring dashboards are accessible | Open Observe section in the web console |
| 17 | Alerts fire correctly | Trigger a test alert (e.g., pod crash loop), verify in Alertmanager |
| 18 | Logging pipeline delivers logs to SIEM | Verify log entries appear in the customer SIEM |
| 19 | OADP backup and restore completes | Back up a namespace, delete it, restore it, verify all resources return |
| 20 | Sample application runs end-to-end | Multi-tier sample app (frontend + backend + database) is accessible |

## Sign-Off

| Role | Name | Signature | Date |
|------|------|-----------|------|
| Customer Technical Lead | | | |
| Customer Infrastructure Lead | | | |
| Red Hat Consultant | | | |
| Project Manager | | | |
