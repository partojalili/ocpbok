# Prepare Infrastructure — Bare Metal (UPI)

## Overview

With User-Provisioned Infrastructure (UPI), you are responsible for provisioning and configuring all servers, networking, load balancers, and DNS before running the OpenShift installer. The installer generates Ignition configs that you apply to each server manually.

## Hardware Requirements

Same as IPI — see the IPI requirements for control plane and worker node sizing.

### Additional UPI Requirements

| Component | Requirement |
|-----------|-------------|
| External load balancer | Required (HAProxy, F5, or equivalent) — IPI's built-in keepalived is not available |
| DHCP or static IPs | Customer-managed |
| PXE or ISO boot | Customer-managed |
| DNS | Customer-managed (same records as IPI) |

---

## Prerequisites

- [ ] `openshift-install` binary (4.22) downloaded
- [ ] `oc` CLI downloaded
- [ ] Pull secret available
- [ ] SSH public key for node access
- [ ] All servers racked, cabled, and accessible
- [ ] External load balancer configured (see `04-Prepare-Load-Balancing`)
- [ ] DNS records created (see `02-Prepare-DNS`)
- [ ] RHCOS ISO or PXE images downloaded

---

## Provisioning Host Requirements

| Resource | Requirement |
|----------|-------------|
| OS | RHEL 9.4+ or Fedora 40+ |
| CPU | 4 cores |
| RAM | 8 GB |
| Disk | 50 GB free |
| Packages | `openshift-install`, `oc` |
| Extras | HTTP server to host Ignition files and RHCOS images |

```bash
# Download the installer
OCP_VERSION="4.22.0"
curl -L "https://mirror.openshift.com/pub/openshift-v4/clients/ocp/${OCP_VERSION}/openshift-install-linux.tar.gz" \
  -o openshift-install-linux.tar.gz
tar xvf openshift-install-linux.tar.gz
sudo mv openshift-install /usr/local/bin/

# Download RHCOS images for PXE or ISO boot
curl -L "https://mirror.openshift.com/pub/openshift-v4/dependencies/rhcos/4.22/latest/rhcos-live.x86_64.iso" \
  -o rhcos-live.iso
```

---

## Generate Ignition Configs

```bash
mkdir -p ~/ocp-upi
cp install-config.yaml ~/ocp-upi/

# Generate manifests (review and customize if needed)
openshift-install create manifests --dir=~/ocp-upi

# Generate Ignition configs
openshift-install create ignition-configs --dir=~/ocp-upi
```

This produces:
- `bootstrap.ign` — for the bootstrap node
- `master.ign` — for all control plane nodes
- `worker.ign` — for all worker nodes

### Host Ignition Files on HTTP Server

```bash
sudo cp ~/ocp-upi/*.ign /var/www/html/
sudo chmod 644 /var/www/html/*.ign

# Verify
curl http://<provisioning-host>/bootstrap.ign | head -c 100
```

---

## Boot Servers with RHCOS

### Option A: ISO Boot with Ignition

Boot each server from the RHCOS live ISO, then apply the Ignition config:

```bash
# On the server console after booting the live ISO
sudo coreos-installer install /dev/sda \
  --ignition-url=http://<provisioning-host>/master.ign \
  --insecure-ignition
sudo reboot
```

### Option B: PXE Boot

Configure PXE to serve the RHCOS kernel, initramfs, and rootfs with Ignition URL passed as a kernel argument:

```
coreos.inst.install_dev=/dev/sda
coreos.inst.ignition_url=http://<provisioning-host>/master.ign
```

---

## Key Differences from IPI

| Aspect | IPI | UPI |
|--------|-----|-----|
| BMC/IPMI required | Yes | No |
| Provisioning network | Required | Not required |
| Load balancer | Built-in (keepalived) | External (customer-managed) |
| Node provisioning | Automated via Ironic | Manual (ISO/PXE) |
| Day-2 machine management | Automated (MachineSets) | Manual |
| Bootstrap cleanup | Automatic | Manual |
