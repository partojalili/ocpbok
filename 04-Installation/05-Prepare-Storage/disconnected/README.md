# Prepare Storage — Disconnected / Air-Gapped

## ODF in Disconnected Environments

ODF Operator images must be included in the `oc-mirror` image set. Ensure these packages are mirrored:

- `odf-operator`
- `local-storage-operator`
- `odf-csi-addons-operator`
- `mcg-operator` (if using NooBaa for object storage)

## Verify ODF Images Are Available

After applying the CatalogSource:

```bash
# Confirm ODF packages appear in the mirrored catalog
oc get packagemanifest -n openshift-marketplace | grep -E 'odf|local-storage'
```

If packages are missing, update the `ImageSetConfiguration` and re-run `oc-mirror`.

## ODF Installation

The ODF installation procedure is the same as connected environments (see `05-Prepare-Storage/baremetal/`). The only difference is that images are pulled from the mirror registry instead of the internet.

## Backup Storage (OADP)

If using OADP for backups, the backup target must be local:

- S3-compatible object storage on the mirror registry host or a dedicated MinIO/Ceph RGW instance
- NFS share for Restic-based backups

External cloud storage (AWS S3, Azure Blob) is not available in air-gapped environments.
