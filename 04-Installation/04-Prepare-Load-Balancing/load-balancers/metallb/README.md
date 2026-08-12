# MetalLB

**Vendor:** Red Hat (upstream: MetalLB)  
**Operator:** `metallb-operator` (via OperatorHub)  
**Purpose:** Provides `type: LoadBalancer` service support on bare metal

## Overview

MetalLB enables Kubernetes `Service type: LoadBalancer` on bare metal clusters where no cloud load balancer integration exists. It assigns external IPs to services from a configured pool and advertises them via BGP or Layer 2 (ARP/NDP).

## Modes

| Mode | Protocol | Use Case |
|------|----------|----------|
| Layer 2 (L2) | ARP (IPv4) / NDP (IPv6) | Simple deployments, no router integration needed |
| BGP | BGP peering with network routers | Production, multi-rack, traffic engineering |

## Installation

```bash
cat <<EOF | oc apply -f -
apiVersion: v1
kind: Namespace
metadata:
  name: metallb-system
---
apiVersion: operators.coreos.com/v1alpha1
kind: Subscription
metadata:
  name: metallb-operator
  namespace: metallb-system
spec:
  channel: stable-4.22
  installPlanApproval: Automatic
  name: metallb-operator
  source: redhat-operators
  sourceNamespace: openshift-marketplace
EOF
```

## Create MetalLB Instance

```yaml
apiVersion: metallb.io/v1beta1
kind: MetalLB
metadata:
  name: metallb
  namespace: metallb-system
spec: {}
```

## Layer 2 Configuration

### IP Address Pool

```yaml
apiVersion: metallb.io/v1beta1
kind: IPAddressPool
metadata:
  name: l2-pool
  namespace: metallb-system
spec:
  addresses:
    - 10.0.10.200-10.0.10.250
```

### L2 Advertisement

```yaml
apiVersion: metallb.io/v1beta1
kind: L2Advertisement
metadata:
  name: l2-advert
  namespace: metallb-system
spec:
  ipAddressPools:
    - l2-pool
```

## BGP Configuration

### IP Address Pool

```yaml
apiVersion: metallb.io/v1beta1
kind: IPAddressPool
metadata:
  name: bgp-pool
  namespace: metallb-system
spec:
  addresses:
    - 192.168.100.0/24
```

### BGP Peer

```yaml
apiVersion: metallb.io/v1beta1
kind: BGPPeer
metadata:
  name: router-peer
  namespace: metallb-system
spec:
  myASN: 64500
  peerASN: 64501
  peerAddress: 10.0.0.1
  password: <shared-secret>
```

### BGP Advertisement

```yaml
apiVersion: metallb.io/v1beta1
kind: BGPAdvertisement
metadata:
  name: bgp-advert
  namespace: metallb-system
spec:
  ipAddressPools:
    - bgp-pool
```

## Using MetalLB with a Service

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-app
spec:
  type: LoadBalancer
  selector:
    app: my-app
  ports:
    - port: 80
      targetPort: 8080
```

MetalLB assigns an IP from the pool automatically.

## Validation

```bash
# Verify MetalLB pods
oc get pods -n metallb-system

# Verify IP address pools
oc get ipaddresspool -n metallb-system

# Verify service got an external IP
oc get svc my-app -o jsonpath='{.status.loadBalancer.ingress[0].ip}'

# Verify L2 or BGP advertisement
oc get l2advertisement -n metallb-system
oc get bgpadvertisement -n metallb-system
```

## Prerequisites

- For L2: IP pool must be on the same subnet as the node network
- For BGP: Network routers must support BGP peering with the cluster nodes
- IP addresses in the pool must not conflict with DHCP or other allocations
