# F5 BIG-IP

**Vendor:** F5 Networks  
**Integration:** F5 Container Ingress Services (CIS)  
**Purpose:** Enterprise load balancing for API, Ingress, and application services

## Overview

F5 BIG-IP provides hardware and virtual load balancing for OpenShift clusters. It can serve as the external load balancer for API/Ingress endpoints and as an application-level load balancer via the F5 Container Ingress Services (CIS) controller.

## Use Cases

| Use Case | Description |
|----------|-------------|
| API/Ingress LB (pre-install) | External LB for `api.*` and `*.apps.*` endpoints |
| Application LB (post-install) | F5 CIS watches OpenShift Routes/Services and configures BIG-IP automatically |
| WAF | BIG-IP Advanced WAF for application security |
| DNS/GSLB | BIG-IP DNS for multi-site traffic management |

## Pre-Install: F5 as API/Ingress Load Balancer

### Virtual Server Configuration (BIG-IP)

```
# API Virtual Server
ltm virtual ocp-api {
    destination 10.0.10.100:6443
    pool ocp-api-pool
    profiles {
        tcp { }
    }
}

ltm pool ocp-api-pool {
    monitor https_head_f5
    members {
        10.0.10.10:6443 { }
        10.0.10.11:6443 { }
        10.0.10.12:6443 { }
    }
}

# Ingress Virtual Server (HTTPS)
ltm virtual ocp-ingress-https {
    destination 10.0.10.101:443
    pool ocp-ingress-https-pool
    profiles {
        tcp { }
    }
}

ltm pool ocp-ingress-https-pool {
    monitor tcp
    members {
        10.0.10.20:443 { }
        10.0.10.21:443 { }
        10.0.10.22:443 { }
    }
}
```

### Health Monitors

```
# API health monitor
ltm monitor https ocp-api-monitor {
    send "GET /readyz HTTP/1.1\r\nHost: api.ocp.example.com\r\n\r\n"
    recv "ok"
    interval 5
    timeout 16
}
```

## Post-Install: F5 Container Ingress Services (CIS)

### Install CIS Controller

```bash
# Create BIG-IP credentials secret
oc create secret generic bigip-credentials \
  --from-literal=username=admin \
  --from-literal=password=<password> \
  -n kube-system

# Deploy CIS controller
# (Via Helm or operator — refer to F5 documentation for the latest chart)
helm repo add f5-stable https://f5networks.github.io/charts/stable
helm install f5-cis f5-stable/f5-bigip-ctlr \
  --namespace kube-system \
  --set bigip_login_secret=bigip-credentials \
  --set args.bigip_url=<bigip-mgmt-ip> \
  --set args.bigip_partition=ocp \
  --set args.openshift_sdn_name=/Common/ocp-vxlan \
  --set args.pool_member_type=cluster
```

### CIS Route Integration

CIS can automatically configure BIG-IP based on OpenShift Routes:

```yaml
apiVersion: route.openshift.io/v1
kind: Route
metadata:
  name: my-app
  annotations:
    virtual-server.f5.com/ip: "10.0.10.150"
    virtual-server.f5.com/partition: "ocp"
spec:
  host: myapp.example.com
  to:
    kind: Service
    name: my-app-svc
```

## Prerequisites

- BIG-IP appliance (hardware or virtual) with LTM license
- BIG-IP management IP reachable from the cluster (for CIS)
- Dedicated partition on BIG-IP for OpenShift
- VLAN/tunnel connectivity between BIG-IP and cluster nodes
- For CIS: BIG-IP admin credentials
