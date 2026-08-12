# Prepare Load Balancing — Bare Metal (Agent-Based Installer)

## Built-In Load Balancing

The Agent-based Installer deploys keepalived + HAProxy, the same as IPI. API and Ingress VIPs are defined in `install-config.yaml`:

```yaml
platform:
  baremetal:
    apiVIPs:
      - 10.0.20.100
    ingressVIPs:
      - 10.0.20.101
```

No external load balancer is required.

## User-Managed Networking (Optional)

If using `platform: none` or user-managed networking, you must configure an external load balancer yourself (same as UPI — see `04-Prepare-Load-Balancing/baremetal/upi/`).
