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
