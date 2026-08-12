# Phase 3 — Hardware and Storage Validation

## Installation Architecture

Build a decision tree for installation.

### Installation Methods

Understand:

- IPI
- UPI
- Agent-based Installer
- Assisted Installer
- PXE
- Installer-provisioned infrastructure
- User-provisioned infrastructure

Then answer: **Which installation method should be used for this infrastructure?**

```
Hardware
   |
   +-- Cloud
   |     |
   |     +-- IPI
   |
   +-- VMware
   |     |
   |     +-- IPI / UPI
   |
   +-- Bare Metal
         |
         +-- IPI
         +-- UPI
         +-- Agent-based
         +-- Assisted Installer
```

---

## Installation Prerequisites Checklist

### Infrastructure

- [ ] Servers available
- [ ] CPU/memory validated
- [ ] Disk configuration validated
- [ ] Firmware validated
- [ ] BIOS configured
- [ ] Network configured
- [ ] VLANs configured
- [ ] DNS configured
- [ ] NTP configured
- [ ] Load balancer configured
- [ ] Firewall rules configured
- [ ] Proxy configured, if required

### OpenShift

- [ ] OpenShift version selected (4.22)
- [ ] Pull secret available
- [ ] Installation method selected
- [ ] Install configuration prepared
- [ ] Machine configuration prepared
- [ ] Storage architecture selected
- [ ] Network architecture validated
