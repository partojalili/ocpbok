# Deploy Bootstrap — Bare Metal (Assisted Installer)

## How Bootstrap Works

The Assisted Installer handles bootstrap differently from IPI and UPI. There is **no dedicated bootstrap node**. The bootstrap process runs on one of the control plane nodes (the one selected as the bootstrap host by the installer).

## Start Installation

### SaaS Method

1. In the console.redhat.com wizard, verify all hosts show **Ready**.
2. Click **Install cluster**.
3. The installation begins automatically.

### On-Premises Method

The installation starts automatically once the required number of agents are approved and bound:

```bash
# Approve discovered agents
oc get agents -n ocp-cluster
oc patch agent <agent-name> -n ocp-cluster --type merge \
  -p '{"spec":{"approved":true}}'
```

## Monitor Bootstrap Progress

### SaaS Method

Monitor via the web console at console.redhat.com — progress is shown in real-time.

### On-Premises Method

```bash
# Watch AgentClusterInstall status
oc get agentclusterinstall ocp -n ocp-cluster -w

# Detailed status
oc get agentclusterinstall ocp -n ocp-cluster \
  -o jsonpath='{.status.conditions[?(@.type=="Completed")].message}'
```

### Expected Timeline

- Bootstrap phase: ~30-40 minutes (runs on a control plane node, no separate bootstrap machine)
