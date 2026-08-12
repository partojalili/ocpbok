# GCP Persistent Disk Pre-Installation Validation

**Vendor:** Google Cloud  
**CSI Driver:** GCE PD CSI (built-in with OpenShift on GCP)  
**Operator:** Cluster CSI Driver Operator (managed by OpenShift)

## Requirements

| Component | Requirement |
|-----------|-------------|
| GCP Project | Active with billing enabled |
| Compute Engine API | Enabled |
| Region/Zone | Same as OpenShift cluster |
| IAM | Persistent disk management permissions |

## Validation Checklist

### GCP Project and API Validation

```bash
# Verify project and active billing
gcloud projects describe <project-id>

# Verify Compute Engine API is enabled
gcloud services list --enabled --filter="name:compute.googleapis.com"
```

### Quota Validation

```bash
# Check persistent disk quota in the region
gcloud compute regions describe <region> \
  --format="table(quotas.filter(metric='SSD_TOTAL_GB').limit, quotas.filter(metric='SSD_TOTAL_GB').usage)"

# Check disk count quota
gcloud compute regions describe <region> \
  --format="table(quotas.filter(metric='DISKS_TOTAL_GB').limit, quotas.filter(metric='DISKS_TOTAL_GB').usage)"
```

### IAM Permissions

```bash
# The CSI driver service account needs:
# compute.disks.create, compute.disks.delete, compute.disks.get
# compute.disks.list, compute.instances.attachDisk, compute.instances.detachDisk
# compute.snapshots.create, compute.snapshots.delete

# Verify service account roles
gcloud projects get-iam-policy <project-id> \
  --flatten="bindings[].members" \
  --filter="bindings.members:serviceAccount:<sa-email>" \
  --format="table(bindings.role)"
```

### Zone Validation

```bash
# Get cluster node zones
oc get nodes -o jsonpath='{range .items[*]}{.metadata.labels.topology\.kubernetes\.io/zone}{"\n"}{end}' | sort -u

# Verify disk types available in those zones
gcloud compute disk-types list --zones=<zone> --format="table(name,validDiskSize)"
```

### Encryption Validation (if required)

```bash
# Verify CMEK key exists
gcloud kms keys list --location <region> --keyring <keyring-name>

# Verify service account has access to the KMS key
gcloud kms keys get-iam-policy <key-name> --location <region> --keyring <keyring-name>
```

### CSI Driver Verification

```bash
# Verify CSI driver pods
oc get pods -n openshift-cluster-csi-drivers -l app=gcp-pd-csi-driver

# Verify default StorageClass
oc get sc standard-csi
```

## Pre-Installation Summary

| Check | Expected |
|-------|----------|
| GCP project active with billing | Project accessible |
| Compute Engine API enabled | Listed in enabled services |
| PD quota sufficient | Headroom for expected PVs |
| IAM roles assigned | Disk CRUD + attach/detach |
| Disk types available in cluster zones | pd-ssd, pd-balanced confirmed |
| CMEK key accessible (if encrypting) | SA has encrypter/decrypter role |
| GCE PD CSI driver running | Pods healthy |
