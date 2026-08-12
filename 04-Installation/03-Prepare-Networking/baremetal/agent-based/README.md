# Prepare Networking — Bare Metal (Agent-Based Installer)

## Network Requirements

The Agent-based Installer does **not** require a provisioning network. All networking is defined in `agent-config.yaml` using NMState before generating the ISO.

| Network | Required | Purpose |
|---------|----------|---------|
| Baremetal / Machine network | Yes | Cluster traffic, API, Ingress |
| Provisioning network | No | Not used |
| BMC network | Optional (only if using virtual media to mount ISO) | Out-of-band management |

## Static IP Configuration

Static IPs are embedded in the agent ISO via `agent-config.yaml`. Each host's `networkConfig` section uses NMState format. See `01-Prepare-Infrastructure/baremetal/agent-based/` for the full `agent-config.yaml` example.

## Firewall Rules

Same ports as IPI (see `03-Prepare-Networking/baremetal/ipi/`). No provisioning network ports needed.

## Connectivity

- The rendezvous node must be reachable by all other nodes on the machine network.
- All nodes must be able to reach DNS and NTP servers.
- If connected, nodes must reach image registries. If disconnected, a local mirror registry must be reachable.
