# Deploy Workers — Bare Metal

## Monitor Worker Deployment

```bash
export KUBECONFIG=~/ocp-install/auth/kubeconfig

# Watch nodes
oc get nodes -w

# Watch CSRs (workers may need CSR approval)
oc get csr
```

### Expected Timeline

- Workers join: ~15-20 minutes after control plane is ready

### CSR Approval

Worker nodes generate Certificate Signing Requests that may need manual approval:

```bash
# List pending CSRs
oc get csr

# Approve all pending CSRs
oc get csr -o go-template='{{range .items}}{{if not .status}}{{.metadata.name}}{{"\n"}}{{end}}{{end}}' \
  | xargs -r oc adm certificate approve
```

### What Happens During Worker Deployment

1. Worker nodes are powered on via BMC.
2. Each worker PXE boots and loads RHCOS with the worker Ignition config.
3. Workers generate CSRs to join the cluster.
4. After CSR approval, workers become Ready.
5. Workloads can be scheduled on worker nodes.
