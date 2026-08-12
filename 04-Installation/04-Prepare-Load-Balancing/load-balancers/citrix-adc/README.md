# Citrix ADC (NetScaler)

**Vendor:** Cloud Software Group (Citrix)  
**Integration:** Citrix Ingress Controller  
**Purpose:** Enterprise load balancing and application delivery

## Overview

Citrix ADC (formerly NetScaler) provides enterprise-grade load balancing for OpenShift. It can act as the external load balancer for API/Ingress endpoints and as an application-level Ingress controller via the Citrix Ingress Controller.

## Deployment Models

| Model | Description |
|-------|-------------|
| Citrix ADC MPX | Hardware appliance |
| Citrix ADC VPX | Virtual appliance (VMware, KVM, cloud) |
| Citrix ADC CPX | Containerized form factor (runs as a pod) |

## Pre-Install: Citrix ADC as API/Ingress Load Balancer

### Virtual Server Configuration (CLI)

```
# API Virtual Server
add lb vserver ocp-api TCP 10.0.10.100 6443
add serviceGroup ocp-api-sg TCP
bind serviceGroup ocp-api-sg 10.0.10.10 6443
bind serviceGroup ocp-api-sg 10.0.10.11 6443
bind serviceGroup ocp-api-sg 10.0.10.12 6443
add lb monitor ocp-api-monitor HTTPS-ECV -send "GET /readyz" -recv "ok" -secure YES
bind serviceGroup ocp-api-sg -monitorName ocp-api-monitor
bind lb vserver ocp-api ocp-api-sg

# Ingress Virtual Server (HTTPS)
add lb vserver ocp-ingress-https TCP 10.0.10.101 443
add serviceGroup ocp-ingress-sg TCP
bind serviceGroup ocp-ingress-sg 10.0.10.20 443
bind serviceGroup ocp-ingress-sg 10.0.10.21 443
bind serviceGroup ocp-ingress-sg 10.0.10.22 443
bind lb vserver ocp-ingress-https ocp-ingress-sg

# Ingress Virtual Server (HTTP)
add lb vserver ocp-ingress-http TCP 10.0.10.101 80
add serviceGroup ocp-ingress-http-sg TCP
bind serviceGroup ocp-ingress-http-sg 10.0.10.20 80
bind serviceGroup ocp-ingress-http-sg 10.0.10.21 80
bind serviceGroup ocp-ingress-http-sg 10.0.10.22 80
bind lb vserver ocp-ingress-http ocp-ingress-http-sg
```

## Post-Install: Citrix Ingress Controller

### Deploy Citrix Ingress Controller

```bash
# Create ADC credentials secret
oc create secret generic nslogin \
  --from-literal=username=nsroot \
  --from-literal=password=<password> \
  -n citrix-system

# Deploy via Helm
helm repo add citrix https://citrix.github.io/citrix-helm-charts/
helm install citrix-cpx citrix/citrix-cloud-native \
  --namespace citrix-system \
  --set cpx.enabled=true \
  --set cpx.openshift=true
```

### Citrix Ingress Controller with External ADC

```bash
helm install citrix-cic citrix/citrix-ingress-controller \
  --namespace citrix-system \
  --set nsIP=<adc-management-ip> \
  --set license.accept=yes \
  --set adcCredentialSecret=nslogin \
  --set openshift=true
```

## Prerequisites

- Citrix ADC appliance (MPX/VPX) or CPX image available
- NSIP (management IP) reachable from the cluster
- SNIP configured on the same subnet as cluster nodes
- ADC license with LB feature enabled
- For Ingress Controller: admin credentials for the ADC
