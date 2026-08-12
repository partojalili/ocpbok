# Load Balancing — Azure

## Overview

On Azure, OpenShift IPI automatically provisions Azure Load Balancers for API and Ingress endpoints. No manual configuration is required for IPI installations.

## Automatically Created Load Balancers

| Load Balancer | Type | Purpose | Ports |
|---------------|------|---------|-------|
| API (external) | Azure Public Load Balancer | External API access | 6443 |
| API (internal) | Azure Internal Load Balancer | Internal API access | 6443, 22623 |
| Ingress | Azure Public Load Balancer | Application traffic (Routes) | 80, 443 |

## Azure Load Balancer SKUs

| SKU | Use Case |
|-----|----------|
| Standard (default) | Production workloads, zone-redundant, required for OpenShift |
| Basic | Not supported by OpenShift |

## Internal vs External Ingress

### External (Default)

```yaml
apiVersion: operator.openshift.io/v1
kind: IngressController
metadata:
  name: default
  namespace: openshift-ingress-operator
spec:
  endpointPublishingStrategy:
    type: LoadBalancerService
    loadBalancer:
      scope: External
```

### Internal-Only

```yaml
apiVersion: operator.openshift.io/v1
kind: IngressController
metadata:
  name: default
  namespace: openshift-ingress-operator
spec:
  endpointPublishingStrategy:
    type: LoadBalancerService
    loadBalancer:
      scope: Internal
      providerParameters:
        type: Azure
        azure:
          subnet: <internal-subnet-name>
```

## Private Cluster

For a private cluster (no public endpoints):

```yaml
# In install-config.yaml
publish: Internal
```

This creates only internal load balancers for both API and Ingress.

## UPI on Azure

For UPI installations, create load balancers via Azure CLI or ARM/Bicep templates:

```bash
# Create public load balancer for API
az network lb create \
  --name ocp-api-lb \
  --resource-group <rg> \
  --sku Standard \
  --frontend-ip-name api-frontend \
  --backend-pool-name api-backend \
  --public-ip-address ocp-api-pip

# Create health probe
az network lb probe create \
  --name api-probe \
  --resource-group <rg> \
  --lb-name ocp-api-lb \
  --protocol Https \
  --port 6443 \
  --path /readyz

# Create load balancing rule
az network lb rule create \
  --name api-rule \
  --resource-group <rg> \
  --lb-name ocp-api-lb \
  --frontend-ip-name api-frontend \
  --backend-pool-name api-backend \
  --probe-name api-probe \
  --protocol Tcp \
  --frontend-port 6443 \
  --backend-port 6443
```

## Validation

```bash
# Verify Azure load balancers
az network lb list --resource-group <rg> -o table

# Verify Ingress Controller service
oc get svc -n openshift-ingress router-default

# Verify IngressController status
oc get ingresscontroller default -n openshift-ingress-operator -o yaml
```
