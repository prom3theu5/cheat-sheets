### Failed to parse config

```sh
Error: failed to parse query parameter 'X-Registry-Config': "n/a": error storing credentials in temporary auth file (server: "https://index.docker.io/v1/", user: ""): key https://index.docker.io/v1/ contains http[s]:// prefix
```

Podman seems more strict than Docker when parsing the config file, check the `~/.docker/config.json` file for the key with the `https://` prefix (as mentioned in the error message) and remove it.


### Sock already exists

```sh
podman machine start ERRO[0000] "/var/folders/x_/bfc7v6kn4fs0rl9k77whs0nw0000gn/T/podman/qemu_podman-machine-default.sock" already exists panic: interface conversion: net.Conn is nil, not *net.UnixConn
```

This seems to happen (for me at least) when I've previously run `podman machine stop`. It looks like the sock file isn't correctly being removed. Doing an `rm` on that file mentioned in the error message will be enough to get you going again.


### Volume mounts

```sh
podman run --rm -it -v $(pwd):/usr/share/nginx/html:ro --publish 8000:80 docker.io/library/nginx:latest Error: statfs /Users/marcus/web: no such file or directory
```

Podman machine currently has no support for mounting volumes from the host machine (your Mac) into the container on the virtual machine. Instead, it attepts to mount a directory matching what you specified from the _virtual machine_ rather than your Mac.

This is a fairly big issue if you're looking for a smooth transition from Docker Desktop.

There's currently a fairly active [issue](https://github.com/containers/podman/issues/8016) about this limitation but as of right now there doesn't seem to be a nice workaround or solution.


### Automatic published port forwarding

```sh
podman run --rm -it --publish 8000:80 docker.io/library/nginx:latest & 
curl http://localhost:8000 curl: (7) Failed to connect to localhost port 8000: Connection refused
```

