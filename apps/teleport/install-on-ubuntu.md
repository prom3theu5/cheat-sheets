## Install on Ubuntu 22.04

1. Add GPG Signing Key
```bash
sudo curl https://apt.releases.teleport.dev/gpg \ -o /usr/share/keyrings/teleport-archive-keyring.asc
```

2. Add Teleport Repository
```bash
source /etc/os-release

echo "deb [signed-by=/usr/share/keyrings/teleport-archive-keyring.asc] \ https://apt.releases.teleport.dev/${ID?} ${VERSION_CODENAME?} stable/v10" \| sudo tee /etc/apt/sources.list.d/teleport.list > /dev/null
```

3. Update Apt
```bash
sudo apt update
```

4. Install Teleport
```bash
sudo apt-get install teleport
```

5. Generate SSL Cert
[[cert-bot]]

6. Generate Configuration File of Teleport
```bash
sudo teleport configure -o /etc/teleport.yaml  \  
--cluster-name=example.domain.com \  
--public-addr=teleport.example.domain.com:443 \  
--cert-file=/etc/letsencrypt/live/example.domain.com/fullchain.pem \  
--key-file=/etc/letsencrypt/live/example.domain.com/privkey.pem
```

You can view the contents of the teleport config file with
```bash
 cat /etc/teleport.yaml
```

7. Enable Teleport Service
```bash
sudo systemctl enable --now teleport  
sudo systemctl status teleport
```

8. Add a teleport user and setup roles
```bash
sudo tctl users add prom3theu5 --roles=editor, access
```

9. Open Firewall Ports
```bash
sudo ufw allow 3022
sudo ufw allow 3023
sudo ufw allow 3024
sudo ufw allow 3025
sudo ufw allow 3026
sudo ufw allow 3080
sudo ufw reload
```


10. Access Teleport webui using the web ui https://teleport.example.domain.com