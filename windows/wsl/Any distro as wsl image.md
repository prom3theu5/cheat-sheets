## Install Podman and utilise skopeo to extract container images

### Download Image
```bash
IMAGE_TO_GET="registry.access.redhat.com/ubi8/ubi:8.1"
mkdir ./output
podman run --rm -v ./output:/output quay.io/skopeo/stable:latest copy docker://$IMAGE_TO_GET --override-os linux --override-arch amd64 dir:./output
```

Now look at the manifest file. If there is more than one layer - follow below - or skip to import step.

## If Image Has More than one Layer

### Extract Layers if more than one

```bash
cd ./output
mkdir ./OUTPUT_METADATA
# move 'manifest.json' and 'version' files to different directory, as they aren't archives
mv version manifest.json ./OUTPUT_METADATA/
for f in $(ls); do tar xvf $f; done
```

### Create Tarball

```bash
tar cv . | xz > distro-name.tar.xz
```

## Use Image by importing to wsl

1.  Create a directory for WSL
    -   `mkdir c:\wsl`
2.  Create the WSL instance by importing the tarball:
    -   `wsl --import <Name> c:\wsl\<name> <name>.tar.gz --version 2`
3.  Create a shortcut to wsl -d rhel to start, or start manually.