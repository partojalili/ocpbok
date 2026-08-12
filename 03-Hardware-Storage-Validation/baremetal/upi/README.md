# Hardware and Storage Validation — Bare Metal UPI

## Hardware Validation Checklist

### Servers

- [ ] All control plane servers (minimum 3) racked and cabled
- [ ] All worker servers (minimum 2) racked and cabled
- [ ] Bootstrap server available (temporary, can be repurposed after install)
- [ ] Provisioning host available (to run the installer and host Ignition files)

### CPU / Memory

Same requirements as IPI (see `baremetal/ipi/`).

### BIOS / Firmware

- [ ] BIOS set to UEFI mode
- [ ] Secure Boot disabled (or configured for RHCOS)
- [ ] Boot order set to boot from ISO/USB or PXE
- [ ] Firmware updated to vendor-supported version

### BMC

BMC is **optional** for UPI. It is helpful for remote power management but not required by the installer.

- [ ] BMC reachable (if available, for remote management convenience)

### NIC Validation

- [ ] At least 1 NIC per node on the machine network
- [ ] No provisioning NIC required (no provisioning network needed)

---

## Storage Validation Checklist

Same as IPI — boot disks and ODF disks (see `baremetal/ipi/`).

---

## Network Validation Checklist

- [ ] Machine network VLAN exists
- [ ] DNS records created and validated
- [ ] **External load balancer configured and tested** (API + Ingress)
- [ ] NTP server reachable
- [ ] Firewall rules applied
- [ ] HTTP server running on provisioning host to serve Ignition files and RHCOS images

```bash
# Verify HTTP server is serving Ignition files
curl -s -o /dev/null -w "%{http_code}" http://<provisioning-host>/bootstrap.ign
# Expected: 200

# Verify load balancer health checks
curl -sk https://api.ocp.example.com:6443/readyz
# Expected: connection refused (before install) — confirms LB is routing to the right port
```

---

## RHCOS Images Validation

- [ ] RHCOS live ISO downloaded for the target OCP version
- [ ] RHCOS PXE images downloaded (if using PXE boot)

```bash
# Verify RHCOS ISO is available
ls -lh rhcos-live.iso
```

---

## Pull Secret Validation

Same as IPI (see `baremetal/ipi/`).
