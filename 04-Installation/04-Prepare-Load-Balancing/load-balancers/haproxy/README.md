# HAProxy (External)

**Type:** Open-source software load balancer  
**Purpose:** API and Ingress load balancing for UPI and non-IPI installations

## Overview

HAProxy is the most common external load balancer for OpenShift bare metal and vSphere UPI installations. It provides Layer 4 TCP load balancing for the API, Machine Config, and Ingress endpoints.

## Required Endpoints

| Frontend | Port | Backend Targets | Health Check |
|----------|------|-----------------|--------------|
| API | 6443 | Control plane nodes | HTTPS `/readyz` |
| Machine Config | 22623 | Control plane nodes | TCP |
| Ingress HTTP | 80 | Worker/infra nodes | TCP |
| Ingress HTTPS | 443 | Worker/infra nodes | TCP |

## Installation

```bash
# RHEL/CentOS
dnf install haproxy -y

# Enable and start
systemctl enable --now haproxy
```

## Complete Configuration

```bash
# /etc/haproxy/haproxy.cfg

global
    log         127.0.0.1 local2
    maxconn     4000
    daemon

defaults
    mode        tcp
    log         global
    option      tcplog
    timeout connect 10s
    timeout client  1m
    timeout server  1m

#---------------------------------------------------
# API (6443) — control plane nodes
# During bootstrap, include the bootstrap node
#---------------------------------------------------
frontend api
    bind *:6443
    default_backend api_servers

backend api_servers
    balance roundrobin
    option httpchk GET /readyz HTTP/1.0
    http-check expect status 200
    server bootstrap 10.0.10.5:6443 check check-ssl verify none
    server master0   10.0.10.10:6443 check check-ssl verify none
    server master1   10.0.10.11:6443 check check-ssl verify none
    server master2   10.0.10.12:6443 check check-ssl verify none

#---------------------------------------------------
# Machine Config (22623) — control plane nodes
# During bootstrap, include the bootstrap node
#---------------------------------------------------
frontend machine-config
    bind *:22623
    default_backend machine_config_servers

backend machine_config_servers
    balance roundrobin
    server bootstrap 10.0.10.5:22623 check
    server master0   10.0.10.10:22623 check
    server master1   10.0.10.11:22623 check
    server master2   10.0.10.12:22623 check

#---------------------------------------------------
# Ingress HTTP (80) — worker or infra nodes
#---------------------------------------------------
frontend ingress-http
    bind *:80
    default_backend ingress_http_servers

backend ingress_http_servers
    balance roundrobin
    server worker0 10.0.10.20:80 check
    server worker1 10.0.10.21:80 check
    server worker2 10.0.10.22:80 check

#---------------------------------------------------
# Ingress HTTPS (443) — worker or infra nodes
#---------------------------------------------------
frontend ingress-https
    bind *:443
    default_backend ingress_https_servers

backend ingress_https_servers
    balance roundrobin
    server worker0 10.0.10.20:443 check
    server worker1 10.0.10.21:443 check
    server worker2 10.0.10.22:443 check

#---------------------------------------------------
# Stats (optional)
#---------------------------------------------------
listen stats
    bind *:9000
    mode http
    stats enable
    stats uri /haproxy-stats
    stats auth admin:password
```

## High Availability with keepalived

For HA, run HAProxy on two nodes with keepalived for VIP failover:

```bash
# /etc/keepalived/keepalived.conf (primary)
vrrp_instance VI_API {
    state MASTER
    interface ens192
    virtual_router_id 51
    priority 100
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass <shared-secret>
    }
    virtual_ipaddress {
        10.0.10.100/24    # API VIP
        10.0.10.101/24    # Ingress VIP
    }
}
```

```bash
# Install and enable keepalived
dnf install keepalived -y
systemctl enable --now keepalived
```

## Post-Bootstrap Cleanup

After the bootstrap is complete, remove the bootstrap node from the API and Machine Config backends:

```bash
# Comment out or remove bootstrap entries
# server bootstrap 10.0.10.5:6443 check check-ssl verify none
# server bootstrap 10.0.10.5:22623 check

# Reload HAProxy
systemctl reload haproxy
```

## Firewall

```bash
firewall-cmd --add-port=6443/tcp --permanent
firewall-cmd --add-port=22623/tcp --permanent
firewall-cmd --add-port=80/tcp --permanent
firewall-cmd --add-port=443/tcp --permanent
firewall-cmd --add-port=9000/tcp --permanent   # stats (optional)
firewall-cmd --reload
```

## SELinux

```bash
# Allow HAProxy to bind to non-standard ports
setsebool -P haproxy_connect_any on
```

## Validation

```bash
# Verify HAProxy is running
systemctl status haproxy

# Test API endpoint
curl -sk https://<api-vip>:6443/readyz

# Check HAProxy stats
curl http://<lb-ip>:9000/haproxy-stats

# Verify backend health
echo "show stat" | socat stdio /var/lib/haproxy/stats
```
