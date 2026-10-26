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
podman run ghcr.io/userbradley/immich-go-docker
```

If you wanted to run `immich-go version` you can do so as below

```shell
podman run ghcr.io/userbradley/immich-go-docker version
```

Any of the arguments can be passed as you would usually.

## Image naming

The images are named with the upstream versin, with and without `v`. Example below

* `ghcr.io/userbradley/immich-go-docker:v0.32.0`
* `ghcr.io/userbradley/immich-go-docker:0.32.0`
* `ghcr.io/userbradley/immich-go-docker:latest`


<!-- TOC -->
* [Immich Go CLI, as an OCI Image](#immich-go-cli-as-an-oci-image)
  * [Why this?](#why-this)
  * [How to use](#how-to-use)
  * [Image naming](#image-naming)
  * [FAQ](#faq)
    * [You talk about OCI, but the repo is named `immich-go-docker`](#you-talk-about-oci-but-the-repo-is-named-immich-go-docker)
    * [How do I know you've not fiddled with the images?](#how-do-i-know-youve-not-fiddled-with-the-images)
    * [How does the CI work](#how-does-the-ci-work)
    * [What tags are available?](#what-tags-are-available)
    * [Is there any AI usage on this repo?](#is-there-any-ai-usage-on-this-repo)
  * [Signing, SBOM and SLSA](#signing-sbom-and-slsa)
    * [Signed images](#signed-images)
    * [SBOM](#sbom)
    * [SLSA Compliance & Artifacts](#slsa-compliance--artifacts)
<!-- TOC -->

## FAQ

### You talk about OCI, but the repo is named `immich-go-docker`

This is because most people will google `Immich go docker` looking for an image

### How do I know you've not fiddled with the images?

The CI used to build the images is available in this repo, and images are signed, SBOM'med and SLSA attested.

Older version of the image, will not be signed or have an SBOM attached. This is purely because it's a lot of work to write the CI
for, and it does not really add much quality of life improvements

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

## Signing, SBOM and SLSA

The following sections explains how to verify and view the SBOM

All the commands require you to get the `sha` for the image you want to verify.

This can be done with `cosign manifest ghcr.io/userbradley/immich-go-docker`

### Signed images

All images built by this pipeline are signed using Sigstore Cosign in keyless mode. The signature is applied recursively, meaning both the multi-arch image index and every individual architecture specific are also signed

How to verify the image signature:

```shell
cosign verify \
--certificate-identity-regexp "https://github.com/userbradley/.*" \
--certificate-oidc-issuer "https://token.actions.githubusercontent.com" \
ghcr.io/userbradley/immich-go-docker@sha256:7af472edd1db5092ae42e1de8b81ad6fb906ad41a37cc5c3ff512b63c7dcb133 
```

### SBOM

All the images have an SBOM (Software bill of material) attached to them

```shell
cosign verify-attestation \
  --certificate-identity-regexp "https://github.com/userbradley/.*" \
  --certificate-oidc-issuer "https://token.actions.githubusercontent.com" \
  ghcr.io/userbradley/immich-go-docker@sha256:7af472edd1db5092ae42e1de8b81ad6fb906ad41a37cc5c3ff512b63c7dcb133 \
  --type https://spdx.dev/Document | jq -r '.payload | @base64d' | jq
```

And if we want 

### SLSA Compliance & Artifacts

This project achieves SLSA Level 3 (Supply chain Levels for Software Artifacts) compliance by utilising the official slsa-framework/slsa-github-generator.


SLSA Artifacts included:

* Build Provenance (in-toto format) pushed directly to the `Sigstore` transparency log.

Get the manifests and then pick wich one you want to view

```shell
crane manifest ghcr.io/userbradley/immich-go-docker:latest | jq
```
We will use the `linux/amd64` image in the below example

```json
{
  "schemaVersion": 2,
  "mediaType": "application/vnd.oci.image.index.v1+json",
  "manifests": [
    {
      "mediaType": "application/vnd.oci.image.manifest.v1+json",
      "size": 1518,
      "digest": "sha256:7af472edd1db5092ae42e1de8b81ad6fb906ad41a37cc5c3ff512b63c7dcb133",
      "platform": {
        "architecture": "amd64",
        "os": "linux"
      },
      "artifactType": "application/vnd.oci.image.config.v1+json"
    },
    {
      "mediaType": "application/vnd.oci.image.manifest.v1+json",
      "size": 1518,
      "digest": "sha256:b4052f29bad1103c786521ec3d9c075ebd41c310965a937a6da6b1f2c014220b",
      "platform": {
        "architecture": "arm64",
        "os": "linux"
      },
      "artifactType": "application/vnd.oci.image.config.v1+json"
    }
  ],
  "annotations": {
    "org.opencontainers.image.base.digest": "sha256:bb5aaba0ab4dead28db13371528ca6c44e2e0653565c2b71301193efaaf6b969",
    "org.opencontainers.image.base.name": "cgr.dev/chainguard/static:latest",
    "org.opencontainers.image.description": "Docker image version of https://github.com/simulot/immich-go"
  }
}
```

```shell
slsa-verifier verify-image ghcr.io/userbradley/immich-go-docker@sha256:b4052f29bad1103c786521ec3d9c075ebd41c310965a937a6da6b1f2c014220b \
  --source-uri "github.com/userbradley/immich-go-docker"
```

If you want to view the provennence file

```shell
slsa-verifier verify-image ghcr.io/userbradley/immich-go-docker@sha256:b4052f29bad1103c786521ec3d9c075ebd41c310965a937a6da6b1f2c014220b \
  --source-uri "github.com/userbradley/immich-go-docker" \
  --print-provenance | jq
```
