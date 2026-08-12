# Installation Types Overview — OpenShift 4.22

Red Hat OpenShift Container Platform supports multiple installation topologies. The choice depends on infrastructure ownership, automation requirements, and operational model.

## Installer-Provisioned Infrastructure (IPI)

- The OpenShift installer provisions and manages the underlying infrastructure automatically.
- Supported platforms: AWS, Azure, GCP, VMware vSphere, Bare Metal, OpenStack, Nutanix.
- The installer handles machine provisioning, OS installation, and cluster bootstrapping.
- Best suited for environments where full automation of the infrastructure lifecycle is desired.
- Bare metal IPI uses the Ironic provisioning service to manage server lifecycle via BMC (Redfish/IPMI).
- As of OpenShift 4.22, Redfish is the preferred and default BMC protocol; IPMI remains supported but is considered legacy.

## User-Provisioned Infrastructure (UPI)

- The customer provisions infrastructure manually or with their own tooling before running the installer.
- The installer generates Ignition configs; the customer is responsible for booting machines with those configs.
- Required when the target platform is not natively supported by IPI or when custom infrastructure constraints exist.
- Provides the most flexibility but requires deeper operational knowledge.

## Assisted Installer (AI)

- A SaaS-based or on-premises service (via Assisted Installer Operator / Infrastructure Operator) that provides a guided, wizard-driven installation workflow.
- Performs host discovery, validation checks, and generates ISOs for booting target nodes.
- Supports bare metal, vSphere, Nutanix, and external platform (platform: none) environments.
- Well suited for smaller deployments, edge, and disconnected environments when paired with the on-premises variant.
- In OpenShift 4.22, the Assisted Installer supports dual-stack networking, OVN-Kubernetes with IPsec, and static IP configurations natively.

## Agent-Based Installer

- Embeds the Assisted Installer logic into a bootable ISO, removing the need for an external service.
- Fully disconnected-capable from the start.
- Configuration is defined in YAML manifests (install-config.yaml and agent-config.yaml).
- Ideal for air-gapped environments and edge deployments.

## Hosted Control Planes (HCP)

- Control plane components run as pods on a management cluster rather than on dedicated machines.
- Worker nodes are provisioned separately and join the hosted cluster.
- Reduces infrastructure cost for the control plane and enables rapid cluster provisioning (sub-10-minute control planes).
- GA on AWS, bare metal (with Agent provider), KubeVirt, and Azure.
- In OpenShift 4.22, HCP is fully integrated into the MCE/ACM operator and supports automated node pool scaling, disconnected deployments, and custom machine CIDR configurations.

## Single Node OpenShift (SNO)

- Runs the full OpenShift stack (control plane + worker) on a single node.
- Designed for edge, retail, and remote/branch office use cases.
- Can be installed via the Assisted Installer, Agent-based Installer, or IPI.

## Comparison Matrix

| Criteria                  | IPI            | UPI            | Assisted       | Agent-Based    | HCP            | SNO            |
|---------------------------|----------------|----------------|----------------|----------------|----------------|----------------|
| Infrastructure automation | Full           | None           | Partial        | Partial        | Full (CP)      | Varies         |
| Disconnected support      | Limited        | Yes            | Yes (on-prem)  | Yes            | Limited        | Yes            |
| Minimum nodes             | 6 (3CP + 3W)  | 6 (3CP + 3W)   | 3 (compact)    | 3 (compact)    | 2 (mgmt + 1W)  | 1              |
| Complexity                | Low            | High           | Low            | Medium         | Medium         | Low            |
| Day-2 machine management  | Automated      | Manual         | Manual         | Manual         | Automated (W)  | N/A            |
| Best for                  | Standard DC    | Custom infra   | Quick start    | Air-gapped     | Multi-tenancy  | Edge           |
