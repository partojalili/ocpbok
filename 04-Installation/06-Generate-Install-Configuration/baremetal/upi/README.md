# Generate Install Configuration — Bare Metal (UPI)

## install-config.yaml

UPI uses `platform: none` since you manage all infrastructure yourself:

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
    replicas: 0
controlPlane:
  name: master
  replicas: 3
platform:
  none: {}
pullSecret: '<PASTE_PULL_SECRET_JSON_HERE>'
sshKey: '<PASTE_SSH_PUBLIC_KEY_HERE>'
```

> **Note:** Set `compute.replicas: 0` for UPI. Workers are added manually after the control plane is up. The installer does not provision them.

---

## Generate Ignition Configs

```bash
mkdir -p ~/ocp-upi
cp install-config.yaml ~/ocp-upi/

# Back up config (the installer consumes it)
cp ~/ocp-upi/install-config.yaml ~/ocp-upi/install-config.yaml.bak

# Optional: create and review manifests first
openshift-install create manifests --dir=~/ocp-upi

# Generate Ignition configs
openshift-install create ignition-configs --dir=~/ocp-upi
```

Output:
- `bootstrap.ign` — for the bootstrap node
- `master.ign` — for all control plane nodes
- `worker.ign` — for all worker nodes
- `auth/kubeconfig` — cluster admin kubeconfig
- `auth/kubeadmin-password` — initial admin password

---

## Host Ignition Files

Serve the Ignition files via an HTTP server accessible to all nodes:

```bash
sudo cp ~/ocp-upi/*.ign /var/www/html/
sudo chmod 644 /var/www/html/*.ign
```
