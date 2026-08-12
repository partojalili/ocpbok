# NSX Advanced Load Balancer (Avi)

**Vendor:** VMware (Broadcom)  
**Integration:** Avi Kubernetes Operator (AKO)  
**Purpose:** Software-defined load balancing for vSphere environments

## Overview

NSX Advanced Load Balancer (formerly Avi Networks) provides software-defined load balancing for OpenShift clusters running on vSphere. It replaces the need for HAProxy or hardware LBs in VMware environments and provides L4-L7 load balancing with analytics.

## Components

| Component | Description |
|-----------|-------------|
| Avi Controller | Central management plane (VM appliance) |
| Service Engines (SE) | Data plane VMs that handle traffic |
| AKO (Avi Kubernetes Operator) | Watches OpenShift Services/Routes, configures Avi |

## Architecture

```
Client → Avi Service Engine (VIP) → OpenShift Node → Pod
                  ↑
         Avi Controller (config)
                  ↑
         AKO (watches OCP Services/Routes)
```

## Pre-Install: NSX ALB as API/Ingress Load Balancer

### Avi Controller Configuration

1. Deploy Avi Controller VM on vSphere
2. Configure cloud connector for vCenter
3. Create SE group for OpenShift
4. Configure VIP network and IP pool

### Virtual Service for API

```json
{
  "name": "ocp-api",
  "vip": [{"ip_address": {"addr": "10.0.10.100", "type": "V4"}}],
  "services": [{"port": 6443}],
  "pool_ref": "/api/pool?name=ocp-api-pool",
  "application_profile_ref": "/api/applicationprofile?name=System-SSL-Application",
  "ssl_profile_ref": "/api/sslprofile?name=System-Standard"
}
```

### Pool for Control Plane Nodes

```json
{
  "name": "ocp-api-pool",
  "servers": [
    {"ip": {"addr": "10.0.10.10", "type": "V4"}, "port": 6443},
    {"ip": {"addr": "10.0.10.11", "type": "V4"}, "port": 6443},
    {"ip": {"addr": "10.0.10.12", "type": "V4"}, "port": 6443}
  ],
  "health_monitor_refs": ["/api/healthmonitor?name=ocp-api-hm"]
}
```

## Post-Install: Avi Kubernetes Operator (AKO)

### Install AKO via Helm

```bash
helm repo add ako https://projects.registry.vmware.com/chartrepo/ako

helm install ako ako/ako \
  --namespace avi-system --create-namespace \
  --set ControllerSettings.controllerHost=<avi-controller-ip> \
  --set avicredentials.username=admin \
  --set avicredentials.password=<password> \
  --set AKOSettings.clusterName=ocp-cluster \
  --set NetworkSettings.vipNetworkList[0].networkName=<vip-network> \
  --set NetworkSettings.vipNetworkList[0].cidr=<vip-cidr> \
  --set L7Settings.shardVSSize=SMALL
```

### AKO Route Integration

AKO automatically creates Avi Virtual Services for OpenShift Routes:

```yaml
apiVersion: route.openshift.io/v1
kind: Route
metadata:
  name: my-app
spec:
  host: myapp.apps.ocp.example.com
  to:
    kind: Service
    name: my-app-svc
  tls:
    termination: edge
```

## Prerequisites

- Avi Controller deployed on vSphere (21.1.x+ recommended)
- vCenter cloud connector configured
- Service Engine group with available capacity
- VIP network with available IP pool
- DNS configured to resolve VIPs
- For AKO: Avi Controller reachable from OpenShift nodes (port 443)
