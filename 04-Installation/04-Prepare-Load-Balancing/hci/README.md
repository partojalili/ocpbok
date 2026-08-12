# Load Balancing — HCI (Hyperconverged Infrastructure)

## Overview

HCI platforms (Nutanix, vSAN, etc.) follow the same load balancing patterns as their underlying hypervisor, with some platform-specific options.

## Nutanix AHV

### IPI (Built-in)

Nutanix IPI supports built-in keepalived + HAProxy with VIPs:

```yaml
platform:
  nutanix:
    apiVIPs:
      - 10.0.10.100
    ingressVIPs:
      - 10.0.10.101
    prismCentral:
      endpoint:
        address: prism-central.example.com
        port: 9440
      username: admin
      password: <password>
    subnetUUIDs:
      - <subnet-uuid>
```

### UPI / External LB

For UPI on Nutanix, use an external load balancer (HAProxy, F5, or Nutanix Flow):
- Same HAProxy configuration as vSphere UPI
- API on port 6443, Machine Config on port 22623
- Ingress on ports 80 and 443

## vSAN-Based HCI

For vSAN environments, follow the VMware vSphere load balancing guide:
- IPI: built-in keepalived with VIPs
- UPI: external HAProxy or F5

## Validation

```bash
# Test API endpoint
curl -sk https://<api-vip>:6443/readyz

# Test Ingress
curl -sk https://<ingress-vip>

# Verify keepalived pods (IPI)
oc get pods -n openshift-nutanix-infra -l app=keepalived
```
