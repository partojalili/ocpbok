# Prepare Networking — Bare Metal (UPI)

## Network Requirements

UPI does **not** require a provisioning network. You manage all networking yourself.

| Network | Required | Purpose |
|---------|----------|---------|
| Baremetal / Machine network | Yes | Cluster traffic, API, Ingress |
| Provisioning network | No | Not used |
| BMC network | Optional | Out-of-band management |

## IP Assignment

You are responsible for assigning IPs to each node. Options:

- **Static IPs:** Configure on each node via RHCOS Ignition (using kernel arguments or afterburn).
- **DHCP reservations:** Configure your DHCP server with MAC-to-IP reservations for each node.

## Firewall Rules

Same ports as IPI (see `03-Prepare-Networking/baremetal/ipi/`). No provisioning network ports needed.

## Key Difference

UPI does not deploy keepalived or HAProxy automatically. You must configure an external load balancer to handle API and Ingress traffic before booting nodes.
