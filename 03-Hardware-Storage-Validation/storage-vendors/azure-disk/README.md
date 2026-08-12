# Azure Disk Pre-Installation Validation

**Vendor:** Microsoft Azure  
**CSI Driver:** Azure Disk CSI (built-in with OpenShift on Azure)  
**Operator:** Cluster CSI Driver Operator (managed by OpenShift)

## Requirements

| Component | Requirement |
|-----------|-------------|
| Azure Subscription | Active with sufficient quota |
| Region | Same region as OpenShift cluster |
| Disk Types | Standard_LRS, Premium_LRS, StandardSSD_LRS, UltraSSD_LRS |

## Validation Checklist

### Azure Credentials and Subscription

```bash
# Verify Azure CLI login
az account show

# Verify subscription has active status
az account show --query '{name:name, state:state, id:id}' -o table
```

### Disk Quota Validation

```bash
# Check managed disk quota in the cluster region
az vm list-usage --location <region> -o table | grep -i "managed disks"

# Check Premium SSD quota
az vm list-usage --location <region> -o table | grep -i "premium"
```

### Resource Group and Permissions

```bash
# Identify the cluster's resource group
oc get infrastructure cluster -o jsonpath='{.status.platformStatus.azure.resourceGroupName}'

# Verify service principal has disk permissions:
# Microsoft.Compute/disks/read, write, delete
# Microsoft.Compute/virtualMachines/write (for attach/detach)
az role assignment list --assignee <sp-app-id> --scope /subscriptions/<sub-id> -o table
```

### Availability Zone Validation

```bash
# Get cluster node AZs
oc get nodes -o jsonpath='{range .items[*]}{.metadata.labels.topology\.kubernetes\.io/zone}{"\n"}{end}' | sort -u

# Verify disk types available in those zones
az disk list-skus --location <region> --zone <zone> -o table | head -20
```

### Ultra Disk Validation (if required)

```bash
# Verify Ultra Disk is available in the region/zone
az vm list-skus --location <region> --zone <zone> --resource-type disks --query "[?name=='UltraSSD_LRS']" -o table

# Verify VM size supports Ultra Disk
az vm list-skus --location <region> --zone <zone> --resource-type virtualMachines \
  --query "[?capabilities[?name=='UltraSSDAvailable' && value=='True']].name" -o table
```

### CSI Driver Verification

```bash
# Verify CSI driver pods are running
oc get pods -n openshift-cluster-csi-drivers -l app=azure-disk-csi-driver

# Verify default StorageClass
oc get sc managed-csi
```

## Pre-Installation Summary

| Check | Expected |
|-------|----------|
| Azure subscription active | Status = Enabled |
| Managed disk quota sufficient | Enough headroom for PVs |
| Service principal has disk permissions | Compute/disks CRUD granted |
| Disk types available in cluster zones | Premium/Standard confirmed |
| Ultra Disk support (if needed) | Region/zone/VM size compatible |
| Azure Disk CSI driver running | Pods healthy |
