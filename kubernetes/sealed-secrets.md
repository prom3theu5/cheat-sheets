**Problem:** "I can manage all my K8s config in git, except Secrets."

**Solution:** Encrypt your Secret into a SealedSecret, which _is_ safe to store - even inside a public repository. The SealedSecret can be decrypted only by the controller running in the target cluster and nobody else (not even the original author) is able to obtain the original Secret from the SealedSecret.

Sealed Secrets is composed of two parts:

-   A cluster-side controller / operator
-   A client-side utility: `kubeseal`

The `kubeseal` utility uses asymmetric crypto to encrypt secrets that only the controller can decrypt.


### SealedSecrets as templates for secrets

The previous example only focused on the encrypted secret items themselves, but the relationship between a `SealedSecret` custom resource and the `Secret` it unseals into is similar in many ways (but not in all of them) to the familiar `Deployment` vs `Pod`.

In particular, the annotations and labels of a `SealedSecret` resource are not the same as the annotations of the `Secret` that gets generated out of it.

To capture this distinction, the `SealedSecret` object has a `template` section which encodes all the fields you want the controller to put in the unsealed `Secret`.

```yml
apiVersion: bitnami.com/v1alpha1
kind: SealedSecret
metadata:
  name: mysecret
  namespace: mynamespace
  annotations:
    "kubectl.kubernetes.io/last-applied-configuration": ....
spec:
  encryptedData:
    .dockerconfigjson: AgBy3i4OJSWK+PiTySYZZA9rO43cGDEq.....
  template:
    type: kubernetes.io/dockerconfigjson
    # this is an example of labels and annotations that will be added to the output secret
    metadata:
      labels:
        "jenkins.io/credentials-type": usernamePassword
      annotations:
        "jenkins.io/credentials-description": credentials from Kubernetes
```

The controller would unencrypt to something like

```yml
apiVersion: v1
kind: Secret
metadata:
  name: mysecret
  namespace: mynamespace
  labels:
    "jenkins.io/credentials-type": usernamePassword
  annotations:
    "jenkins.io/credentials-description": credentials from Kubernetes
  ownerReferences:
  - apiVersion: bitnami.com/v1alpha1
    controller: true
    kind: SealedSecret
    name: mysecret
    uid: 5caff6a0-c9ac-11e9-881e-42010aac003e
type: kubernetes.io/dockerconfigjson
data:
  .dockerconfigjson: ewogICJjcmVk...
```

As you can see, the generated `Secret` resource is a "dependent object" of the `SealedSecret` and as such it will be updated and deleted whenever the `SealedSecret` object gets updated or deleted.

SealedSecrets are from the POV of an end user a "write only" device.

The idea is that the SealedSecret can be decrypted only by the controller running in the target cluster and nobody else (not even the original author) is able to obtain the original Secret from the SealedSecret.

The user may or may not have direct access to the target cluster. More specifically, the user might or might not have access to the Secret unsealed by the controller.

There are many ways to configure RBAC on k8s, but it's quite common to forbid low-privilege users from reading Secrets. It's also common to give users one or more namespaces where they have higher privileges, which would allow them to create and read secrets (and/or create deployments that can reference those secrets).

Encrypted `SealedSecret` resources are designed to be safe to be looked at without gaining any knowledge about the secrets it conceals. This implies that we cannot allow users to read a SealedSecret meant for a namespace they wouldn't have access to and just push a copy of it in a namespace where they can read secrets from.

Sealed-secrets thus behaves _as if_ each namespace had its own independent encryption key and thus once you seal a secret for a namespace, it cannot be moved in another namespace and decrypted there.

We don't technically use an independent private key for each namespace, but instead we _include_ the namespace name during the encryption process, effectively achieving the same result.

Furthermore, namespaces are not the only level at which RBAC configurations can decide who can see which secret. In fact, it's possible that users can access a secret called `foo` in a given namespace but not any other secret in the same namespace. We cannot thus by default let users freely rename `SealedSecret` resources otherwise a malicious user would be able to decrypt any SealedSecret for that namespace by just renaming it to overwrite the one secret user does have access to. We use the same mechanism used to include the namespace in the encryption key to also include the secret name.

