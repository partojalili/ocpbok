# Phase 4 — Cluster Installation

The knowledge should walk through the installation itself.

## Installation Steps

1. Prepare infrastructure
2. Prepare DNS
3. Prepare networking
4. Prepare load balancing
5. Prepare storage
6. Generate installation configuration
7. Deploy bootstrap
8. Deploy control plane
9. Deploy workers
10. Complete installation
11. Remove bootstrap infrastructure
12. Validate cluster

## Post-Install Verification

```bash
oc get nodes
oc get clusterversion
oc get clusteroperators
oc get pods -A
```

**The important point is not simply knowing the commands.** The engineer should understand what each component is doing and what failure looks like.
