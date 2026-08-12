# Complete Installation

This step is platform-agnostic — the same process applies to all infrastructure types.

## Wait for Installation to Complete

```bash
openshift-install wait-for install-complete --dir=~/ocp-install --log-level=info
```

### Expected Timeline

- Operators converge: ~20-30 minutes after workers join
- **Total installation: ~90-120 minutes**

## Access the Cluster

```bash
# Console URL and credentials are printed at install completion
# Credentials are also stored in:
cat ~/ocp-install/auth/kubeadmin-password

# Web Console
echo "https://console-openshift-console.apps.ocp.example.com"

# API
export KUBECONFIG=~/ocp-install/auth/kubeconfig
oc whoami
oc get nodes
```