That said, there are many scenarios where you might not care about this level of protection. For example, the only people who have access to your clusters are either admins or they cannot read any `Secret` resource at all. You might have a use case for moving a sealed secret to other namespaces (e.g. you might not know the namespace name upfront), or you might not know the name of the secret (e.g. it could contain a unique suffix based on the hash of the contents etc).

These are the possible scopes:

-   `strict` (default): the secret must be sealed with exactly the same _name_ and _namespace_. These attributes become _part of the encrypted data_ and thus changing name and/or namespace would lead to "decryption error".
-   `namespace-wide`: you can freely _rename_ the sealed secret within a given namespace.
-   `cluster-wide`: the secret can be unsealed in _any_ namespace and can be given _any_ name.
In contrast to the restrictions of _name_ and _namespace_, secret _items_ (i.e. JSON object keys like `spec.encryptedData.my-key`) can be renamed at will without losing the ability to decrypt the sealed secret.

The scope is selected with the `--scope` flag:
```bash
kubeseal --scope cluster-wide <secret.yaml >sealed-secret.json
```
It's also possible to request a scope via annotations in the input secret you pass to `kubeseal`:

-   `sealedsecrets.bitnami.com/namespace-wide: "true"` -> for `namespace-wide`
-   `sealedsecrets.bitnami.com/cluster-wide: "true"` -> for `cluster-wide`

The lack of any of such annotations means `strict` mode. If both are set, `cluster-wide` takes precedence.

---

## Installation

### Installation (Controller)

```bash
helm repo add sealed-secrets https://bitnami-labs.github.io/sealed-secrets

helm install sealed-secrets -n kube-system --set-string fullnameOverride=sealed-secrets-controller sealed-secrets/sealed-secrets
```


### Installation (Kubeseal)

#### Linux

```bash
wget https://github.com/bitnami-labs/sealed-secrets/releases/download/<release-tag>/kubeseal-<version>-linux-amd64.tar.gz
tar -xvzf kubeseal-<version>-linux-amd64.tar.gz kubeseal
sudo install -m 755 kubeseal /usr/local/bin/kubeseal
```

#### Mac
```bash
brew install kubeseal
```

### Usage
```bash
# Create a json/yaml-encoded Secret somehow:
# (note use of `--dry-run` - this is just a local file!)
echo -n bar | kubectl create secret generic mysecret --dry-run=client --from-file=foo=/dev/stdin -o json >mysecret.json

# This is the important bit:
# (note default format is json!)
kubeseal <mysecret.json >mysealedsecret.json

# At this point mysealedsecret.json is safe to upload to Github,
# post on Twitter, etc.

# Eventually:
kubectl create -f mysealedsecret.json

# Profit!
kubectl get secret mysecret
```


---

### Own encryption cert
Setting up own certificate for encryption

```bash
# Set ENV Vars
export PRIVATEKEY="mytls.key"
export PUBLICKEY="mytls.crt"
export NAMESPACE="kube-system"
export SECRETNAME="sealedsecrets"

# Create Certificate
openssl req -x509 -nodes -newkey rsa:4096 -keyout "$PRIVATEKEY" -out "$PUBLICKEY" -subj "/CN=sealed-secret/O=sealed-secret"

# Create secret in k8s and set as active
kubectl -n "$NAMESPACE" create secret tls "$SECRETNAME" --cert="$PUBLICKEY" --key="$PRIVATEKEY"

kubectl -n "$NAMESPACE" label secret "$SECRETNAME" sealedsecrets.bitnami.com/sealed-secrets-key=active

# Delete existing pod if deployed
kubectl -n  "$NAMESPACE" delete pod -l name=sealed-secrets-controller
```

Confirm is in use:

```bash
# Confirm new pod is using secret
kubectl -n "$NAMESPACE" logs -l name=sealed-secrets-controller
```

Expected Output should show Secret with secret name

```
controller version: v0.12.1+dirty
2020/06/09 14:30:45 Starting sealed-secrets controller version: v0.12.1+dirty
2020/06/09 14:30:45 Searching for existing private keys
2020/06/09 14:30:45 ----- sealedsecrets
2020/06/09 14:30:45 HTTP server serving on :8080
```