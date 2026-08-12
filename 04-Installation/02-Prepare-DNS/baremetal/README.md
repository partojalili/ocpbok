# Prepare DNS — Bare Metal

All DNS records must be resolvable before installation begins. Replace `ocp.example.com` with the actual cluster and base domain.

## Required Records

| Record | Type | Value | Purpose |
|--------|------|-------|---------|
| `api.ocp.example.com` | A | VIP (e.g., 10.0.20.100) | Kubernetes API endpoint |
| `api-int.ocp.example.com` | A | VIP (e.g., 10.0.20.100) | Internal API endpoint |
| `*.apps.ocp.example.com` | A | VIP (e.g., 10.0.20.101) | Wildcard ingress for Routes |
| `master-0.ocp.example.com` | A | 10.0.20.10 | Control plane node 0 |
| `master-1.ocp.example.com` | A | 10.0.20.11 | Control plane node 1 |
| `master-2.ocp.example.com` | A | 10.0.20.12 | Control plane node 2 |
| `worker-0.ocp.example.com` | A | 10.0.20.20 | Worker node 0 |
| `worker-1.ocp.example.com` | A | 10.0.20.21 | Worker node 1 |
| `worker-2.ocp.example.com` | A | 10.0.20.22 | Worker node 2 |

Reverse DNS (PTR) records must also be configured for all node IPs.

## Validation

```bash
# Forward lookups
for RECORD in api.ocp.example.com api-int.ocp.example.com test.apps.ocp.example.com \
  master-0.ocp.example.com master-1.ocp.example.com master-2.ocp.example.com \
  worker-0.ocp.example.com worker-1.ocp.example.com worker-2.ocp.example.com; do
  echo "$RECORD -> $(dig +short $RECORD)"
done

# Reverse lookups
for IP in 10.0.20.100 10.0.20.101 10.0.20.10 10.0.20.11 10.0.20.12 \
  10.0.20.20 10.0.20.21 10.0.20.22; do
  echo "$IP -> $(dig +short -x $IP)"
done
```
