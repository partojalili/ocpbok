# AWS EBS Pre-Installation Validation

**Vendor:** Amazon Web Services  
**CSI Driver:** AWS EBS CSI (built-in with OpenShift on AWS)  
**Operator:** Cluster CSI Driver Operator (managed by OpenShift)

## Requirements

| Component | Requirement |
|-----------|-------------|
| AWS Region | EBS supported (all commercial regions) |
| EC2 Instance Type | Must support EBS (all standard types) |
| IAM Permissions | EC2, EBS volume management |

## Validation Checklist

### AWS Credentials and Permissions

```bash
# Verify AWS CLI access
aws sts get-caller-identity

# Verify EBS permissions (required IAM actions):
# ec2:CreateVolume, ec2:DeleteVolume, ec2:AttachVolume, ec2:DetachVolume
# ec2:DescribeVolumes, ec2:DescribeInstances, ec2:CreateSnapshot
# ec2:DeleteSnapshot, ec2:DescribeSnapshots, ec2:CreateTags

# Check current IAM policies
aws iam list-attached-user-policies --user-name <username>
```

### Service Quotas

```bash
# Check EBS volume limits
aws service-quotas get-service-quota \
  --service-code ebs \
  --quota-code L-D18FCD1D  # General Purpose SSD (gp3)

# Check snapshot limits
aws service-quotas get-service-quota \
  --service-code ebs \
  --quota-code L-309BACF6  # Snapshots per region
```

### Availability Zone Validation

```bash
# List AZs where cluster nodes are deployed
oc get nodes -o jsonpath='{range .items[*]}{.metadata.labels.topology\.kubernetes\.io/zone}{"\n"}{end}' | sort -u

# Verify EBS is available in those AZs (standard in all AZs)
aws ec2 describe-availability-zones --region <region> --query 'AvailabilityZones[].ZoneName'
```

### Encryption Validation (if required)

```bash
# Verify KMS key for EBS encryption
aws kms describe-key --key-id <key-id>

# Verify default encryption setting
aws ec2 get-ebs-default-kms-key-id --region <region>
aws ec2 get-ebs-encryption-by-default --region <region>
```

### CSI Driver Verification

```bash
# Verify CSI driver is running on the cluster
oc get pods -n openshift-cluster-csi-drivers -l app=aws-ebs-csi-driver

# Verify default StorageClass
oc get sc gp3-csi
```

## Pre-Installation Summary

| Check | Expected |
|-------|----------|
| AWS credentials valid | `sts get-caller-identity` succeeds |
| IAM permissions for EBS | Volume create/attach/delete granted |
| EBS quota sufficient | Enough headroom for expected PVs |
| AZs match cluster nodes | Volumes will provision in correct AZs |
| KMS key available (if encrypting) | Key accessible by IAM role |
| EBS CSI driver running | Pods healthy in openshift-cluster-csi-drivers |
