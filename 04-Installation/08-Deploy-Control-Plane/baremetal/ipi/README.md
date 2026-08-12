# Deploy Control Plane — Bare Metal

## Monitor Control Plane Deployment

After bootstrap completes, the control plane nodes take over.

```bash
export KUBECONFIG=~/ocp-install/auth/kubeconfig

# Watch nodes come up
oc get nodes -w

# Watch ClusterOperators
oc get co
```

### Expected Timeline

- Control plane: ~30-40 minutes after bootstrap

### What Happens During Control Plane Deployment

1. The three control plane nodes are powered on via BMC.
2. Each node PXE boots and loads RHCOS with the Ignition config.
3. etcd forms a quorum across the three control plane nodes.
4. The Kubernetes API migrates from the bootstrap node to the control plane.
5. ClusterOperators begin deploying (authentication, console, monitoring, etc.).
