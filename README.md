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

```

<!-- TOC -->
* [Immich Go CLI, as an OCI Image](#immich-go-cli-as-an-oci-image)
  * [Why this?](#why-this)
  * [How to use](#how-to-use)
  * [FAQ](#faq)
    * [You talk about OCI, but the repo is named `immich-go-docker`](#you-talk-about-oci-but-the-repo-is-named-immich-go-docker)
    * [How do I know you've not fiddled with the images?](#how-do-i-know-youve-not-fiddled-with-the-images)
    * [How does the CI work](#how-does-the-ci-work)
    * [What tags are available?](#what-tags-are-available)
    * [Is there any AI usage on this repo?](#is-there-any-ai-usage-on-this-repo)
  * [Signing, SBOM and SLSA](#signing-sbom-and-slsa)
    * [Verify image signature](#verify-image-signature)
    * [Verify SLSA compliance](#verify-slsa-compliance)
      * [View SLSA Provenance](#view-slsa-provenance)
<!-- TOC -->

## FAQ

### You talk about OCI, but the repo is named `immich-go-docker`

This is because most people will google `Immich go docker` looking for an image

### How do I know you've not fiddled with the images?

The CI used to build the images is available in this repo, and images are signed, SBOM'med and SLSA attested.

Older version of the image, 

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

All images are signed at both the tag, and the individual digests

### Verify image signature

You will need to `crane` command line tool installed, ideally

### Verify SLSA compliance

You can view the SLSA details of an image using the below.

Let's assume we want to see if SLSA is present on `ghcr.io/userbradley/immich-go-docker:40312388416-v0.32.0` (Note, this is an example!)

```shell
crane manifest ghcr.io/userbradley/immich-go-docker:40312388416-v0.32.0 | jq
```

You get an output like

```shell
{
  "schemaVersion": 2,
  "mediaType": "application/vnd.oci.image.index.v1+json",
  "manifests": [
    {
      "mediaType": "application/vnd.oci.image.manifest.v1+json",
      "size": 1416,
      "digest": "sha256:3038daaa8b7816150781f8e5168016ec872b35483ee9d17687a9c777077caacc",
      "platform": {
        "architecture": "amd64",
        "os": "linux"
      },
      "artifactType": "application/vnd.oci.image.config.v1+json"
    },
    {
      "mediaType": "application/vnd.oci.image.manifest.v1+json",
      "size": 1416,
      "digest": "sha256:c17ca9ad5f871bc86d6272f2e71ab57877a7e97cf6c307b98f74d83a0d66ab36",
      "platform": {
        "architecture": "arm64",
        "os": "linux"
      },
      "artifactType": "application/vnd.oci.image.config.v1+json"
    }
  ],
  "annotations": {
    "org.opencontainers.image.base.digest": "sha256:1144e6830320ccf2d7a607a02d6ffa396fe6bdc7927c47facc8415e767dd6cd3",
    "org.opencontainers.image.base.name": "cgr.dev/chainguard/static:latest"
  }
}
```

If we want to see the SLSA for the `linux/amd64` image we get the `digest`

```shell
slsa-verifier verify-image ghcr.io/userbradley/immich-go-docker@sha256:c17ca9ad5f871bc86d6272f2e71ab57877a7e97cf6c307b98f74d83a0d66ab36 --source-uri "github.com/userbradley/immich-go-docker"
```

This gives an output like

```text
Verified build using builder "https://github.com/slsa-framework/slsa-github-generator/.github/workflows/generator_container_slsa3.yml@refs/tags/v2.1.0" at commit c28217d53172880877d62b4fec3c08c1cb1e77a8
PASSED: SLSA verification passed
```

#### View SLSA Provenance

Prepend the `slsa-verifier` command with `--print-provenance | jq`

```shell
slsa-verifier verify-image ghcr.io/userbradley/immich-go-docker@sha256:c17ca9ad5f871bc86d6272f2e71ab57877a7e97cf6c307b98f74d83a0d66ab36 --source-uri "github.com/userbradley/immich-go-docker" --print-provenance | jq
```
