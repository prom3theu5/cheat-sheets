Client Config:
```yaml
pki:
  # The CAs that are accepted by this node. Must contain one or more certificates created by 'nebula-cert ca'
  ca: c:/nebula/ca.crt
  cert: c:/nebula/desktop.crt
  key: c:/nebula/desktop.key

static_host_map:
  "100.100.100.1": ["IP_OF_LIGHTHOUSE:1717"]


lighthouse:
  am_lighthouse: false
  interval: 60
  hosts:
    - "IP_OF_LIGHTHOUSE"

listen:
  # To listen on both any ipv4 and ipv6 use "[::]"
  host: 0.0.0.0
  port: 1717

punchy:
  punch: true

relay:
  am_relay: false
  use_relays: true

tun:
  disabled: false
  dev: nebula1
  drop_local_broadcast: false
  drop_multicast: false
  tx_queue: 500
  mtu: 1300
  routes:
  unsafe_routes:

logging:
  level: info
  format: text

firewall:
  conntrack:
    tcp_timeout: 12m
    udp_timeout: 3m
    default_timeout: 10m

  outbound:
    - port: any
      proto: any
      host: any

  inbound:
    - port: any
      proto: any
      host: any

```

Start on all systems with:

```bash
nebula -config /path/to/config/file
```

On windows - you can install as a service, then run it with:

```powershell
.\nebula -service install -config C:\nebula\config.yaml
.\nebula -service start
```