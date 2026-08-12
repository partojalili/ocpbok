# AWS EFS Pre-Installation Validation

**Vendor:** Amazon Web Services  
**CSI Driver:** AWS EFS CSI Driver  
**Operator:** Installed via OperatorHub (aws-efs-csi-driver-operator)

## Requirements

| Component | Requirement |
|-----------|-------------|
| AWS Region | EFS supported region |
| VPC | Same VPC as OpenShift cluster |
| Security Groups | NFS port 2049 from cluster subnets |
| IAM | EFS permissions for CSI driver |

## Validation Checklist

### EFS Filesystem Validation

```bash
# List existing EFS filesystems
aws efs describe-file-systems --query 'FileSystems[*].[FileSystemId,LifeCycleState,Name]' --output table

# If no EFS exists yet, verify ability to create one
aws efs create-file-system --performance-mode generalPurpose --throughput-mode bursting --tags Key=Name,Value=ocp-efs --dry-run 2>&1 || echo "Dry-run not supported; verify IAM permissions"
```

### Network and Security Group Validation

```bash
# Get cluster VPC ID
VPC_ID=$(aws ec2 describe-vpcs --filters "Name=tag:Name,Values=*ocp*" --query 'Vpcs[0].VpcId' --output text)

# Get cluster subnet IDs
aws ec2 describe-subnets --filters "Name=vpc-id,Values=${VPC_ID}" --query 'Subnets[*].[SubnetId,AvailabilityZone]' --output table

# Verify security group allows NFS (2049) from cluster nodes
aws ec2 describe-security-groups --filters "Name=vpc-id,Values=${VPC_ID}" \
  --query 'SecurityGroups[*].[GroupId,GroupName]' --output table

# Check EFS mount targets (if EFS already exists)
aws efs describe-mount-targets --file-system-id <fs-id> --query 'MountTargets[*].[MountTargetId,SubnetId,LifeCycleState]' --output table
```

### Mount Target Availability

```bash
# EFS needs a mount target in every AZ where cluster nodes run
# Get cluster node AZs
oc get nodes -o jsonpath='{range .items[*]}{.metadata.labels.topology\.kubernetes\.io/zone}{"\n"}{end}' | sort -u

# Verify mount targets exist in those AZs
aws efs describe-mount-targets --file-system-id <fs-id>
```

### IAM Permissions

```bash
# The EFS CSI driver needs these IAM actions:
# elasticfilesystem:DescribeAccessPoints
# elasticfilesystem:CreateAccessPoint
# elasticfilesystem:DeleteAccessPoint
# elasticfilesystem:DescribeFileSystems
# elasticfilesystem:DescribeMountTargets

# Verify the OpenShift IAM role has these permissions
aws iam get-role-policy --role-name <ocp-efs-role> --policy-name <policy-name>
```

### NFS Connectivity from Nodes

```bash
# Test NFS port from cluster nodes to EFS mount target
oc debug node/<node-name> -- chroot /host nc -zv <efs-mount-target-ip> 2049

# Or use the EFS DNS name
oc debug node/<node-name> -- chroot /host nc -zv <fs-id>.efs.<region>.amazonaws.com 2049
```

### Operator Availability

```bash
# Verify EFS CSI operator is available
oc get packagemanifest -n openshift-marketplace | grep efs
```

## Pre-Installation Summary

| Check | Expected |
|-------|----------|
| EFS filesystem exists or can be created | Permissions verified |
| Mount targets in all cluster AZs | One per AZ |
| Security group allows port 2049 | From cluster node subnets |
| IAM role has EFS permissions | Access point CRUD + describe |
| NFS port reachable from nodes | Port 2049 open |
| EFS CSI operator available | Listed in OperatorHub |
