# GCP Filestore Pre-Installation Validation

**Vendor:** Google Cloud  
**CSI Driver:** GCP Filestore CSI  
**Operator:** Manual deployment

## Requirements

| Component | Requirement |
|-----------|-------------|
| GCP Project | Active with billing enabled |
| Filestore API | Enabled |
| VPC Network | Same network as OpenShift cluster |
| Firewall | NFS port 2049 allowed |

## Validation Checklist

### API Enablement

```bash
# Verify Filestore API is enabled
gcloud services list --enabled --filter="name:file.googleapis.com"

# Enable if not
gcloud services enable file.googleapis.com
```

### Quota Validation

```bash
# Check Filestore instance quota
gcloud compute regions describe <region> \
  --format="json" | grep -i filestore

# Check per-tier limits
gcloud filestore instances list --project=<project-id> --location=<region>
```

### Network Validation

```bash
# Verify VPC network used by the cluster
gcloud compute networks list

# Verify firewall rules allow NFS from cluster nodes
gcloud compute firewall-rules list --filter="network:<vpc-name>" \
  --format="table(name,direction,allowed,sourceRanges)"

# Verify NFS port (2049) is allowed
gcloud compute firewall-rules describe <rule-name>
```

### Existing Filestore Instances

```bash
# List existing Filestore instances
gcloud filestore instances list --location=<region> \
  --format="table(name,tier,networks.ipAddresses,fileShares.name,fileShares.capacityGb,state)"
```

### IAM Permissions

```bash
# Filestore CSI needs:
# file.instances.create, file.instances.delete, file.instances.get
# file.instances.list, file.instances.update

# Verify service account roles
gcloud projects get-iam-policy <project-id> \
  --flatten="bindings[].members" \
  --filter="bindings.members:serviceAccount:<sa-email>" \
  --format="table(bindings.role)" | grep -i file
```

### Connectivity Test (if Filestore exists)

```bash
# Test NFS from cluster nodes
oc debug node/<node-name> -- chroot /host nc -zv <filestore-ip> 2049

# Test mount capability
oc debug node/<node-name> -- chroot /host showmount -e <filestore-ip>
```

## Pre-Installation Summary

| Check | Expected |
|-------|----------|
| Filestore API enabled | Listed in enabled services |
| Filestore quota sufficient | Capacity for expected instances |
| VPC network matches cluster | Same network |
| Firewall allows NFS port 2049 | Rule exists |
| IAM roles for Filestore | file.instances CRUD granted |
| NFS port reachable (if existing) | Port 2049 open |
