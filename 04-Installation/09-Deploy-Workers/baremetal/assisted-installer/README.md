# Deploy Workers — Bare Metal (Assisted Installer)

## Worker Deployment

Workers were already booted from the Discovery ISO and are deployed automatically as part of the installation. The Assisted Installer handles CSR approval automatically.

## Monitor Progress

### SaaS Method

Monitor each host's progress in the console.redhat.com cluster detail page.

### On-Premises Method

```bash
oc get agents -n ocp-cluster -w
```

After the cluster API is available:

```bash
export KUBECONFIG=~/ocp-install/auth/kubeconfig
oc get nodes -w
```

### Expected Timeline

- Workers join: ~15-20 minutes after control plane is ready

## Adding Workers Post-Installation

To add more workers after the cluster is installed:

1. Boot new servers from the same Discovery ISO.
2. The new hosts appear in the inventory.
3. Approve the agents and they join the cluster automatically.
