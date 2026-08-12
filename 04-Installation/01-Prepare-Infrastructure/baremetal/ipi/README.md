# Prepare Infrastructure — Bare Metal

## Hardware Requirements

### Control Plane Nodes (minimum 3)

| Resource | Minimum | Recommended |
|----------|---------|-------------|
| CPU | 8 vCPU / physical cores | 16 cores |
| RAM | 32 GB | 64 GB |
| Boot disk | 120 GB SSD | 250 GB SSD |
| NIC | 2x 10 GbE (provisioning + baremetal) | 2x 25 GbE |
| BMC | IPMI 2.0 / Redfish | Redfish preferred |

### Worker Nodes (minimum 3)

| Resource | Minimum | Recommended |
|----------|---------|-------------|
| CPU | 8 vCPU / physical cores | 32 cores |
| RAM | 32 GB | 128 GB |
| Boot disk | 120 GB SSD | 250 GB SSD |
| ODF disks | 1x 500 GB NVMe/SSD (raw, unpartitioned) | 2x 1 TB NVMe |
| NIC | 2x 10 GbE | 2x 25 GbE |
| BMC | IPMI 2.0 / Redfish | Redfish preferred |

---

## BMC/IPMI Validation

Run these checks on every node before starting installation:

```bash
# Test IPMI connectivity (replace with actual BMC IPs and credentials)
for BMC_IP in 10.0.10.11 10.0.10.12 10.0.10.13 10.0.10.14 10.0.10.15 10.0.10.16; do
  echo "--- Testing $BMC_IP ---"
  ipmitool -I lanplus -H $BMC_IP -U admin -P <password> chassis status
  ipmitool -I lanplus -H $BMC_IP -U admin -P <password> mc info
done
```

Verify the following for each node:

- [ ] BMC is reachable from the provisioning host.
- [ ] IPMI credentials are valid and have administrator-level access.
- [ ] Boot order can be set to PXE/network first.
- [ ] Virtual media (if using Redfish) is functional.
- [ ] BMC firmware is at a supported version (check vendor compatibility matrix).

---

## NTP Validation

```bash
# Verify NTP is reachable from the provisioning host
chronyc sources -v
# Confirm time is synchronized
chronyc tracking
```

All nodes must be able to reach the NTP server(s) after booting. If the environment is air-gapped, an internal NTP source is mandatory.

---

## Pull Secret and Subscription

- [ ] Download pull secret from [console.redhat.com](https://console.redhat.com/openshift/install/pull-secret).
- [ ] Verify the pull secret is valid JSON: `cat pull-secret.json | jq .`
- [ ] Confirm the OpenShift subscription covers the target node count and support level.

---

## Provisioning Host Requirements

The provisioning host is the machine from which you run the installer. It must have:

| Resource | Requirement |
|----------|-------------|
| OS | RHEL 9.4+ or Fedora 40+ |
| CPU | 4 cores |
| RAM | 16 GB |
| Disk | 100 GB free |
| NICs | 2 (one on provisioning network, one on baremetal network) |
| Packages | `podman`, `firewalld`, `ipmitool`, `nmcli`, `nmstate` |

```bash
# Install required packages
sudo dnf install -y podman firewalld ipmitool NetworkManager nmstate

# Download the installer
OCP_VERSION="4.22.0"
curl -L "https://mirror.openshift.com/pub/openshift-v4/clients/ocp/${OCP_VERSION}/openshift-install-linux.tar.gz" \
  -o openshift-install-linux.tar.gz
tar xvf openshift-install-linux.tar.gz
sudo mv openshift-install /usr/local/bin/

# Download oc client
curl -L "https://mirror.openshift.com/pub/openshift-v4/clients/ocp/${OCP_VERSION}/openshift-client-linux.tar.gz" \
  -o openshift-client-linux.tar.gz
tar xvf openshift-client-linux.tar.gz
sudo mv oc kubectl /usr/local/bin/

# Verify
openshift-install version
oc version --client
```

---

## Generate SSH Key

```bash
ssh-keygen -t ed25519 -N '' -f ~/.ssh/ocp-key
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/ocp-key
```
