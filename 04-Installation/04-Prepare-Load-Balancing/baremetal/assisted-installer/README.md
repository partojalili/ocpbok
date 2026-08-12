# Prepare Load Balancing — Bare Metal (Assisted Installer)

## Built-In Load Balancing

The Assisted Installer deploys the same keepalived + HAProxy stack as IPI. You provide API and Ingress VIPs during cluster configuration, and the cluster manages them automatically.

| VIP | Purpose |
|-----|---------|
| API VIP | Kubernetes API (port 6443) and Machine Config Server (port 22623) |
| Ingress VIP | Application Routes (ports 80 and 443) |

No external load balancer is required.

## User-Managed Load Balancing (Optional)

If you prefer to use an external load balancer, select the **User-Managed Networking** option in the Assisted Installer. In that case, you must configure the load balancer yourself (same as UPI — see `04-Prepare-Load-Balancing/baremetal/upi/`).
