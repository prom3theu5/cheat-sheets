Expose extra ports in case you wish to setup Exernal routing using ingressTcpRoutes

Created a custom manifest for traefik at the `k3s data dir` / `servers` / `manifest`s  path.. i.e.
`/etc/rancher/k3s/server/manifests/traefik-custom.yaml`

The following contents show mqtt as an example

```yaml
apiVersion: helm.cattle.io/v1
kind: HelmChartConfig
metadata:
  name: traefik
  namespace: kube-system
spec:
  valuesContent: |-
    additionalArguments:
      - "--entrypoints.mqtt.address=:1883/tcp"
    ports:
      traefik:
        expose: true
      mqtt:
        port: 1883
        expose: true
        exposedPort: 1883
        protocol: TCP
    providers:
      kubernetesCRD:
        allowCrossNamespace: true
```

Next create an ingress tcp resource which is a traefik CRD
It exposes the service and mounts it on the port thats added as an endpoint


```yaml
apiVersion: traefik.containo.us/v1alpha1
kind: IngressRouteTCP
metadata:
  name: mqtt
spec:
  entryPoints:
  - mqtt
  routes:
    - match: HostSNI(`*`)
      services:
        - name: mosquitto
          port: 1883
```

Thats all. External Services should now be able to the service without having to go through the cluster.