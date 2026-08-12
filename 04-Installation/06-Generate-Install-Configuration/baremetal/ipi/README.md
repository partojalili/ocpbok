# Generate Install Configuration — Bare Metal

## Create install-config.yaml

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
    provisioningNetworkInterface: eno1
    provisioningNetworkCIDR: 172.22.0.0/24
    hosts:
      - name: master-0
        role: master
        bmc:
          address: ipmi://10.0.10.11
          username: admin
          password: <REDACTED>
        bootMACAddress: AA:BB:CC:DD:EE:01
        rootDeviceHints:
          deviceName: /dev/sda
      - name: master-1
        role: master
        bmc:
          address: ipmi://10.0.10.12
          username: admin
          password: <REDACTED>
        bootMACAddress: AA:BB:CC:DD:EE:02
        rootDeviceHints:
          deviceName: /dev/sda
      - name: master-2
        role: master
        bmc:
          address: ipmi://10.0.10.13
          username: admin
          password: <REDACTED>
        bootMACAddress: AA:BB:CC:DD:EE:03
        rootDeviceHints:
          deviceName: /dev/sda
      - name: worker-0
        role: worker
        bmc:
          address: ipmi://10.0.10.14
          username: admin
          password: <REDACTED>
        bootMACAddress: AA:BB:CC:DD:EE:04
        rootDeviceHints:
          deviceName: /dev/sda
      - name: worker-1
        role: worker
        bmc:
          address: ipmi://10.0.10.15
          username: admin
          password: <REDACTED>
        bootMACAddress: AA:BB:CC:DD:EE:05
        rootDeviceHints:
          deviceName: /dev/sda
      - name: worker-2
        role: worker
        bmc:
          address: ipmi://10.0.10.16
          username: admin
          password: <REDACTED>
        bootMACAddress: AA:BB:CC:DD:EE:06
        rootDeviceHints:
          deviceName: /dev/sda
pullSecret: '<PASTE_PULL_SECRET_JSON_HERE>'
sshKey: '<PASTE_SSH_PUBLIC_KEY_HERE>'
```

---

## Back Up the Configuration

```bash
mkdir -p ~/ocp-install-backup
cp install-config.yaml ~/ocp-install-backup/install-config.yaml.bak
```

The installer consumes and deletes `install-config.yaml` during the process. Always keep a backup.

---

## Generate Manifests (Optional)

```bash
mkdir -p ~/ocp-install
cp install-config.yaml ~/ocp-install/

# Create the manifests (optional: review before proceeding)
openshift-install create manifests --dir=~/ocp-install
```
