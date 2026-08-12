# Hardware and Storage Validation — GCP

## GCP Project Validation

### Credentials

- [ ] Service account with required IAM roles
- [ ] Service account key JSON file available

```bash
# Verify GCP credentials
gcloud auth activate-service-account --key-file=<key-file>.json
gcloud config set project <project-id>
gcloud projects describe <project-id>
```

### API Enablement

- [ ] Required APIs enabled on the project:

```bash
for API in compute.googleapis.com dns.googleapis.com iam.googleapis.com \
  serviceusage.googleapis.com storage-api.googleapis.com; do
  gcloud services list --enabled --filter="name:${API}" --format="value(name)" | \
    grep -q "$API" && echo "$API: enabled" || echo "$API: DISABLED"
done
```

### Quotas

| Resource | Minimum Required |
|----------|-----------------|
| CPUs (per region) | 48+ |
| In-use IP addresses | 6+ |
| Persistent disk SSD (GB) | 2,000+ |
| Firewall rules | 20+ |

```bash
gcloud compute regions describe us-central1 --format="table(quotas[].metric,quotas[].limit,quotas[].usage)"
```

---

## Compute Validation

### Machine Types

| Node Role | Minimum | Recommended |
|-----------|---------|-------------|
| Control Plane | n2-standard-4 (4 vCPU, 16 GB) | n2-standard-8 (8 vCPU, 32 GB) |
| Worker | n2-standard-4 (4 vCPU, 16 GB) | n2-standard-8 (8 vCPU, 32 GB) |

```bash
gcloud compute machine-types list --zones=us-central1-a --filter="name:n2-standard-8"
```

---

## Storage Validation

### Persistent Disks

- [ ] pd-ssd available in the target region (default for OCP 4.22)
- [ ] GCE PD CSI driver is the default StorageClass

### Filestore (for RWX)

- [ ] Filestore available in the target region (if RWX required)

### Cloud Storage (for object)

- [ ] GCS bucket accessible for backups

---

## Network Validation

### VPC

- [ ] VPC exists or will be created by IPI
- [ ] Subnet CIDRs do not conflict with cluster/service/pod CIDRs

### DNS

- [ ] Cloud DNS managed zone exists for the base domain

```bash
gcloud dns managed-zones list --filter="dnsName=example.com."
```

### Firewall Rules

IPI creates firewall rules automatically. For UPI:
- [ ] Required firewall rules are in place

---

## IAM Validation

The service account requires these roles:

- Compute Admin
- DNS Administrator
- Security Admin
- Service Account Admin
- Service Account Key Admin
- Service Account User
- Storage Admin

```bash
gcloud projects get-iam-policy <project-id> \
  --flatten="bindings[].members" \
  --filter="bindings.members:serviceAccount:<sa-email>" \
  --format="table(bindings.role)"
```

---

## Pull Secret Validation

Same as bare metal (see `baremetal/ipi/`).
