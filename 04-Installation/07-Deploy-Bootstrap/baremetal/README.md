# Deploy Bootstrap — Bare Metal

## Create the Cluster

```bash
openshift-install create cluster --dir=~/ocp-install --log-level=info
```

This single command handles bootstrap, control plane, and worker deployment for IPI.

## Monitor Bootstrap Progress

In a separate terminal:

```bash
openshift-install wait-for bootstrap-complete --dir=~/ocp-install --log-level=info
```

### Expected Timeline

- Bootstrap: ~20-30 minutes

### What Happens During Bootstrap

1. The installer powers on the bootstrap node via BMC (Redfish/IPMI).
2. The bootstrap node PXE boots and loads RHCOS.
3. A temporary control plane starts on the bootstrap node.
4. The temporary etcd and API server come up.
5. The bootstrap node begins provisioning the control plane nodes.

### Troubleshooting Bootstrap

```bash
# Bootstrap logs (if bootstrap has not been torn down)
ssh core@<bootstrap-ip> journalctl -b -f -u release-image.service -u bootkube.service
```
