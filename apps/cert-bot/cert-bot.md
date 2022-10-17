1. Ensure Snap is installed
```bash
sudo apt update
sudo apt install -y snapd
sudo snap install core; sudo snap refresh core
```

2. Install certbot
```bash
sudo snap install --classic certbot
sudo ln -s /snap/bin/certbot /usr/bin/certbot
```

3.  Open port 80 UFW Firewall
```bash
sudo ufw allow 80
sudo ufw allow 443
sudo ufw reload
```

4. Generate Cert (Make sure nothing is using port 80)
```bash
sudo certbot certonly --standalone --preferred-challenges http -d example.com
```
Replace `example.com` with your domain
The certs will be stored in `/etc/letsencrypt/live/` directory

5. Test automatic renewal
The Certbot packages on your system come with a cron job or systemd timer that will renew your certificates automatically before they expire. You will not need to run Certbot again, unless you change your configuration. You can test automatic renewal for your certificates by running this command:

`sudo certbot renew --dry-run`

6. View Cert files
```bash
sudo ls /etc/letsencrypt/live/example.com
```