# Prepare Load Balancing — Bare Metal

## IPI Built-In Load Balancing

Bare metal IPI uses keepalived and HAProxy to manage the API and Ingress VIPs internally. No external load balancer is required for the POC.

| VIP | Address | Purpose |
|-----|---------|---------|
| API VIP | 10.0.20.100 | Kubernetes API (port 6443) and Machine Config Server (port 22623) |
| Ingress VIP | 10.0.20.101 | Application Routes (ports 80 and 443) |

These VIPs are managed automatically by the cluster. They float between nodes using VRRP (keepalived).

---

## Production Considerations

For production, consider:

- An external load balancer (F5, HAProxy, MetalLB) for Ingress traffic.
- MetalLB Operator for LoadBalancer-type Services.
