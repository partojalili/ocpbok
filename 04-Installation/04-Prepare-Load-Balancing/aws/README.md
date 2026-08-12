# Load Balancing — AWS

## Overview

On AWS, OpenShift IPI automatically provisions Elastic Load Balancers (ELB) for the API and Ingress endpoints. No manual load balancer configuration is required for IPI installations.

## Automatically Created Load Balancers

| Load Balancer | Type | Purpose | Ports |
|---------------|------|---------|-------|
| API (external) | Network Load Balancer (NLB) | External API access | 6443 |
| API (internal) | Network Load Balancer (NLB) | Internal API access | 6443, 22623 |
| Ingress | Classic Load Balancer (CLB) or NLB | Application traffic (Routes) | 80, 443 |

## Load Balancer Types on AWS

| Type | Use Case | Layer |
|------|----------|-------|
| Network Load Balancer (NLB) | High-performance TCP/UDP, static IPs | Layer 4 |
| Application Load Balancer (ALB) | HTTP/HTTPS routing, path-based rules | Layer 7 |
| Classic Load Balancer (CLB) | Legacy, basic TCP/HTTP | Layer 4/7 |

## Switching Ingress to NLB

By default, OpenShift creates a Classic Load Balancer for Ingress. To use NLB instead:

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
      providerParameters:
        type: AWS
        aws:
          type: NLB
```

## AWS Load Balancer Operator (ALB)

For ALB-based Ingress (Layer 7), install the AWS Load Balancer Operator:

```bash
# Install via OperatorHub
oc get packagemanifest -n openshift-marketplace | grep aws-load-balancer

# After installation, create an AWSLoadBalancerController
cat <<EOF | oc apply -f -
apiVersion: networking.olm.openshift.io/v1
kind: AWSLoadBalancerController
metadata:
  name: cluster
spec:
  subnetTagging: Auto
  ingressClass: alb
EOF
```

### ALB Ingress Example

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: my-app
  annotations:
    alb.ingress.kubernetes.io/scheme: internet-facing
    alb.ingress.kubernetes.io/target-type: instance
spec:
  ingressClassName: alb
  rules:
    - host: myapp.example.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: my-app-svc
                port:
                  number: 8080
```

## UPI on AWS

For UPI installations, you must create the load balancers manually or via CloudFormation/Terraform:

```bash
# API Load Balancer (NLB)
aws elbv2 create-load-balancer \
  --name ocp-api \
  --type network \
  --scheme internet-facing \
  --subnets <subnet-ids>

# Target Group for API (port 6443)
aws elbv2 create-target-group \
  --name ocp-api-6443 \
  --protocol TCP \
  --port 6443 \
  --vpc-id <vpc-id> \
  --target-type ip \
  --health-check-protocol HTTPS \
  --health-check-path /readyz

# Target Group for Machine Config (port 22623)
aws elbv2 create-target-group \
  --name ocp-api-22623 \
  --protocol TCP \
  --port 22623 \
  --vpc-id <vpc-id> \
  --target-type ip
```

## Validation

```bash
# Verify load balancers
aws elbv2 describe-load-balancers --query 'LoadBalancers[*].[LoadBalancerName,Type,State.Code]' --output table

# Verify Ingress Controller LB type
oc get svc -n openshift-ingress router-default -o jsonpath='{.metadata.annotations}'
```
