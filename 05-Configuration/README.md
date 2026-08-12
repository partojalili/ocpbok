# Phase 5 — Day-1 Configuration

After installation, configure the cluster for operational readiness.

---

## Identity

- Identity provider
- LDAP
- Active Directory
- OAuth
- RBAC
- Groups

## Security

- SCC (Security Context Constraints)
- RBAC
- NetworkPolicy
- Secrets
- Certificates
- Encryption
- Compliance

## Networking

- Ingress
- Routes
- DNS
- Egress
- Network policies
- Load balancers

## Storage

- StorageClasses
- Default StorageClass
- PVC testing
- Snapshot testing
- Expansion testing

## Registry

- Internal image registry
- External registry
- Mirroring
- Image signatures
- Image policies

---

## Automation and GitOps

Include:

- Git
- GitOps
- Argo CD
- Application deployment
- Cluster configuration
- Policy management
- Secrets management
- External Secrets
- RBAC
- Application lifecycle

You should distinguish **declarative configuration** from **runtime/autonomous behavior**.

For example, understand what happens when:

```
Git configuration
       ↓
Argo CD
       ↓
Kubernetes resource
       ↓
HPA/VPA/Operator
       ↓
Runtime changes
```

This is an important area for real-world POC discussions.
