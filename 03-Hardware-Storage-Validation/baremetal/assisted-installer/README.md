# Hardware and Storage Validation — Bare Metal (Assisted Installer)

## Hardware Validation Checklist

### Servers

- [ ] All target servers racked and cabled (minimum 3 for compact, 6 for standard)
- [ ] No dedicated provisioning host required
- [ ] No dedicated bootstrap server required

### CPU / Memory

Same requirements as IPI. Compact clusters (3 nodes with combined control-plane + worker role) require 32 GB RAM minimum per node.

### BIOS / Firmware

- [ ] BIOS set to UEFI mode
- [ ] Boot order set to boot from virtual media / USB / PXE
- [ ] Firmware updated to vendor-supported version

### BMC

BMC is **optional** but helpful for mounting the Discovery ISO via virtual media:

- [ ] Redfish virtual media functional (if mounting ISO remotely)

---

## Assisted Installer Service Validation

### SaaS

- [ ] Red Hat account has access to console.redhat.com
- [ ] Workstation has browser access to console.redhat.com
- [ ] All servers can reach the Assisted Installer service URLs:
  - `api.openshift.com`
  - `console.redhat.com`
  - Image registries (quay.io, registry.redhat.io)

### On-Premises

- [ ] Hub cluster is running and healthy
- [ ] MCE or ACM operator is installed
- [ ] Infrastructure Operator (Assisted Installer) is deployed
- [ ] `AgentServiceConfig` resource is created

```bash
# Verify Infrastructure Operator is running
oc get pods -n multicluster-engine | grep assisted
# or
oc get agentserviceconfig agent -o jsonpath='{.status.conditions}' | jq .
```

---

## Storage Validation Checklist

Same as IPI — boot disks and ODF disks (see `baremetal/ipi/`).

---

## Network Validation Checklist

- [ ] Machine network VLAN exists
- [ ] DNS records created and validated
- [ ] NTP server reachable from all nodes
- [ ] Firewall rules applied (no provisioning network ports needed)
- [ ] API VIP and Ingress VIP are unused and routable

---

## Discovery ISO Validation

After generating the Discovery ISO:

- [ ] ISO successfully mounts on target servers (virtual media or USB)
- [ ] Servers boot from the ISO without errors
- [ ] All hosts appear in the Assisted Installer inventory
- [ ] All hosts pass the built-in hardware and network validation checks

The Assisted Installer performs its own pre-flight validation — it will flag any hosts that don't meet requirements before allowing installation to proceed.

---

## Pull Secret Validation

Same as IPI (see `baremetal/ipi/`).
