# Prepare Infrastructure — Bare Metal (Assisted Installer)

## Overview

The Assisted Installer provides a guided, wizard-driven workflow for bare metal installations. It can be used via the SaaS service at console.redhat.com or deployed on-premises using the Infrastructure Operator (formerly Assisted Installer Operator).

## Hardware Requirements

### Control Plane Nodes (minimum 3, or 3 compact)

| Resource | Minimum | Recommended |
|----------|---------|-------------|
| CPU | 8 cores | 16 cores |
| RAM | 16 GB (compact: 32 GB) | 64 GB |
| Boot disk | 120 GB SSD | 250 GB SSD |
| NIC | 1x 10 GbE | 2x 25 GbE |

### Worker Nodes (minimum 2, or 0 for compact)

| Resource | Minimum | Recommended |
|----------|---------|-------------|
| CPU | 4 cores | 16 cores |
| RAM | 16 GB | 64 GB |
| Boot disk | 120 GB SSD | 250 GB SSD |
| NIC | 1x 10 GbE | 2x 25 GbE |

> **Note:** The Assisted Installer supports compact clusters (3 nodes running both control plane and worker roles), reducing the minimum to 3 nodes total.

---

## Prerequisites

### SaaS-Based (console.redhat.com)

- [ ] Red Hat account with access to console.redhat.com
- [ ] Pull secret downloaded
- [ ] Internet connectivity from all nodes to console.redhat.com and image registries
- [ ] SSH public key for node access

### On-Premises (Infrastructure Operator)

- [ ] An existing OpenShift cluster (hub cluster) to run the Infrastructure Operator
- [ ] MCE (Multicluster Engine) or ACM operator installed on the hub cluster
- [ ] Local mirror registry if disconnected

---

## Provisioning Host Requirements

No dedicated provisioning host is required. The Assisted Installer service generates a Discovery ISO that you boot on each server. You need:

- A workstation with a browser (for SaaS) or `oc` CLI access (for on-premises)
- Ability to mount ISOs via BMC virtual media, USB, or PXE

---

## Generate Discovery ISO

### SaaS Method

1. Log in to [console.redhat.com/openshift](https://console.redhat.com/openshift)
2. Click **Create cluster** → **Datacenter** → **Bare Metal (x86_64)**
3. Enter cluster name, base domain, and OpenShift version (4.22)
4. Download the Discovery ISO
5. Boot all target servers from the ISO

### On-Premises Method (InfraEnv)

```bash
cat <<EOF | oc apply -f -
apiVersion: agent-install.openshift.io/v1beta1
kind: InfraEnv
metadata:
  name: ocp-infraenv
  namespace: ocp-cluster
spec:
  clusterRef:
    name: ocp
    namespace: ocp-cluster
  pullSecretRef:
    name: pull-secret
  sshAuthorizedKey: '<PASTE_SSH_PUBLIC_KEY_HERE>'
  nmStateConfigLabelSelector:
    matchLabels:
      cluster: ocp
EOF

# Retrieve the ISO download URL
oc get infraenv ocp-infraenv -n ocp-cluster -o jsonpath='{.status.isoDownloadURL}'
```

---

## Boot Servers from Discovery ISO

For each server:

1. Mount the Discovery ISO via BMC virtual media (Redfish preferred) or USB.
2. Set boot order to boot from the ISO.
3. Power on the server.
4. The host registers with the Assisted Installer service and appears in the inventory.

```bash
# Redfish virtual media mount example
curl -k -u admin:<password> \
  -X POST \
  -H "Content-Type: application/json" \
  -d '{"Image":"http://<http-server>/discovery.iso","Inserted":true}' \
  https://<bmc-ip>/redfish/v1/Managers/1/VirtualMedia/CD/Actions/VirtualMedia.InsertMedia
```

---

## Host Discovery and Validation

After booting, the Assisted Installer automatically:

- Discovers hardware (CPU, RAM, disks, NICs)
- Validates minimum requirements
- Detects network connectivity between hosts
- Reports host status in the UI or via API

### Validation Checklist

- [ ] All hosts appear in the inventory with status **Ready**
- [ ] Hardware meets minimum requirements (no validation errors)
- [ ] Network connectivity between all hosts is confirmed
- [ ] Host roles (control-plane / worker) are assigned correctly
- [ ] Hostnames and IPs are correct
