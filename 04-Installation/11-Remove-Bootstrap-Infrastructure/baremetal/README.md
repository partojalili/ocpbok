# Remove Bootstrap Infrastructure — Bare Metal

## IPI Automatic Cleanup

For bare metal IPI, the installer automatically removes the bootstrap node after the control plane is established. No manual action is required.

The installer:

1. Detects that the control plane API is healthy.
2. Powers off the bootstrap node via BMC.
3. Reports bootstrap complete.

## Manual Cleanup (if needed)

If the bootstrap node was not automatically cleaned up:

```bash
# Power off the bootstrap node via IPMI
ipmitool -I lanplus -H <bootstrap-bmc-ip> -U admin -P <password> chassis power off
```

The bootstrap node can be safely repurposed after the cluster installation is complete.
