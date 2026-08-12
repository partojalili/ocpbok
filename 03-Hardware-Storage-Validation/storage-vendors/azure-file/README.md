# Azure File Pre-Installation Validation

**Vendor:** Microsoft Azure  
**CSI Driver:** Azure File CSI (built-in with OpenShift on Azure)  
**Operator:** Cluster CSI Driver Operator (managed by OpenShift)

## Requirements

| Component | Requirement |
|-----------|-------------|
| Azure Subscription | Active with storage account permissions |
| Storage Account | General-purpose v2 or Premium |
| Networking | Cluster nodes can reach storage account endpoints |

## Validation Checklist

### Storage Account Validation

```bash
# List existing storage accounts in the cluster resource group
RG=$(oc get infrastructure cluster -o jsonpath='{.status.platformStatus.azure.resourceGroupName}')
az storage account list --resource-group $RG -o table

# If creating a new storage account for Azure Files:
az storage account check-name --name <proposed-name>
```

### Quota and Limits

```bash
# Check storage account limits
az storage account show --name <account-name> --resource-group $RG \
  --query '{kind:kind, sku:sku.name, largeFileShares:largeFileSharesState}'

# Check file share quota
az storage share-rm list --storage-account <account-name> --resource-group $RG -o table
```

### Network Access Validation

```bash
# Check storage account network rules
az storage account show --name <account-name> --resource-group $RG \
  --query 'networkRuleSet.defaultAction'

# If using service endpoints or private endpoints:
az storage account network-rule list --account-name <account-name> --resource-group $RG -o table

# Verify SMB port (445) from cluster nodes
oc debug node/<node-name> -- chroot /host nc -zv <account-name>.file.core.windows.net 445

# Verify NFS port (2049) if using NFS protocol
oc debug node/<node-name> -- chroot /host nc -zv <account-name>.file.core.windows.net 2049
```

### NFS Protocol Validation (if required)

```bash
# NFS Azure Files requires:
# - Premium storage account
# - Large file shares enabled
# - No private endpoint restrictions on NFS

# Verify Premium storage account exists or can be created
az storage account check-name --name <proposed-premium-name>
```

### Service Principal Permissions

```bash
# Azure File CSI needs:
# Microsoft.Storage/storageAccounts/read
# Microsoft.Storage/storageAccounts/listKeys/action
# Microsoft.Storage/storageAccounts/fileServices/shares/read, write, delete

az role assignment list --assignee <sp-app-id> -o table | grep -i storage
```

### CSI Driver Verification

```bash
# Verify CSI driver pods
oc get pods -n openshift-cluster-csi-drivers -l app=azure-file-csi-driver

# Verify default StorageClass
oc get sc azurefile-csi
```

## Pre-Installation Summary

| Check | Expected |
|-------|----------|
| Storage account accessible | Account exists or can be created |
| SMB port 445 reachable | From cluster nodes |
| NFS port 2049 reachable (if NFS) | Premium account configured |
| Network rules allow cluster access | Firewall/VNet rules set |
| Service principal has storage permissions | Storage CRUD granted |
| Azure File CSI driver running | Pods healthy |
