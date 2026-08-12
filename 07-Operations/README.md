# Phase 7 — Day-2 Operations

This is where the POC becomes meaningful.

---

## Monitoring

- Prometheus
- Alertmanager
- Thanos (long-term metric storage and querying)
- Cluster monitoring
- Application monitoring

## Logging

- Cluster logging
- Application logs
- Log forwarding
- External SIEM

## Troubleshooting

Engineers should know how to troubleshoot:

- Node NotReady
- Pod Pending
- CrashLoopBackOff
- ImagePullBackOff
- PVC Pending
- Storage failures
- DNS failures
- Network failures
- Certificate problems
- Operator degradation
- API availability
- Ingress problems

---

## Scaling

### Manual Scaling

- Adding worker nodes
- Removing worker nodes
- MachineSets
- Node labels
- Taints/tolerations

### Automatic Scaling

- HPA (Horizontal Pod Autoscaler)
- VPA (Vertical Pod Autoscaler)
- Cluster Autoscaler
- Machine Autoscaler

And importantly, understand how these interact with:

- GitOps
- Argo CD
- MachineSets
- Cluster Autoscaler

This becomes particularly important when configuration is managed declaratively.

---

## Backup and Disaster Recovery

Understand:

- etcd backup
- Application backup
- PV backup
- Snapshot
- Object storage
- Cross-cluster backup
- Restore
- DR architecture
- RPO
- RTO

```
Production Cluster
       |
       +---- Application Backup
       |
       +---- Persistent Data
       |
       +---- etcd Backup
       |
       v
Backup / DR Location
       |
       v
Secondary OpenShift Cluster
```

**The POC should validate restore, not merely backup.**

---

## Upgrade and Lifecycle Management

The knowledge base should cover:

- OpenShift release lifecycle
- Upgrade channels
- Pre-upgrade validation
- Cluster health
- Operator compatibility
- Application compatibility
- Control-plane upgrade
- Worker upgrade
- MachineConfig
- Rollback considerations
- Post-upgrade validation

**A very important POC question is:** Can the customer operate and upgrade the cluster without requiring the original installation team?

---

## Troubleshooting — Bare Metal

### Common Issues

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

### Log Collection

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

### Support Resources

| Resource | Location |
|----------|----------|
| Red Hat Customer Portal | https://access.redhat.com |
| OpenShift Documentation | https://docs.openshift.com |
| Red Hat Support Cases | https://access.redhat.com/support/cases |
| Must-gather upload | https://access.redhat.com/articles/sosreport |
| Knowledge Base | https://access.redhat.com/search |
