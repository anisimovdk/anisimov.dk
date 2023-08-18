---
layout: post
title: "Mirror docker images to airgaped (disconnected) environment"
categories: skopeo mirror docker airgaped disconnected
toc: true
comments: true
---

If you need to deploy something into an air gaped cluster without changing images reference this post for you.

## Requirements

* skopeo
* docker
* golang

## Run local image registry

```bash
mkdir -p registry-data
docker run -d \
  -p 5000:5000 \
  --restart=always \
  --name registry \
  -v $(realpath ./registry-data):/var/lib/registry \
  registry:2
```

## Build skopeo from the source with the patch (Workaroud)

After merging [PR](https://github.com/containers/skopeo/pull/1531), this step should be omitted.

The current version does not preserve image namespace:

* `quay.io/coreos/etcd:latest` -> `registry.example.com/etcd:latest` - by default
* `quay.io/coreos/etcd:latest` -> `registry.example.com/quay.io/coreos/etcd:latest` - with flag `--scoped`,
* `quay.io/coreos/etcd:latest` -> `registry.example.com/coreos/etcd:latest` - with flags `--scoped` and `--scoped-append-registry=false`

To add `--scoped-append-registry=false` flag we need to patch skopeo sources and rebuild the binary:

```bash
git clone https://github.com/containers/skopeo
cd skopeo
wget https://patch-diff.githubusercontent.com/raw/containers/skopeo/pull/1531.patch
git apply 1531.patch
make bin/skopeo
cp bin/skopeo /usr/local/bin/skopeo
```

## Sync images to the local registry

Define image list, create `sync.yaml`

```yaml
# sync.yaml
# more info at https://github.com/containers/skopeo/blob/main/docs/skopeo-sync.1.md#yaml-file-content-used-source-for---src-yaml
quay.io:
  images:
    coreos/etcd:
      - latest
gcr.io:
  images:
    spark-operator/spark-operator:
      - latest
docker.io:
  images:
    centos:
      - latest
```

Run syncing to the local registry

```bash
skopeo sync \
  --insecure-policy \
  --override-os=linux \
  --override-arch=amd64 \
  --dest-tls-verify=false \
  --scoped \
  --scoped-append-registry=false \
  --src yaml \
  --dest docker \
  sync.yaml localhost:5000
```

## Sync image from local to remote registry

Define local registry as a mirror in `registries`.conf`:

```ini
# registries.conf
# mirror registry have higher priority
# more info https://github.com/containers/image/blob/main/docs/containers-registries.conf.5.md#example
[[registry]]
prefix="docker.io"
location="docker.io"
insecure=false
[[registry.mirror]]
location="localhost:5000"
insecure=true

[[registry]]
prefix="quay.io"
location="quay.io"
insecure=false
[[registry.mirror]]
location="localhost:5000"
insecure=true

[[registry]]
prefix="gcr.io"
location="gcr.io"
insecure=false
[[registry.mirror]]
location="localhost:5000"
insecure=true
```

Sync from local to remote registry, additionally specify `--registries-conf=registries.conf`:

```bash
skopeo sync \
  --registries-conf=registries.conf \
  --insecure-policy \
  --override-os=linux \
  --override-arch=amd64 \
  --dest-tls-verify=false \
  --scoped \
  --scoped-append-registry=false \
  --src yaml \
  --dest docker \
  sync.yaml registry.example.com:5000
```
