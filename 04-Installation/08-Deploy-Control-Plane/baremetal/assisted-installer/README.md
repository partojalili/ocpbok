# Deploy Control Plane — Bare Metal (Assisted Installer)

## Control Plane Deployment

The Assisted Installer deploys the control plane automatically after the bootstrap phase. All three control plane nodes were already booted from the Discovery ISO during infrastructure preparation.

## Monitor Progress

### SaaS Method

Monitor via the console.redhat.com cluster detail page — each host shows its installation progress.

### On-Premises Method

```bash
# Watch agent status
oc get agents -n ocp-cluster -w

# Watch AgentClusterInstall for overall progress
oc get agentclusterinstall ocp -n ocp-cluster \
  -o jsonpath='{.status.progress.totalPercentage}'
```

### Expected Timeline

- Control plane: ~30-40 minutes after bootstrap phase