The current latest version of Podman ([v3.3.1](https://github.com/containers/podman/releases/tag/v3.3.1)) has a bug where the automatic port forwarding from host to VM when publishing a port with the `-p / --publish` flag doesn't work.

There's currently a couple workarounds for this:

The first is passing in the `--network bridge` flag to the podman command, e.g.

```sh
podman run --rm -it --publish 8000:80 --network bridge docker.io/library/nginx:latest
```

The other, more perminant option is to add `rootless_networking = "cni"` under the `[containers]` section of your `~/.config/containers/containers.conf` file.

To follow the progress of this bug, please refer to the [issue](https://github.com/containers/podman/issues/11396). **UPDATE**: This has now been merged and is expected to be released in v3.3.2 in the next few days or so.


## short-name resolution

```sh
Error: error creating build container: short-name resolution enforced but cannot prompt without a TTY
```

Ok, this is the big one and the major issue you'll likely hit making the switch today from Docker to Podman. Lets dive into it in a bit more detail...

First we need to understand what a short-name is in this context. It refers to container images that don't have a full domain name prefixed. You've likely come across these quite a lot before - e.g. `alpine:latest`, `ubuntu:12`, `giantswarm/pause:latest`, etc.

When using Docker, these images are actually first prefixed with `docker.io` (or `docker.io/library` for those official images without a namespace) before being pulled.

Podman doesn't have this as a default. It can work in the same way as Docker but needs a bit of configuring.

It's worth briefly pausing here to explain _why_ this behavior is different. Podman takes the **secure by default** attitude to configuration and installation, and this difference is a prime example of that mindset. You've likely heard in the news over the past few years about some of the supply chain hacks that have had a big impact on some companies and projects. One of the common attack vectors is tricking users into installing what they think is a legitimate package but actually contains malicious code. The use of short names for images opens up the risk of accidentally pulling the wrong image from the wrong registry.

To mitigate this risk Podman has a feature where it will prompt you asking which registry you'd like to pull the shot named image from and will then save that choice to speed things up later. (On a side note, there's a repo where the community is trying to collate a collection of some of the most widely used shortcodes - https://github.com/containers/shortnames)

So, Podman has this handy feature to help out with security so why are we seeing an error? Well, when running Podman on MacOS (or Windows) we're actually running it in a Linux VM and remotely connecting to Podman running in that machine. Because of this we don't have an interactive terminal with the underlying Podman engine so it is unable to receive our choice if it asked us which registry to use.

#### Fix

There's a couple of solutions for this:

1.  Instead of using short names we could switch to using fully prefixed images (this includes updating any `FROM` commands in our Dockerfiles also).
    
2.  The other approach is to reduce this security feature to be on-par with the experience we're used to with Docker.
    

As the first solution really just relies on you changing the image names you're referencing, which will depend on how you're working, I'll focus on the second solution.

With our machine created and started (as outlined above) we need to access the machine to make a small configuration change. Thankfully Podman makes this quite easy:

```sh
podman machine ssh
```

This will drop you into an SSH session within the virtual machine created for Podman. Once in this machine we want to make a change to the `/etc/containers/registries.conf` file. If we take a look at the file contents we'll see the final lines of it (at time of writing this) as followed:

```yaml
# Enforcing mode for short names is default for Fedora 34 and newer
short-name-mode="enforcing"
```

The `short-name-mode` property has 3 possible values:

-   **enforcing**: If no alias is found and more than one unqualified-search registry is set, prompt the user to select one registry to pull from. If the user cannot be prompted (i.e., stdin or stdout are not a TTY), Podman will throw an error.
-   **permissive**: Behaves as enforcing but will not throw an error if the user cannot be prompted. Instead, Podman will try all unqualified-search registries in the given order. Note that no alias will be recorded.
-   **disabled**: Podman will try all unqualified-search registries in the given order, and no alias will be recorded. This is pretty much the same behavior of Podman before short names were introduced.

If we want Podman to perform more like Docker we'll want to change this value to `permissive`:

```sh
sudo sed -i 's/short-name-mode="enforcing"/short-name-mode="permissive"/g' /etc/containers/registries.conf
```

There's one more property in this file that it's worth at least being aware of.

```yaml
unqualified-search-registries = ["registry.fedoraproject.org", "registry.access.redhat.com", "docker.io", "quay.io"]
```

This property contains a list of all the registries that will be checked (in order) when looking up a short name image. **Be sure the values in here are ones that you trust!**

With that change made we can `exit` from the virtual machine and Podman should then search for any short name images using these registries from now on.

<hr>

Fixes for TestContainers under Podman

Edit `~/.testcontainers.properties` and add the following line

```
ryuk.container.privileged=true
```

Then run the following

```
brew install podman
podman machine init -v $HOME:$HOME
```

If you're podman 4.1 or higher, you don't need the the `-v $HOME:$HOME` volume mount.

If you're on mac,

```
sudo /opt/homebrew/Cellar/podman/4.0.3/bin/podman-mac-helper install
```

Then (for all oses)

```
podman machine set --rootful
podman machine start
```

The rootful part makes ryuk behave ok, based on testing (see [containers/podman#14238](https://github.com/containers/podman/discussions/14238)). 
If ryuk is still causing problems or you don't want to run rootful then `export TESTCONTAINERS_RYUK_DISABLED=true` also does the trick, but containers can sometimes hang around and cause problems.

Finally, if you're on Mac M1, update podman to behave better with images for a different architecture (see [https://edofic.com/posts/2021-09-12-podman-m1-amd64/](https://edofic.com/posts/2021-09-12-podman-m1-amd64/) )

```
podman machine ssh
sudo -i
rpm-ostree install qemu-user-static
systemctl reboot
```

Once the virtual machine restarts, you should be good to run testcontainers.

 #### DOCKER Host helper for mac

 Run the following command to ensure that the podman host socket is discoverable on the default DOCKER_HOST env var on a mac

 ```
 sudo podman-mac-helper install
 ```