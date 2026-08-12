# Generate Install Configuration — Bare Metal (Assisted Installer)

## SaaS Method (console.redhat.com)

Configuration is entered through the web wizard:

1. **Cluster details:** Name, base domain, OpenShift version (4.22), pull secret
2. **Hosts:** Boot servers from Discovery ISO — hosts appear automatically
3. **Networking:** Select cluster-managed or user-managed networking, set API/Ingress VIPs, cluster/service/machine CIDRs
4. **Storage:** Select installation disks per host
5. **Review and install**

No `install-config.yaml` file is needed for the SaaS method.

---

## On-Premises Method (Infrastructure Operator)

Create the following resources on the hub cluster:

### ClusterDeployment

```yaml
apiVersion: hive.openshift.io/v1
kind: ClusterDeployment
metadata:
  name: ocp
  namespace: ocp-cluster
spec:
  baseDomain: example.com
  clusterName: ocp
  platform:
    agentBareMetal:
      agentSelector:
        matchLabels:
          cluster: ocp
  pullSecretRef:
    name: pull-secret
  clusterInstallRef:
    group: extensions.hive.openshift.io
    version: v1beta1
    kind: AgentClusterInstall
    name: ocp
```

### AgentClusterInstall

```yaml
apiVersion: extensions.hive.openshift.io/v1beta1
kind: AgentClusterInstall
metadata:
  name: ocp
  namespace: ocp-cluster
spec:
  clusterDeploymentRef:
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
  provisionRequirements:
    controlPlaneAgents: 3
    workerAgents: 3
  apiVIPs:
    - 10.0.20.100
  ingressVIPs:
    - 10.0.20.101
  sshPublicKey: '<PASTE_SSH_PUBLIC_KEY_HERE>'
  imageSetRef:
    name: openshift-v4.22.0
```

### Apply Configuration

```bash
oc apply -f cluster-deployment.yaml
oc apply -f agent-cluster-install.yaml
```

Monitor progress:

```bash
oc get agentclusterinstall ocp -n ocp-cluster -o jsonpath='{.status.conditions}' | jq .
```
