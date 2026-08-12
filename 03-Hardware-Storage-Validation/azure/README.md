# Hardware and Storage Validation — Azure

## Azure Account Validation

### Credentials

- [ ] Service principal with required permissions, or user-assigned managed identity
- [ ] Subscription ID, tenant ID, client ID, and client secret available

```bash
# Verify Azure credentials
az login --service-principal -u <client-id> -p <client-secret> --tenant <tenant-id>
az account show
```

### Subscription Quotas

Verify sufficient quotas in the target region:

| Resource | Minimum Required | Check Command |
|----------|-----------------|---------------|
| vCPUs (Standard_D8s_v3 family) | 48+ | `az vm list-usage --location eastus -o table` |
| Public IP addresses | 3 | `az network list-usages --location eastus -o table` |
| Network interfaces | 6+ | Same as above |
| Managed disks | 18+ | `az vm list-usage --location eastus -o table` |

### Region Availability

- [ ] Target region supports the required VM sizes
- [ ] Target region has at least 3 availability zones

```bash
az vm list-sizes --location eastus --query "[?name=='Standard_D8s_v3']"
```

---

## Compute Validation

### VM Sizes

| Node Role | Minimum | Recommended |
|-----------|---------|-------------|
| Control Plane | Standard_D8s_v3 (8 vCPU, 32 GB) | Standard_D16s_v3 (16 vCPU, 64 GB) |
| Worker | Standard_D4s_v3 (4 vCPU, 16 GB) | Standard_D8s_v3 (8 vCPU, 32 GB) |

---

## Storage Validation

### Managed Disks

- [ ] Premium SSD (P30 or larger) available for control plane
- [ ] Standard SSD or Premium SSD for workers
- [ ] Azure Disk CSI driver is the default for OCP 4.22

### Azure Files (for RWX)

- [ ] Azure Files available in the target region
- [ ] Storage account created or can be created dynamically

### Azure Blob (for object storage)

- [ ] Blob storage accessible for OADP backups

---

## Network Validation

### VNet

- [ ] VNet created or will be created by IPI
- [ ] VNet CIDR does not conflict with cluster/service/pod CIDRs
- [ ] Subnets planned (control plane subnet, worker subnet)

### DNS

- [ ] Azure DNS zone exists for the base domain (or external DNS with delegation)

```bash
az network dns zone list --query "[?name=='example.com']"
```

### NSG (Network Security Groups)

IPI creates NSGs automatically. For UPI, verify:
- [ ] NSG rules allow all required ports

### Load Balancers

IPI creates Azure Load Balancers automatically (public or internal).

---

## IAM Validation

The service principal requires these role assignments:

- **Contributor** on the subscription or resource group
- **User Access Administrator** (if IPI needs to create role assignments)

```bash
az role assignment list --assignee <client-id> -o table
```

---

## Resource Group

- [ ] Resource group created (or IPI will create one named `<cluster-name>-rg`)

```bash
az group show --name ocp-rg 2>/dev/null && echo "Exists" || echo "Will be created by IPI"
```

---

## Pull Secret Validation

Same as bare metal (see `baremetal/ipi/`).
