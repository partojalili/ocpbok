# Deploy Control Plane — Bare Metal (UPI)

## Boot Control Plane Nodes

After the bootstrap node is running, boot each control plane node from the RHCOS ISO and apply the master Ignition config:

```bash
# On each control plane server console
sudo coreos-installer install /dev/sda \
  --ignition-url=http://<provisioning-host>/master.ign \
  --insecure-ignition
sudo reboot
```

## Monitor Progress

```bash
export KUBECONFIG=~/ocp-upi/auth/kubeconfig

# Watch nodes
oc get nodes -w

# Watch ClusterOperators
oc get co
```

### Expected Timeline

- Control plane: ~30-40 minutes after bootstrap
