# Immich Go CLI, as an OCI Image

This repo is purely a CI only repo. That means it's pulling from the upstream [simulot/immich-go](https://github.com/simulot/immich-go)
repo, and packaging the code as an OCI image.

## Why this?

The original repo does not build an OCI Image (Sometimes called Docker images), and the maintainer seems unresponsive to
the issue [1356](https://github.com/simulot/immich-go/issues/1356) opened. Further, there's been some community interest
in a repo similar from my fork I made to build an image.

## How to use

You can use this locally on Podman/ Docker or in your Kubernetes cluster

```shell
# todo: Write this section once the images are built
```

## FAQ

### You talk about OCI, but the repo is named `immich-go-docker`

This is because most people will google `Immich go docker` looking for an image

### How do I know you've not fiddled with the images?

The CI used to build the images is available in this repo, and images are signed, SBOM'med and SLSA attested.

### How does the CI work

Renovate is installed on this repository, and is set up to watch tags released on [immich-go](https://github.com/simulot/immich-go) 
repository.

When a tag is released, the CI will git clone that repo, create a `.go.yaml` file in the cloned repo, and then run `ko build`

### What tags are available?

Eventually I will backdate all the tags releases meaning you've got access to all upstream versions. In the meantime
only the latest version (which is `v0.32.0` at time of writing (2026-10-26))

### Is there any AI usage on this repo?

No. I lovingly handcrafted this repo. In my mind, it's rude to get a clanker to write CI for a project someone lovingly 
handcrafted themselves. 

