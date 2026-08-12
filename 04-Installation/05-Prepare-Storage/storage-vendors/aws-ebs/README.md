# AWS EBS CSI Driver

**Vendor:** Amazon Web Services  
**CSI Driver:** AWS EBS CSI (built-in with OpenShift on AWS)  
**Operator:** Cluster CSI Driver Operator (managed by OpenShift)

## Overview

The AWS EBS CSI driver is automatically installed and managed by OpenShift when the platform is AWS.

## Supported Volume Types

| Volume Type | Use Case | Max IOPS | Max Throughput |
|------------|----------|----------|----------------|
| gp3 (default) | General purpose | 16,000 | 1,000 MB/s |
| gp2 | Legacy general purpose | 16,000 | 250 MB/s |
| io2 | High-performance databases | 64,000 | 1,000 MB/s |
| st1 | Throughput-optimized HDD | 500 | 500 MB/s |
| sc1 | Cold storage HDD | 250 | 250 MB/s |

## Access Modes

- RWO only (EBS volumes are single-AZ, single-attach)
- For RWX, use AWS EFS (see `aws-efs/`)

## Default StorageClass

OpenShift creates `gp3-csi` as the default StorageClass on AWS:

```bash
oc get sc gp3-csi
```

## Custom StorageClass

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: aws-io2
provisioner: ebs.csi.aws.com
parameters:
  type: io2
  iopsPerGB: "50"
  encrypted: "true"
reclaimPolicy: Delete
allowVolumeExpansion: true
volumeBindingMode: WaitForFirstConsumer
```

## Prerequisites

- OpenShift installed on AWS
- IAM role with EBS permissions (automatically configured by IPI)
- Sufficient EBS volume quota in the target region
