	1. Get binaries
```bash
wget https://github.com/slackhq/nebula/releases/download/v1.6.1/nebula-linux-amd64.tar.gz
```

	2. Extract and Cleanup
```bash
tar -zxvf nebula-linux-amd64.tar.gz
sudo mv nebula* /usr/local/bin
rm nebula-linux-amd64.tar.gz
```

	3. Create Neubla CA cert for your network
```bash
mkdir ~/nebula && cd ~/nebula
sudo nebula-cert ca -name "My Nebula Network"
```
Replace `My Nebula Network` with your network name.
Before locking it away, we need it to create some client certificates, 1 for each client. When generating a client certificate, you need have decided what subnet you’ll be using for the VPN network. I’m going to use `100.100.100.0/24`.

	4. Create Lighthouse Cert
```bash
sudo nebula-cert sign -name "lighthouse" -ip "100.100.100.1/24"
```
This will be needed on each device

	5. Create cert for each device as well as assign IP
```bash
sudo nebula-cert sign -name "desktop" -ip "100.100.100.2/24"
sudo nebula-cert sign -name "laptop" -ip "100.100.100.3/24"
```

### Lighthouse

As mentioned before, the lighthouse client is used to establish a tunnel directly between clients to allow them to communicate.

When running, the nebula daemon uses very little resources. A few MB of RAM, and basically no CPU. Whatever device you have to run this, will run it just fine. The only requirement for a lighthouse is that it be reliable, and have a static IP.

#### Rootless lighthouse

By default, a lighthouse will also be accessible on the Nebula network as any other client, however it’s possible to tell Nebula not to create a network interface. This means it won’t be able to be communicated with over the nebula network, but means you can run your lighthouse without root.

### Configuration

Configuration for Nebula, like everything in the cloud, is done using a YAML file. The best way to get started is using the [example config](https://github.com/slackhq/nebula/blob/master/examples/config.yml). It won’t work at all by default, so we need to make some changes to it.

The example configuration shows some great examples on what keys are and how you may want to use them. If you’re going to deploy Nebula I recommend giving it a read in full. But these are the things you need to change to get a working tunnel:

#### Specify the keys

Under the `pki` key, specify the paths to the `ca.crt` along with the 2 keys created for the specific client.

```yaml
pki:
  ca: /etc/nebula/ca.crt
  cert: /etc/nebula/lighthouse.crt
  key: /etc/nebula/lighthouse.key
```

#### Static hosts

The static hosts define a list of nodes with a static IP for communication. This is mostly designed for specifying the lighthouse, but may also be useful for other nodes which will always have the same fixed IP.

```yaml
static_host_map:
  "100.100.100.1": ["100.64.22.11:4242"]
```

As noted in the example config, this should be a mapping from the _VPN IP_ to the _public IP_ and port.

#### Lighthouse configuration

For a node to be a lighthouse, you have to tell it that it’s a lighthouse using the `am_lighthouse` key. If the node is a lighthouse, make it `true`. If it’s not, make it `false`.

```yaml
lighthouse:
  am_lighthouse: false
```

Below that, you need to specify the _VPN IPs_ of all the lighthouses. If the node you’re editing is a lighthouse, make sure hosts is empty. Yes, nebula supports multiple lighthouses!

```yaml
lighthouse:
  hosts:
    - "100.100.100.1"
```

#### Listen

By default, Nebula listens on port 4242 (UDP) for lighthouse traffic. For your lighthouse, this should be set to something static, and kept in sync with the static hosts of clients. For nodes which aren’t fixed, such as regular client devices, it’s recommend to set this to 0, so nebula assigns a random port on startup.

```yaml
listen:
  host: 0.0.0.0
  port: 0
```

#### Firewall

Nebula isn’t just a tunnel, it also has some firewalling built in. Nebula nodes can be added to groups, and these groups can have certain access to ports on nodes. For example, you may want to only allow servers in the “db” group to allow traffic on port 5432, or only allow nodes in the group “web” to send traffic on ports 443, 80 and 5432. The example config [shows](https://github.com/slackhq/nebula/blob/master/examples/config.yml#L213) how to configure these as needed.

For testing, I’d recommend just allowing all traffic to all devices. If you want to tighten things down after you’ve got everything working, then it’s best to do that afterwards.

```yaml
firewall:
  outbound:
    - port: any
      proto: any
      host: any

  inbound:
    - port: any
      proto: any
      host: any
```

	6. Create Config folder
```bash
sudo mkdir /etc/nebula
sudo cp ~/nebula/ca.crt /etc/nebula/
sudo cp ~/nebula/ca.key /etc/nebula/
sudo cp ~/nebula/lighthouse.crt /etc/nebula/
sudo cp ~/nebula/lighthouse.key /etc/nebula/
sudo cp ~/nebula/config.yaml /etc/nebula/
```

	7. Test nebula - Fire up the lighthouse
```bash
sudo nebula -config /etc/nebula/config.yml
```

	8. Create Systemd Service, and Start it
```bash
sudo nano /etc/systemd/system/nexus.service
```

```toml
[Unit]
Description=nebula
Wants=basic.target
After=basic.target network.target
Before=sshd.service

[Service]
SyslogIdentifier=nebula
ExecReload=/bin/kill -HUP $MAINPID
ExecStart=/usr/local/bin/nebula -config /etc/nebula/config.yaml
Restart=always

[Install]
WantedBy=multi-user.target
```
Paste in the above, and save the file

Then start the service
```bash
sudo systemctl enable nebula --now
sudo systemctl status nebula
```

You should see the following
```bash
● nebula.service - nebula
     Loaded: loaded (/etc/systemd/system/nebula.service; enabled; vendor preset: enabled)
     Active: active (running) since Tue 2022-10-25 09:49:51 UTC; 3min 10s ago
   Main PID: 58023 (nebula)
      Tasks: 8 (limit: 1116)
     Memory: 2.2M
        CPU: 280ms
     CGroup: /system.slice/nebula.service
             └─58023 /usr/local/bin/nebula -config /etc/nebula/config.yaml
```