# Prepare Networking — Bare Metal (Assisted Installer)

## Network Requirements

The Assisted Installer does **not** require a separate provisioning network. All nodes communicate over the baremetal (machine) network.

| Network | Required | Purpose |
|---------|----------|---------|
| Baremetal / Machine network | Yes | Cluster traffic, API, Ingress |
| Provisioning network | No | Not used by the Assisted Installer |
| BMC network | Yes (for virtual media boot) | Out-of-band management |

## Static IP Configuration

Static IPs are configured via NMState in the Assisted Installer UI or via `NMStateConfig` manifests (on-premises).

```yaml
apiVersion: agent-install.openshift.io/v1beta1
kind: NMStateConfig
metadata:
  name: master-0
  namespace: ocp-cluster
  labels:
    cluster: ocp
spec:
  config:
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
  interfaces:
    - name: eno1
      macAddress: "AA:BB:CC:DD:EE:01"
```

## Firewall Rules

Same ports as IPI (see `03-Prepare-Networking/baremetal/ipi/`). The only difference is that provisioning network ports (DHCP/TFTP) are not needed.

## Connectivity Validation

The Assisted Installer performs automatic network connectivity checks between all discovered hosts before allowing installation to proceed.
