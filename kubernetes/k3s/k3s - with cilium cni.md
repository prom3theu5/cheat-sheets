Cilium is an open source, cloud native solution for providing, securing, and observing network connectivity between workloads, fuelled by the revolutionary Kernel technology eBPF

[Cilium - Cloud Native, eBPF-based Networking, Observability, and Security](https://cilium.io/)

## K3s

First Install K3s (Skip backend if using embedded - don't recommend. Use postgres...)
Be sure to disable:

- flannel backend
- network policy
- servicelb

Optionally disable traefik if using nginx

```bash
POSTGRES_DATABASE=k3s
POSTGRES_USER=postgres
POSTGRES_PASSWORD=SuperSecretPassword123!

curl -sfL https://get.k3s.io | INSTALL_K3S_VERSION="v1.27.6+k3s1" INSTALL_K3S_EXEC="--flannel-backend=none --disable-network-policy --disable servicelb --disable traefik --write-kubeconfig-mode 644 --tls-san 88.99.199.13 --datastore-endpoint postgres://$POSTGRES_USER$:$POSTGRES_PASSWORD@localhost:5432/$POSTGRES_DATABASE --resolv-conf=/etc/k3s-resolv.conf" sh -
```

---

## Cilium

Now - install the cilium cli

### Linux:

```bash
CILIUM_CLI_VERSION=$(curl -s https://raw.githubusercontent.com/cilium/cilium-cli/main/stable.txt)
CLI_ARCH=amd64
if [ "$(uname -m)" = "aarch64" ]; then CLI_ARCH=arm64; fi
curl -L --fail --remote-name-all https://github.com/cilium/cilium-cli/releases/download/${CILIUM_CLI_VERSION}/cilium-linux-${CLI_ARCH}.tar.gz{,.sha256sum}
sha256sum --check cilium-linux-${CLI_ARCH}.tar.gz.sha256sum
sudo tar xzvfC cilium-linux-${CLI_ARCH}.tar.gz /usr/local/bin
rm cilium-linux-${CLI_ARCH}.tar.gz{,.sha256sum}
```

### Mac:

```bash
CILIUM_CLI_VERSION=$(curl -s https://raw.githubusercontent.com/cilium/cilium-cli/main/stable.txt)
CLI_ARCH=amd64
if [ "$(uname -m)" = "arm64" ]; then CLI_ARCH=arm64; fi
curl -L --fail --remote-name-all https://github.com/cilium/cilium-cli/releases/download/${CILIUM_CLI_VERSION}/cilium-darwin-${CLI_ARCH}.tar.gz{,.sha256sum}
shasum -a 256 -c cilium-darwin-${CLI_ARCH}.tar.gz.sha256sum
sudo tar xzvfC cilium-darwin-${CLI_ARCH}.tar.gz /usr/local/bin
rm cilium-darwin-${CLI_ARCH}.tar.gz{,.sha256sum}
```

### Others:

See releases page: [Release Release v0.15.20 · cilium/cilium-cli (github.com)](https://github.com/cilium/cilium-cli/releases/tag/v0.15.20)

---

### Install Cilium CNI

Next step - I like to cordon and drain the cluster

```bash
NODE=the-node-name

kubectl cordon $NODE
kubectl drain $NODE --ignore-daemonsets --delete-empty-dir
```

Cilium cli utilises the helm chart to install cilium in the cluster, so we can apply some custom values. We need to ensure the cluster pool CIDR matches the default 10.42.0.0/16 in k3s


```bash
CILIUM_VERSION=1.14.6
CIDR_POOL=10.42.0.0/16

cilium install --version $CILIUM_VERSION --set ipam.operator.clusterPoolIPv4PodCIDRList=$CIDR_POOL --set ipam.mode=cluster-pool
```

Wait for cilium operator and pods to start, and the status to show as fine

```bash
cilium status --wait

    /¯¯\
 /¯¯\__/¯¯\    Cilium:             OK
 \__/¯¯\__/    Operator:           OK
 /¯¯\__/¯¯\    Envoy DaemonSet:    disabled (using embedded mode)
 \__/¯¯\__/    Hubble Relay:       disabled
    \__/       ClusterMesh:        disabled

DaemonSet              cilium             Desired: 1, Ready: 1/1, Available: 1/1
Deployment             cilium-operator    Desired: 1, Ready: 1/1, Available: 1/1
Containers:            cilium             Running: 1
                       cilium-operator    Running: 1
Cluster Pods:          2/2 managed by Cilium
Helm chart version:    1.14.6
Image versions         cilium             quay.io/cilium/cilium:v1.14.6@sha256:37a49f1abb333279a9b802ee8a21c61cde9dd9138b5ac55f77bdfca733ba852a: 1
                       cilium-operator    quay.io/cilium/operator-generic:v1.14.6@sha256:2f0bf8fb8362c7379f3bf95036b90ad5b67378ed05cd8eb0410c1afc13423848: 1
```

Finally - un-cordon the node, and allow everything to start

```bash
NODE=the-node-name

kubectl uncordon $NODE
```

All done
