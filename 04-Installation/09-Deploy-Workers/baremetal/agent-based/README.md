# Deploy Workers — Bare Metal (Agent-Based Installer)

## Worker Deployment

Workers were already booted from the agent ISO and are deployed automatically. The Agent-based Installer handles CSR approval.

## Monitor Progress

```bash
export KUBECONFIG=~/agent-install/auth/kubeconfig
oc get nodes -w
```

### Expected Timeline

- Workers join: ~15-20 minutes after control plane is ready

## Adding Workers Post-Installation

To add workers after the initial installation, generate a new worker ISO:

```bash
# Create a new agent-config.yaml with only the new worker hosts
openshift-install agent create image --dir=~/agent-install-day2
```

Boot the new servers from this ISO. They will join the existing cluster.
