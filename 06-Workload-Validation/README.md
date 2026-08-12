# Phase 6 — Workload Validation

## Application Migration

The knowledge should cover:

- Containerization
- Image creation
- Image registry
- Deployment
- Service
- Route
- ConfigMap
- Secret
- PVC
- SecurityContext
- SCC
- NetworkPolicy
- Operators

### Migration Patterns

- VM → Container
- VM → OpenShift Virtualization
- Existing Kubernetes → OpenShift
- Traditional application → Cloud-native application

---

## Performance Validation

Don't just say: *"The application works."*

**Measure:**

- CPU
- Memory
- IOPS
- Throughput
- Network throughput
- Latency
- Pod startup time
- Application response time
- Scaling time
- Storage latency

**Establish:**

```
Baseline → Test → Results → Acceptance criteria
```

---

## High Availability Testing

POC testing should deliberately introduce failures.

### Test Scenarios

- Control-plane node failure
- Worker node failure
- Network failure
- Storage failure
- Application failure
- Load balancer failure
- DNS failure
- Power failure, where applicable

### Documentation Template

| Scenario | Expected Behavior | Actual Behavior | Recovery Time | Lessons Learned |
|----------|-------------------|-----------------|---------------|-----------------|
| | | | | |
