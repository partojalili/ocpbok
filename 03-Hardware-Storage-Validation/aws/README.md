# Hardware and Storage Validation — AWS

## AWS Account Validation

### Credentials

- [ ] AWS access key and secret key with sufficient permissions
- [ ] Or IAM role with required policies attached

```bash
# Verify AWS credentials
aws sts get-caller-identity
```

### Service Quotas

Verify sufficient quotas in the target region:

| Resource | Minimum Required | Check Command |
|----------|-----------------|---------------|
| EC2 instances (m5.xlarge or larger) | 6+ | `aws service-quotas get-service-quota --service-code ec2 --quota-code L-1216C47A` |
| Elastic IPs | 1 | `aws service-quotas get-service-quota --service-code ec2 --quota-code L-0263D0A3` |
| VPCs | 1 | `aws service-quotas get-service-quota --service-code vpc --quota-code L-F678F1CE` |
| EBS volumes | 18+ | `aws service-quotas get-service-quota --service-code ebs --quota-code L-D18FCD1D` |
| NLB / ALB | 2 | `aws service-quotas get-service-quota --service-code elasticloadbalancing --quota-code L-53DA6B97` |

### Region Availability

- [ ] Target region supports the required instance types
- [ ] Target region has at least 3 availability zones (for HA)

```bash
# List AZs in target region
aws ec2 describe-availability-zones --region us-east-1 --query 'AvailabilityZones[].ZoneName'
```

---

## Compute Validation

### Instance Types

| Node Role | Minimum | Recommended |
|-----------|---------|-------------|
| Control Plane | m5.xlarge (4 vCPU, 16 GB) | m5.2xlarge (8 vCPU, 32 GB) |
| Worker | m5.xlarge (4 vCPU, 16 GB) | m5.4xlarge (16 vCPU, 64 GB) |

```bash
# Verify instance type availability in region
aws ec2 describe-instance-type-offerings --location-type availability-zone \
  --filters Name=instance-type,Values=m5.2xlarge --region us-east-1
```

---

## Storage Validation

### EBS

- [ ] gp3 volume type available in target region (default for OCP 4.22)
- [ ] Sufficient EBS volume quota

### EFS (for RWX)

- [ ] EFS is available in the target region (if RWX workloads are required)
- [ ] AWS EFS CSI driver operator available in OCP 4.22

### S3 (for object storage)

- [ ] S3 is accessible from the VPC (via gateway endpoint or NAT)

---

## Network Validation

### VPC

- [ ] VPC CIDR does not conflict with cluster/service/pod CIDRs
- [ ] Public and private subnets available (at least 1 per AZ for HA)
- [ ] Internet Gateway (public clusters) or NAT Gateway (private clusters) exists

### DNS

- [ ] Route 53 hosted zone exists for the base domain (or delegated zone)
- [ ] Hosted zone is resolvable

```bash
aws route53 list-hosted-zones-by-name --dns-name example.com
```

### Security Groups

IPI creates security groups automatically. For UPI, verify:

- [ ] Security group rules allow all required ports (see `04-Installation/03-Prepare-Networking`)

---

## IAM Validation

IPI requires an IAM user or role with broad permissions. Verify:

```bash
# Quick check — list policies attached to the user
aws iam list-attached-user-policies --user-name openshift-installer
```

Required policies (simplified):
- EC2 full access
- ELB full access
- IAM limited access (create roles, instance profiles)
- Route 53 full access
- S3 full access
- VPC full access

---

## Pull Secret Validation

Same as bare metal (see `baremetal/ipi/`).
