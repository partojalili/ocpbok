# Prepare Infrastructure — Disconnected / Air-Gapped

This is a cross-cutting concern that applies to any installation method (IPI, UPI, Assisted Installer, Agent-based) when the target environment has no internet access.

---

## Mirror Registry

A local container registry is required to host all OpenShift images. This registry must be accessible from all cluster nodes.

### Mirror Registry Options

| Option | Description |
|--------|-------------|
| `mirror-registry` (Red Hat) | Purpose-built, single-command deployment of Quay on a RHEL host |
| Quay | Full-featured enterprise registry |
| Harbor | Open-source registry with image signing |
| JFrog Artifactory | Enterprise artifact manager |
| Docker Registry v2 | Minimal, lightweight option |

### Install mirror-registry (Recommended for POC)

```bash
# Download mirror-registry
curl -L "https://developers.redhat.com/content-gateway/rest/mirror/pub/openshift-v4/clients/mirror-registry/latest/mirror-registry.tar.gz" \
  -o mirror-registry.tar.gz
tar xvf mirror-registry.tar.gz

# Install (creates a Quay instance on the local host)
./mirror-registry install \
  --quayHostname registry.example.com \
  --quayRoot /opt/quay
```

The install outputs credentials and the CA certificate. Save both.

### Mirror Registry Requirements

| Resource | Requirement |
|----------|-------------|
| CPU | 4 cores |
| RAM | 8 GB |
| Disk | 500 GB+ (depends on number of operators mirrored) |
| TLS | Valid certificate or self-signed CA trusted by all nodes |
| DNS | Resolvable hostname from all cluster nodes |

---

## Mirror OpenShift Images with oc-mirror

`oc-mirror` is the supported tool for mirroring OpenShift release images and Operator catalogs.

### Install oc-mirror

```bash
curl -L "https://mirror.openshift.com/pub/openshift-v4/clients/ocp/4.22.0/oc-mirror.tar.gz" \
  -o oc-mirror.tar.gz
tar xvf oc-mirror.tar.gz
chmod +x oc-mirror
sudo mv oc-mirror /usr/local/bin/
```

### Create ImageSetConfiguration

```yaml
apiVersion: mirror.openshift.io/v2alpha1
kind: ImageSetConfiguration
mirror:
  platform:
    channels:
      - name: stable-4.22
        minVersion: 4.22.0
        maxVersion: 4.22.0
  operators:
    - catalog: registry.redhat.io/redhat/redhat-operator-index:v4.22
      packages:
        - name: odf-operator
        - name: local-storage-operator
        - name: oadp-operator
        - name: cluster-logging
        - name: loki-operator
  additionalImages:
    - name: registry.redhat.io/ubi9/ubi:latest
    - name: registry.access.redhat.com/ubi9/httpd-24:latest
```

### Run the Mirror

```bash
# First run: mirror from internet to disk (on a connected host)
oc-mirror --config=imageset-config.yaml file:///opt/mirror-output

# Transfer the output to the disconnected environment (USB, SCP, etc.)

# Second run: mirror from disk to the local registry (on the disconnected host)
oc-mirror --from=/opt/mirror-output docker://registry.example.com:8443
```

This generates:
- `ImageDigestMirrorSet` (IDMS) manifests — apply post-install or embed in the installer
- `CatalogSource` manifests — for OperatorHub in disconnected mode

---

## Disconnected Pull Secret

Create a combined pull secret that includes both the Red Hat pull secret (for oc-mirror on the connected side) and the mirror registry credentials (for the disconnected cluster):

```bash
# Add mirror registry credentials to pull secret
REG_USER="init"
REG_PASS="<mirror-registry-password>"
REG_HOST="registry.example.com:8443"

# Encode credentials
AUTH=$(echo -n "${REG_USER}:${REG_PASS}" | base64 -w0)

# Merge into pull secret
cat pull-secret.json | jq --arg host "$REG_HOST" --arg auth "$AUTH" \
  '.auths += {($host): {"auth": $auth}}' > merged-pull-secret.json
```

---

## Trust the Mirror Registry CA

If the mirror registry uses a self-signed CA, the CA certificate must be trusted by the installer and all cluster nodes.

### Add CA to install-config.yaml

```yaml
additionalTrustBundle: |
  -----BEGIN CERTIFICATE-----
  <paste mirror registry CA cert here>
  -----END CERTIFICATE-----
```

### Add CA to the provisioning host

```bash
sudo cp registry-ca.crt /etc/pki/ca-trust/source/anchors/
sudo update-ca-trust
```

---

## Checklist

- [ ] Mirror registry deployed and accessible
- [ ] OpenShift release images mirrored (oc-mirror)
- [ ] Required Operator catalogs mirrored
- [ ] Combined pull secret created (Red Hat + mirror registry)
- [ ] Mirror registry CA certificate available
- [ ] DNS resolves the mirror registry hostname from all nodes
- [ ] Mirror registry has sufficient disk space
- [ ] IDMS and CatalogSource manifests generated
