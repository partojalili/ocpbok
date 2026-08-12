# Hardware and Storage Validation — Bare Metal (Agent-Based Installer)

## Hardware Validation Checklist

### Servers

- [ ] All target servers racked and cabled (minimum 3 for compact, 6 for standard)
- [ ] Provisioning host available (to generate the agent ISO — does not need network to target servers)
- [ ] No dedicated bootstrap server required (rendezvous node handles bootstrap)

### CPU / Memory

Same requirements as IPI (see `baremetal/ipi/`).

### BIOS / Firmware

- [ ] BIOS set to UEFI mode
- [ ] Boot order set to boot from ISO/USB or virtual media
- [ ] Firmware updated to vendor-supported version

### BMC

BMC is **optional**. Useful for mounting the agent ISO via virtual media, but not required by the installer.

---

## Pre-ISO Generation Validation

Before generating the agent ISO, validate your configuration files:

### install-config.yaml

```bash
# Verify syntax
openshift-install agent create image --dir=~/agent-install --log-level=debug 2>&1 | head -20
```

### agent-config.yaml

- [ ] `rendezvousIP` is set to a valid control plane node IP
- [ ] All hosts have correct MAC addresses
- [ ] NMState network config is valid for each host
- [ ] Roles (master/worker) are correctly assigned

```bash
# Validate NMState config syntax
nmstatectl gc agent-config.yaml 2>&1 || echo "Check NMState config"
```

---

## Storage Validation Checklist

Same as IPI — boot disks and ODF disks (see `baremetal/ipi/`).

---

## Network Validation Checklist

- [ ] Machine network VLAN exists
- [ ] DNS records created and validated
- [ ] NTP server reachable from all nodes
- [ ] Firewall rules applied
- [ ] API VIP and Ingress VIP are unused and routable
- [ ] All nodes can reach each other on the machine network
- [ ] Rendezvous node IP is routable from all other nodes

---

## Agent ISO Validation

- [ ] ISO generated successfully (`agent.x86_64.iso`)
- [ ] ISO size is reasonable (~1-2 GB for connected, larger for disconnected with embedded images)
- [ ] ISO boots successfully on target hardware
- [ ] All nodes discover each other via the rendezvous node

---

## Pull Secret Validation

Same as IPI (see `baremetal/ipi/`).
