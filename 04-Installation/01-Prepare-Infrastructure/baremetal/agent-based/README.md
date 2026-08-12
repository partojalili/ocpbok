# Prepare Infrastructure — Bare Metal (Agent-Based Installer)

## Overview

The Agent-based Installer embeds the Assisted Installer logic into a single bootable ISO. No external service or hub cluster is required. All configuration is defined in YAML manifests before generating the ISO. This is the preferred method for fully disconnected and air-gapped environments.

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

---

## Prerequisites

- [ ] `openshift-install` binary (4.22) downloaded
- [ ] Pull secret available (or mirror registry pull secret for disconnected)
- [ ] SSH public key for node access
- [ ] NMState-based network configuration for each host (if using static IPs)
- [ ] Mirror registry populated (if disconnected)

---

## Provisioning Host Requirements

The provisioning host generates the agent ISO. It does not need network connectivity to the target servers during installation (the ISO is self-contained).

| Resource | Requirement |
|----------|-------------|
| OS | RHEL 9.4+ or Fedora 40+ |
| CPU | 4 cores |
| RAM | 8 GB |
| Disk | 50 GB free |
| Packages | `openshift-install`, `nmstatectl` |

```bash
# Download the installer
OCP_VERSION="4.22.0"
curl -L "https://mirror.openshift.com/pub/openshift-v4/clients/ocp/${OCP_VERSION}/openshift-install-linux.tar.gz" \
  -o openshift-install-linux.tar.gz
tar xvf openshift-install-linux.tar.gz
sudo mv openshift-install /usr/local/bin/

# Install nmstatectl for network config validation
sudo dnf install -y nmstate
```

---

## Prepare Configuration Files

The Agent-based Installer requires two YAML files:

### install-config.yaml

Defines the cluster configuration (same as other methods — see `06-Generate-Install-Configuration`).

### agent-config.yaml

Defines per-host configuration including BMC credentials and network:

```yaml
apiVersion: v1beta1
kind: AgentConfig
metadata:
  name: ocp
rendezvousIP: 10.0.20.10
hosts:
  - hostname: master-0
    role: master
    interfaces:
      - name: eno1
        macAddress: AA:BB:CC:DD:EE:01
    networkConfig:
      interfaces:
        - name: eno1
          type: ethernet
          state: up
          ipv4:
            enabled: true
            address:
              - ip: 10.0.20.10
                prefix-length: 24
            dhcp: false
      dns-resolver:
        config:
          server:
            - 10.0.20.1
      routes:
        config:
          - destination: 0.0.0.0/0
            next-hop-address: 10.0.20.1
            next-hop-interface: eno1
  # Repeat for each host...
```

> **rendezvousIP:** The IP of the node that will coordinate the installation. Typically the first control plane node.

---

## Generate the Agent ISO

```bash
mkdir -p ~/agent-install
cp install-config.yaml agent-config.yaml ~/agent-install/

openshift-install agent create image --dir=~/agent-install
```

This produces `~/agent-install/agent.x86_64.iso`.

---

## Boot Servers from Agent ISO

Boot all target servers from the generated ISO via:

- BMC virtual media (Redfish preferred)
- USB drive
- PXE (extract the ISO contents)

The rendezvous node coordinates the installation. All nodes communicate locally — no external service is needed.
