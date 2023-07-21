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