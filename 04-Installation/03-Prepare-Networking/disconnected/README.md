# Prepare Networking — Disconnected / Air-Gapped

## Network Constraints

In a disconnected environment, nodes have **no internet access**. All traffic stays within the local network.

## Firewall Rules — What Changes

### Not Required (blocked by design)

| Destination | Port | Notes |
|-------------|------|-------|
| quay.io | 443 | Replaced by local mirror registry |
| registry.redhat.io | 443 | Replaced by local mirror registry |
| cdn.redhat.com | 443 | Not needed |
| console.redhat.com | 443 | Not needed |
| sso.redhat.com | 443 | Not needed |

### Required

| Source | Destination | Port | Purpose |
|--------|-------------|------|---------|
| All nodes | Mirror registry | 443 or 8443 | Image pulls |
| All nodes | DNS server | 53 | Name resolution |
| All nodes | NTP server | 123 | Time synchronization |
| All nodes | All nodes | (same as connected) | Cluster-internal traffic |

## Proxy Considerations

If the environment uses a proxy for partial connectivity (semi-disconnected), configure the proxy in `install-config.yaml`:

```yaml
proxy:
  httpProxy: http://proxy.example.com:3128
  httpsProxy: http://proxy.example.com:3128
  noProxy: .example.com,10.0.0.0/8,172.16.0.0/12,192.168.0.0/16,.svc,.cluster.local
```

For fully air-gapped environments, no proxy configuration is needed.

## NTP

An internal NTP source is mandatory. External NTP servers (e.g., pool.ntp.org) are unreachable. Configure chrony to use a local NTP server:

```yaml
# MachineConfig for chrony (embed in install manifests)
apiVersion: machineconfiguration.openshift.io/v1
kind: MachineConfig
metadata:
  name: 99-worker-chrony
  labels:
    machineconfiguration.openshift.io/role: worker
spec:
  config:
    ignition:
      version: 3.4.0
    storage:
      files:
        - path: /etc/chrony.conf
          mode: 0644
          contents:
            source: data:text/plain;charset=utf-8;base64,<BASE64_ENCODED_CHRONY_CONF>
```
