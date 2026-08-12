# Deploy Control Plane — Bare Metal (Agent-Based Installer)

## Control Plane Deployment

The Agent-based Installer deploys the control plane automatically. All nodes were already booted from the agent ISO.

## Monitor Progress

```bash
openshift-install agent wait-for install-complete --dir=~/agent-install --log-level=info
```

After the API becomes available:

```bash
export KUBECONFIG=~/agent-install/auth/kubeconfig
oc get nodes -w
oc get co
```

### Expected Timeline

- Control plane: ~30-40 minutes after bootstrap phase
