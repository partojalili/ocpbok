# Validate Cluster

This step is platform-agnostic — the same validation applies to all infrastructure types.

## Cluster Health Check Script

```bash
#!/bin/bash
echo "=== Cluster Version ==="
oc get clusterversion

echo ""
echo "=== Nodes ==="
oc get nodes -o wide

echo ""
echo "=== ClusterOperators (Degraded/Unavailable) ==="
oc get co | grep -E 'True.*True|True.*False.*True|False'

echo ""
echo "=== Pending CSRs ==="
oc get csr --no-headers | grep -c Pending

echo ""
echo "=== Pod Health (non-Running pods) ==="
oc get pods -A --field-selector=status.phase!=Running,status.phase!=Succeeded --no-headers | wc -l

echo ""
echo "=== Storage Classes ==="
oc get sc

echo ""
echo "=== PVCs (non-Bound) ==="
oc get pvc -A | grep -v Bound | grep -v NAME

echo ""
echo "=== etcd Health ==="
oc get etcd -o=jsonpath='{range .items[0].status.conditions[?(@.type=="EtcdMembersAvailable")]}{.message}{"\n"}{end}'

echo ""
echo "=== Certificate Expiry (next 30 days) ==="
oc get co kube-apiserver -o json | jq -r '.status.conditions[] | select(.type=="NodeInstallerProgressing") | .message'
```

## Smoke Test Deployment

```bash
# Create test project
oc new-project smoke-test

# Deploy a test application
oc new-app --name=hello \
  --image=registry.access.redhat.com/ubi9/httpd-24:latest

# Expose the application
oc expose svc/hello

# Wait for rollout
oc rollout status deployment/hello

# Test access
ROUTE=$(oc get route hello -o jsonpath='{.spec.host}')
curl -s -o /dev/null -w "%{http_code}" http://$ROUTE
# Expected: 200 or 403 (default httpd page)

# Test storage
cat <<EOF | oc apply -f -
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: test-pvc
  namespace: smoke-test
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 5Gi
EOF

oc get pvc test-pvc -w
# Expected: STATUS = Bound

# Cleanup
oc delete project smoke-test
```
