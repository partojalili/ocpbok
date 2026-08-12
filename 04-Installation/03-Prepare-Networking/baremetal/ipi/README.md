# Prepare Networking — Bare Metal

## DHCP Requirements (Provisioning Network)

The provisioning network must **not** have an external DHCP server. The Metal3 provisioning service (Ironic) manages DHCP on this network.

The baremetal network requires either:
- **Option A (Recommended):** Static IPs defined in `install-config.yaml` (no DHCP needed).
- **Option B:** An external DHCP server with reservations for every node MAC address.

---

## Network Architecture

```
                        +-----------------------+
                        |    External Network   |
                        |    (Corporate LAN)    |
                        +-----------+-----------+
                                    |
                              [ Firewall ]
                                    |
                        +-----------+-----------+
                        |  Baremetal Network     |
                        |  VLAN 20: 10.0.20.0/24|
                        +-----------+-----------+
                                    |
          +------------+------------+------------+------------+
          |            |            |            |            |
     +----+----+  +----+----+  +----+----+  +----+----+  +----+----+
     |Master-0 |  |Master-1 |  |Master-2 |  |Worker-0 |  |Worker-1 | ...
     |.10      |  |.11      |  |.12      |  |.20      |  |.21      |
     +----+----+  +----+----+  +----+----+  +----+----+  +----+----+
          |            |            |            |            |
          +------------+------------+------------+------------+
                                    |
                        +-----------+-----------+
                        | Provisioning Network  |
                        | VLAN 10: 172.22.0.0/24|
                        | (L2 only, no DHCP)    |
                        +-----------------------+

     API VIP:     10.0.20.100
     Ingress VIP: 10.0.20.101
```

## Network Definitions

| Network | VLAN | Subnet | Gateway | Purpose |
|---------|------|--------|---------|---------|
| Provisioning | 10 | 172.22.0.0/24 | None | PXE boot, RHCOS image delivery |
| Baremetal | 20 | 10.0.20.0/24 | 10.0.20.1 | Cluster traffic, API, Ingress |
| BMC/IPMI | 30 | 10.0.10.0/24 | 10.0.10.1 | Out-of-band management |
| Storage (optional) | 40 | 10.0.40.0/24 | None | Dedicated ODF replication traffic |

---

## Provisioning Host Network Configuration

```bash
# Configure provisioning NIC (no IP needed; bridge will be created by installer)
nmcli connection add type bridge con-name provisioning ifname provisioning
nmcli connection add type ethernet con-name eno1-prov ifname eno1 master provisioning
nmcli connection modify provisioning ipv4.addresses 172.22.0.1/24 ipv4.method manual
nmcli connection modify provisioning ipv4.dns "" ipv4.gateway ""
nmcli connection up provisioning

# Configure baremetal NIC
nmcli connection add type bridge con-name baremetal ifname baremetal
nmcli connection add type ethernet con-name eno2-bm ifname eno2 master baremetal
nmcli connection modify baremetal ipv4.addresses 10.0.20.2/24 ipv4.method manual \
  ipv4.gateway 10.0.20.1 ipv4.dns "10.0.20.1"
nmcli connection up baremetal
```

---

## OVN-Kubernetes CNI Configuration

OVN-Kubernetes is the sole supported CNI for OpenShift 4.22 (OpenShift SDN was removed in 4.17). The following parameters are set in `install-config.yaml`:

| Parameter | Value |
|-----------|-------|
| Cluster network CIDR | 10.128.0.0/14 |
| Host prefix | /23 (512 pods per node) |
| Service network CIDR | 172.30.0.0/16 |
| Network type | OVNKubernetes |

---

## Firewall and Connectivity

The following ports must be open between all cluster nodes and from the provisioning host:

| Source | Destination | Port | Protocol | Purpose |
|--------|-------------|------|----------|---------|
| All nodes | All nodes | 6443 | TCP | Kubernetes API |
| All nodes | All nodes | 22623 | TCP | Machine Config Server |
| All nodes | All nodes | 2379-2380 | TCP | etcd |
| All nodes | All nodes | 9000-9999 | TCP | Host-level services |
| All nodes | All nodes | 10250-10259 | TCP | Kubelet, kube-controller |
| All nodes | All nodes | 30000-32767 | TCP | NodePort services |
| All nodes | All nodes | 6081 | UDP | Geneve (OVN-Kubernetes) |
| All nodes | All nodes | 9000-9999 | UDP | Host-level services |
| All nodes | All nodes | 500 | UDP | IPsec IKE |
| All nodes | All nodes | 4500 | UDP | IPsec NAT-T |
| Provisioning host | BMC network | 623 | UDP | IPMI |
| Provisioning host | BMC network | 443 | TCP | Redfish |
| All nodes | Internet / Mirror | 443 | TCP | Container image pull |

### Validation

```bash
# Test API VIP port readiness (should fail before install, confirms no port conflict)
nc -zv 10.0.20.100 6443

# Test outbound connectivity (if connected)
curl -s https://quay.io/v2/ && echo "Quay reachable" || echo "Quay NOT reachable"
curl -s https://registry.redhat.io/v2/ && echo "Registry reachable" || echo "Registry NOT reachable"
```
