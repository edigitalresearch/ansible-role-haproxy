# HAProxy Role for Ansible

This is a role that installs a Haproxy service on RedHat/CentOS Linux servers.
It requires a single host group, named `haproxy`.


## Variables

Available variables are listed below, along with default values (see `defaults/main.yml`):
```---
haproxy:
  certs_path: /etc/haproxy/certs
  logs:
   - /dev/log  local0
   - /dev/log  local1 notice
  global:
    pidfile: /var/run/haproxy.pid
    chroot:  /var/lib/haproxy
    user: haproxy
    group: haproxy
    stats: socket /var/lib/haproxy/stats
    ssl-default-bind-options: "no-sslv3"
    ssl-default-bind-ciphers: "ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-SHA384:ECDHE-RSA-AES256-SHA384:ECDHE-ECDSA-AES128-SHA256:ECDHE-RSA-AES128-SHA256K"
    ssl-default-server-options: "no-sslv3"
    ssl-default-server-ciphers: "ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-SHA384:ECDHE-RSA-AES256-SHA384:ECDHE-ECDSA-AES128-SHA256:ECDHE-RSA-AES128-SHA256K"
  listen:
    stats:
      bind: '*:8181'
      stats:
        enable: ""
        uri: "/"
        realm: 'Haproxy Statistics'
        auth: haproxy:haproxy
  frontend:
    hafrontend:
      bind: '*:80'
      mode: 'http'
      option:
        http-server-close:
      default_backend: "habackend"
  backend:
    habackend:
      bind: '*:80'
      mode: 'http'
      balance: 'roundrobin'
      option: # Dictionary of options and values
        forwardfor:
      reqadd: # Headers set by the Load Balancer
        X-Forwarded-Proto: '\ https if { ssl_fc }'
      server: # Dictionary of web servers and applicable ports, followed by any server arguments
        web01: web01.example.com:80 check
  timeout:
    http-request: 10s
    queue: 1m
    connect: 10s
    client: 1m
    server: 1m
    http-keep-alive: 10s
    check: 10s
  defaults:
    mode: 'http'
    log: 'global'
    retries: 3
    option:
      httplog:
      dontlognull:
      http-server-close:
      forwardfor: except 127.0.0.0/8
      redispatch:
```
The above will generate a TLS v1.2 only compliant server.

To add SSL certificates, combined the cert, intermediary and private key into a vault file.
```
---
  vault_haproxy:
    certs:
      example.com:
        content: |
          -----BEGIN CERTIFICATE-----
          -----END CERTIFICATE-----
          -----BEGIN PRIVATE KEY-----
          -----END PRIVATE KEY-----

```
## Handlers

This role provides a single handler:

* `restart haproxy`

## Example Task with role

```  
- name: Install and configure Haproxy
  hosts: haproxy
  roles:
    - edr.haproxy
```
