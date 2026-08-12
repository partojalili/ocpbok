# AWS EFS CSI Driver

**Vendor:** Amazon Web Services  
**CSI Driver:** AWS EFS CSI  
**Operator:** `aws-efs-csi-driver-operator` (via OperatorHub)

## Overview

AWS EFS provides managed NFS storage with RWX support. Unlike EBS, EFS volumes are accessible across multiple availability zones.

## Access Modes

- RWO, RWX, ROX

## Installation

```bash
cat <<EOF | oc apply -f -
apiVersion: operators.coreos.com/v1alpha1
kind: Subscription
metadata:
  name: aws-efs-csi-driver-operator
  namespace: openshift-cluster-csi-drivers
spec:
  channel: stable
  installPlanApproval: Automatic
  name: aws-efs-csi-driver-operator
  source: redhat-operators
  sourceNamespace: openshift-marketplace
EOF
```

### Create the CSI Driver CR

```bash
cat <<EOF | oc apply -f -
apiVersion: operator.openshift.io/v1
kind: ClusterCSIDriver
metadata:
  name: efs.csi.aws.com
spec:
  managementState: Managed
EOF
```

## StorageClass Example

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: aws-efs
provisioner: efs.csi.aws.com
parameters:
  provisioningMode: efs-ap
  fileSystemId: fs-0123456789abcdef0
  directoryPerms: "700"
reclaimPolicy: Delete
volumeBindingMode: Immediate
```

## Prerequisites

- EFS filesystem created in the same VPC as the OpenShift cluster
- Security group allows NFS (port 2049) from cluster nodes to EFS mount targets
- EFS mount targets created in each availability zone used by the cluster
- IAM role with EFS permissions
