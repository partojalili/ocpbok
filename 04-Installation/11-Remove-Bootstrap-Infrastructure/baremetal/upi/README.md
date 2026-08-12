# Remove Bootstrap Infrastructure — Bare Metal (UPI)

## Manual Cleanup Required

UPI uses a dedicated bootstrap node that must be removed manually after the control plane is self-hosting.

## Steps

### 1. Confirm Bootstrap is Complete

```bash
openshift-install wait-for bootstrap-complete --dir=~/ocp-upi --log-level=info
```

Wait for the message: `It is now safe to remove the bootstrap resources`.

### 2. Remove Bootstrap from Load Balancer

Remove the bootstrap node from the API and Machine Config Server backend pools in your external load balancer.

### 3. Power Off Bootstrap Node

```bash
# Via IPMI
ipmitool -I lanplus -H <bootstrap-bmc-ip> -U admin -P <password> chassis power off

# Or via Redfish
curl -k -u admin:<password> \
  -X POST \
  https://<bootstrap-bmc-ip>/redfish/v1/Systems/1/Actions/ComputerSystem.Reset \
  -H "Content-Type: application/json" \
  -d '{"ResetType":"ForceOff"}'
```

The bootstrap server can be safely repurposed after removal.
