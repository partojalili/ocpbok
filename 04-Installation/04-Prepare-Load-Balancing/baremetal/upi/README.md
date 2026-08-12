# Prepare Load Balancing — Bare Metal (UPI)

## External Load Balancer Required

UPI does **not** deploy keepalived or HAProxy. You must configure an external load balancer before installation.

## API Load Balancer

| Frontend | Backend Pool | Health Check |
|----------|-------------|--------------|
| `api.ocp.example.com:6443` | All control plane nodes, port 6443 | HTTPS `/readyz` on port 6443 |
| `api.ocp.example.com:22623` | All control plane nodes, port 22623 | HTTPS `/healthz` on port 22623 |

> During bootstrap, include the bootstrap node in the API backend pool. Remove it after bootstrap completes.

## Ingress Load Balancer

| Frontend | Backend Pool | Health Check |
|----------|-------------|--------------|
| `*.apps.ocp.example.com:443` | All worker nodes (or infra nodes), port 443 | TCP check on port 443 |
| `*.apps.ocp.example.com:80` | All worker nodes (or infra nodes), port 80 | TCP check on port 80 |

## HAProxy Example Configuration

```
frontend api
    bind *:6443
    default_backend api-backend

backend api-backend
    balance roundrobin
    option httpchk GET /readyz HTTP/1.0
    option log-health-checks
    server bootstrap 10.0.20.5:6443 check verify none
    server master-0  10.0.20.10:6443 check verify none
    server master-1  10.0.20.11:6443 check verify none
    server master-2  10.0.20.12:6443 check verify none

frontend machine-config
    bind *:22623
    default_backend machine-config-backend

backend machine-config-backend
    balance roundrobin
    server bootstrap 10.0.20.5:22623 check verify none
    server master-0  10.0.20.10:22623 check verify none
    server master-1  10.0.20.11:22623 check verify none
    server master-2  10.0.20.12:22623 check verify none

frontend ingress-https
    bind *:443
    default_backend ingress-https-backend

backend ingress-https-backend
    balance roundrobin
    server worker-0 10.0.20.20:443 check
    server worker-1 10.0.20.21:443 check
    server worker-2 10.0.20.22:443 check

frontend ingress-http
    bind *:80
    default_backend ingress-http-backend

backend ingress-http-backend
    balance roundrobin
    server worker-0 10.0.20.20:80 check
    server worker-1 10.0.20.21:80 check
    server worker-2 10.0.20.22:80 check
```
