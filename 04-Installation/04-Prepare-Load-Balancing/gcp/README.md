# Load Balancing — GCP

## Overview

On GCP, OpenShift IPI automatically provisions Google Cloud Load Balancers for API and Ingress endpoints. No manual configuration is required for IPI installations.

## Automatically Created Load Balancers

| Load Balancer | Type | Purpose | Ports |
|---------------|------|---------|-------|
| API (external) | External TCP/UDP Network LB | External API access | 6443 |
| API (internal) | Internal TCP/UDP LB | Internal API access | 6443, 22623 |
| Ingress | External TCP Proxy LB | Application traffic (Routes) | 80, 443 |

## GCP Load Balancer Types

| Type | Layer | Use Case |
|------|-------|----------|
| External TCP/UDP Network LB | Layer 4 | API endpoints, high-throughput TCP |
| Internal TCP/UDP LB | Layer 4 | Internal API, internal services |
| External HTTP(S) LB | Layer 7 | Path-based routing, CDN, SSL offload |
| Internal HTTP(S) LB | Layer 7 | Internal HTTP services |

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
```

## Private Cluster

```yaml
# In install-config.yaml
publish: Internal
```

## UPI on GCP

For UPI installations, create load balancers via gcloud CLI or Deployment Manager:

```bash
# Create health check for API
gcloud compute health-checks create https api-health-check \
  --port 6443 \
  --request-path /readyz

# Create backend service
gcloud compute backend-services create ocp-api-backend \
  --protocol TCP \
  --health-checks api-health-check \
  --region <region>

# Add instance groups
gcloud compute backend-services add-backend ocp-api-backend \
  --instance-group <control-plane-ig> \
  --instance-group-region <region> \
  --region <region>

# Create forwarding rule
gcloud compute forwarding-rules create ocp-api \
  --load-balancing-scheme EXTERNAL \
  --ports 6443 \
  --region <region> \
  --backend-service ocp-api-backend
```

## Validation

```bash
# Verify GCP load balancers
gcloud compute forwarding-rules list --format="table(name,IPAddress,target,portRange)"

# Verify Ingress Controller service
oc get svc -n openshift-ingress router-default

# Verify IngressController status
oc get ingresscontroller default -n openshift-ingress-operator -o yaml
```
