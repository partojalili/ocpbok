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
