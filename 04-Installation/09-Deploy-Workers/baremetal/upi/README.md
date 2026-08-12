# Deploy Workers — Bare Metal (UPI)

## Boot Worker Nodes

After the control plane is running, boot each worker node from the RHCOS ISO and apply the worker Ignition config:

```bash
# On each worker server console
sudo coreos-installer install /dev/sda \
  --ignition-url=http://<provisioning-host>/worker.ign \
  --insecure-ignition
sudo reboot
```

## Approve CSRs

UPI requires **manual CSR approval** for worker nodes:

```bash
export KUBECONFIG=~/ocp-upi/auth/kubeconfig

# List pending CSRs
oc get csr

# Approve all pending CSRs
oc get csr -o go-template='{{range .items}}{{if not .status}}{{.metadata.name}}{{"\n"}}{{end}}{{end}}' \
  | xargs -r oc adm certificate approve
```

> **Note:** Workers generate two rounds of CSRs. Run the approval command again after a few minutes to catch the second round.

## Update Load Balancer

Add the worker node IPs to the Ingress backend pool of your external load balancer.

### Expected Timeline

- Workers join: ~15-20 minutes after CSR approval

## Adding Workers Post-Installation

Repeat the same process: boot from RHCOS ISO with `worker.ign`, approve CSRs, update load balancer.
