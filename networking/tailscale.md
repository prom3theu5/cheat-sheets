# Tailscale
**Tailscale** is a zero config **VPN ([[vpn]])** for building secure networks, powered by **WireGuard ([[wireguard]])**. Install on any device in minutes. Remote access from any network or physical location.

Project Homepage: https://tailscale.com

---
## Lockdown Server with UFW

First find your tailscale IP on the server

```bash
tailscale ip -4
```

Next disconnect and reconnect over ssh using your tailscale ip

```bash
ssh <username>@<copied 100.x.y.z address>
```

Now enable ufw, and add the rules

```bash
sudo ufw allow in on tailscale0
sudo ufw enable
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp
sudo ufw allow 41641/udp
sudo ufw reload
```

## Lockdown with FirewallD

```bash
# sudo ufw enable
sudo systemctl start firewalld
sudo systemctl enable firewalld
sudo firewall-cmd --set-default-zone=drop
sudo firewall-cmd --add-port=80/tcp --permanent
sudo firewall-cmd --add-port=443/tcp --permanent
sudo firewall-cmd --zone=public --add-port=41641/udp --permanent
sudo firewall-cmd --remove-service=cockpit --zone=public --permanent
sudo firewall-cmd --remove-service=ssh --zone=public --permanent
sudo firewall-cmd --remove-service=dhcpv6-client --zone=public --permanent
sudo firewall-cmd --reload
```

