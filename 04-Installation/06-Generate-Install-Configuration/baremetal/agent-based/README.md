# Generate Install Configuration — Bare Metal (Agent-Based Installer)

## Required Files

The Agent-based Installer requires two YAML files in the same directory:

1. `install-config.yaml` — cluster-level configuration
2. `agent-config.yaml` — per-host configuration (BMC, network, roles)

---

## install-config.yaml

```yaml
apiVersion: v1
baseDomain: example.com
metadata:
  name: ocp
networking:
  networkType: OVNKubernetes
  clusterNetwork:
    - cidr: 10.128.0.0/14
      hostPrefix: 23
  serviceNetwork:
    - 172.30.0.0/16
  machineNetwork:
    - cidr: 10.0.20.0/24
compute:
  - name: worker
    replicas: 3
controlPlane:
  name: master
  replicas: 3
platform:
  baremetal:
    apiVIPs:
      - 10.0.20.100
    ingressVIPs:
      - 10.0.20.101
pullSecret: '<PASTE_PULL_SECRET_JSON_HERE>'
sshKey: '<PASTE_SSH_PUBLIC_KEY_HERE>'
```

> **Note:** The `platform.baremetal.hosts` section is **not** used in the Agent-based Installer. Host definitions go in `agent-config.yaml` instead.

---

## agent-config.yaml

See `01-Prepare-Infrastructure/baremetal/agent-based/` for the full example with per-host NMState network configuration.

---

## Generate the Agent ISO

```bash
mkdir -p ~/agent-install
cp install-config.yaml agent-config.yaml ~/agent-install/

# Back up configs (the installer consumes them)
cp ~/agent-install/install-config.yaml ~/agent-install/install-config.yaml.bak
cp ~/agent-install/agent-config.yaml ~/agent-install/agent-config.yaml.bak

# Generate the ISO
openshift-install agent create image --dir=~/agent-install
```

Output: `~/agent-install/agent.x86_64.iso`

---

## Optional: Include Extra Manifests

Place additional manifests in `~/agent-install/openshift/` before generating the ISO. These are applied during installation:

```bash
mkdir -p ~/agent-install/openshift/
# Example: add a MachineConfig, chrony config, etc.
cp my-custom-machineconfig.yaml ~/agent-install/openshift/
```
