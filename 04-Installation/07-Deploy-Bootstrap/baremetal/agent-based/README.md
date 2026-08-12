# Deploy Bootstrap — Bare Metal (Agent-Based Installer)

## How Bootstrap Works

The Agent-based Installer uses the **rendezvous node** (specified by `rendezvousIP` in `agent-config.yaml`) to coordinate the installation. There is no dedicated bootstrap node — the rendezvous node is one of the control plane nodes.

## Start Installation

1. Boot all servers from the agent ISO (`agent.x86_64.iso`).
2. The rendezvous node starts the Assisted Installer service locally.
3. Other nodes register with the rendezvous node.
4. Installation begins automatically once all nodes are discovered and validated.

## Monitor Bootstrap Progress

```bash
# Monitor from the provisioning host (or any machine with the kubeconfig)
openshift-install agent wait-for bootstrap-complete --dir=~/agent-install --log-level=info
```

### Expected Timeline

- Bootstrap phase: ~30-40 minutes
