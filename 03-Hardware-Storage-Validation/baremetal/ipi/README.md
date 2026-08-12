# Hardware and Storage Validation — Bare Metal IPI

## Hardware Validation Checklist

### Servers

- [ ] All control plane servers (minimum 3) racked and cabled
- [ ] All worker servers (minimum 3) racked and cabled
- [ ] Provisioning host racked and cabled with dual NICs

### CPU / Memory

| Node Role | Minimum CPU | Minimum RAM | Validated |
|-----------|-------------|-------------|-----------|
| Control Plane | 8 cores | 32 GB | [ ] |
| Worker | 8 cores | 32 GB | [ ] |
| Provisioning Host | 4 cores | 16 GB | [ ] |

### BIOS / Firmware

- [ ] BIOS set to UEFI mode
- [ ] Secure Boot disabled (or configured for RHCOS)
- [ ] Boot order set to PXE/network first on provisioning NIC
- [ ] Firmware updated to vendor-supported version
- [ ] Virtualization extensions enabled (VT-x / AMD-V)
- [ ] Hyper-Threading enabled (recommended)

### BMC / Redfish Validation

```bash
# Test Redfish connectivity (preferred for OCP 4.22)
for BMC_IP in 10.0.10.11 10.0.10.12 10.0.10.13 10.0.10.14 10.0.10.15 10.0.10.16; do
  echo "--- Testing $BMC_IP ---"
  curl -sk -u admin:<password> https://${BMC_IP}/redfish/v1/Systems/1 | jq '.PowerState, .Boot'
done

# Test IPMI connectivity (legacy fallback)
for BMC_IP in 10.0.10.11 10.0.10.12 10.0.10.13 10.0.10.14 10.0.10.15 10.0.10.16; do
  echo "--- Testing $BMC_IP ---"
  ipmitool -I lanplus -H $BMC_IP -U admin -P <password> chassis status
  ipmitool -I lanplus -H $BMC_IP -U admin -P <password> mc info
done
```

- [ ] BMC reachable from provisioning host
- [ ] BMC credentials valid with administrator access
- [ ] Virtual media functional (Redfish)
- [ ] Power on/off via BMC works
- [ ] BMC firmware at supported version

### NIC Validation

- [ ] Provisioning NIC identified (eno1 or equivalent)
- [ ] Baremetal NIC identified (eno2 or equivalent)
- [ ] NIC naming consistent across all nodes
- [ ] Link speed validated (10 GbE minimum)
- [ ] VLAN trunking configured on switch ports (if applicable)

```bash
# From provisioning host — verify NIC naming
ip link show
nmcli device status
```

---

## Storage Validation Checklist

### Boot Disks

- [ ] Boot disk available on all nodes (minimum 120 GB SSD)
- [ ] Boot disk is clean (no existing partitions or OS)

### ODF Disks (Worker Nodes)

- [ ] Each worker has at least 1 raw, unpartitioned disk for ODF (minimum 500 GB)
- [ ] ODF disks have no filesystem signatures
- [ ] Disk type confirmed (NVMe or SSD recommended)

```bash
# Validate disks after nodes are booted (via debug pod)
for node in worker-0 worker-1 worker-2; do
  echo "--- ${node} ---"
  oc debug node/${node}.ocp.example.com -- chroot /host lsblk -d -o NAME,SIZE,TYPE,MODEL
done
```

### Wipe Residual Data (if needed)

```bash
# Only if disks have old partitions or filesystem signatures
oc debug node/worker-0.ocp.example.com -- chroot /host wipefs -a /dev/sdb
```

---

## Network Validation Checklist

- [ ] Provisioning network VLAN exists (L2, no external DHCP)
- [ ] Baremetal network VLAN exists
- [ ] BMC network reachable from provisioning host
- [ ] DNS records created and validated (api, api-int, *.apps, all nodes, PTR)
- [ ] NTP server reachable
- [ ] Firewall rules applied (see `04-Installation/03-Prepare-Networking`)
- [ ] API VIP and Ingress VIP are unused and routable

```bash
# DNS validation
for RECORD in api.ocp.example.com api-int.ocp.example.com test.apps.ocp.example.com; do
  echo "$RECORD -> $(dig +short $RECORD)"
done

# NTP validation
chronyc sources -v

# VIP availability check (should NOT respond)
ping -c 2 10.0.20.100 && echo "WARNING: API VIP already in use!" || echo "API VIP available"
ping -c 2 10.0.20.101 && echo "WARNING: Ingress VIP already in use!" || echo "Ingress VIP available"
```

---

## Pull Secret Validation

```bash
cat pull-secret.json | jq . > /dev/null && echo "Valid JSON" || echo "INVALID JSON"
```

- [ ] Pull secret is valid JSON
- [ ] OpenShift subscription covers the target node count
