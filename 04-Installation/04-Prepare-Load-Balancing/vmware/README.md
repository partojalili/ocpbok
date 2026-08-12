# Load Balancing — VMware vSphere

## Overview

OpenShift on vSphere does not automatically provision external load balancers. You must provide load balancing for the API and Ingress endpoints.

## Options

| Option | Type | Best For |
|--------|------|----------|
| HAProxy (manual) | Software LB | POC, small deployments |
| NSX Advanced Load Balancer (Avi) | Virtual appliance | Enterprise vSphere environments |
| F5 BIG-IP | Hardware/virtual | Existing F5 customers |
| keepalived + HAProxy (IPI) | Built-in | vSphere IPI with VIP support |

## vSphere IPI Built-in Load Balancing

vSphere IPI supports built-in keepalived + HAProxy when you specify VIPs in `install-config.yaml`:

```yaml
platform:
  vsphere:
    apiVIPs:
      - 10.0.10.100
    ingressVIPs:
      - 10.0.10.101
    vcenters:
      - server: vcenter.example.com
        user: administrator@vsphere.local
        password: <password>
        datacenters:
          - DC1
```

## vSphere UPI — External HAProxy

For UPI, configure an external HAProxy load balancer:

```bash
# /etc/haproxy/haproxy.cfg

frontend api
    bind *:6443
    mode tcp
    default_backend api_servers

frontend machine-config
    bind *:22623
    mode tcp
    default_backend machine_config_servers

frontend ingress-http
    bind *:80
    mode tcp
    default_backend ingress_http_servers

frontend ingress-https
    bind *:443
    mode tcp
    default_backend ingress_https_servers

backend api_servers
    mode tcp
    balance roundrobin
    option httpchk GET /readyz HTTP/1.0
    option log-health-checks
    server master0 10.0.10.10:6443 check check-ssl verify none
    server master1 10.0.10.11:6443 check check-ssl verify none
    server master2 10.0.10.12:6443 check check-ssl verify none

backend machine_config_servers
    mode tcp
    balance roundrobin
    server master0 10.0.10.10:22623 check
    server master1 10.0.10.11:22623 check
    server master2 10.0.10.12:22623 check

backend ingress_http_servers
    mode tcp
    balance roundrobin
    server worker0 10.0.10.20:80 check
    server worker1 10.0.10.21:80 check
    server worker2 10.0.10.22:80 check

backend ingress_https_servers
    mode tcp
    balance roundrobin
    server worker0 10.0.10.20:443 check
    server worker1 10.0.10.21:443 check
    server worker2 10.0.10.22:443 check
```

## Validation

```bash
# Test API endpoint
curl -sk https://<api-vip>:6443/readyz

# Test Ingress endpoint
curl -sk https://<ingress-vip>

# Verify load balancer endpoints from within the cluster
oc get endpoints -n openshift-kube-apiserver
```
