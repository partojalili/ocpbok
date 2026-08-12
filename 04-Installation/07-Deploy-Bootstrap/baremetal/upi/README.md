# Deploy Bootstrap — Bare Metal (UPI)

## Bootstrap Node

UPI requires a **dedicated bootstrap node** that you provision manually.

### Boot the Bootstrap Node

1. Boot the bootstrap server from the RHCOS live ISO.
2. Apply the bootstrap Ignition config:

```bash
sudo coreos-installer install /dev/sda \
  --ignition-url=http://<provisioning-host>/bootstrap.ign \
  --insecure-ignition
sudo reboot
```

3. Ensure the bootstrap node is included in the API load balancer backend pool.

## Monitor Bootstrap Progress

```bash
openshift-install wait-for bootstrap-complete --dir=~/ocp-upi --log-level=info
```

You can also SSH into the bootstrap node to watch services:

```bash
ssh core@<bootstrap-ip>
journalctl -b -f -u release-image.service -u bootkube.service
```

### Expected Timeline

- Bootstrap: ~20-30 minutes

## What Happens

1. The bootstrap node starts a temporary etcd and API server.
2. Control plane nodes pull their config from the bootstrap Machine Config Server (port 22623).
3. Once the control plane is self-hosting, bootstrap is no longer needed.
