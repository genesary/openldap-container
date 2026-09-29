# AI Agent Guide

This file provides guidance to AI coding agents when working with code in this repository.

## What this repo is

A Bitnami-compatible OpenLDAP Docker image, built as a drop-in replacement for the official
[Bitnami OpenLDAP image](https://github.com/bitnami/containers/blob/main/bitnami/openldap/README.md).
The two key differences from Bitnami's image: OpenLDAP is compiled from upstream source (no
Bitnami-built tarball), and the base image is `debian:13-slim` instead of `minideb`.

## Repository layout

Each supported OpenLDAP release line lives in its own directory, entirely self-contained:

- `2.6/debian-13/` — OpenLDAP 2.6.x line
- `2.7/debian-13/` — OpenLDAP 2.7.x line

Both directories mirror the same structure and are meant to stay in sync (same modules, entrypoint
scripts, and config variables — only the OpenLDAP source version differs):

- `Dockerfile` — multi-stage build: compiles OpenLDAP + contrib modules from source in a `cpp:dev-debian13`
  build stage, then copies the result into a slim `debian:13-slim` runtime stage.
- `customize.sh` — post-`make install` filesystem reshuffle (moves libexec/subdir layout to match
  Bitnami's expected paths) run inside the build stage.
- `prebuildfs/` — files copied into the runtime image *before* installing OS packages (helper
  scripts like `install_packages`, and the `opt/bitnami/scripts/lib*.sh` shared shell libraries
  copied from the Bitnami containers project).
- `rootfs/` — files copied into the runtime image *after* the OpenLDAP build artifacts are in place:
  `opt/bitnami/scripts/openldap/{entrypoint,run,setup,postunpack}.sh` and `libopenldap.sh`. This is
  where OpenLDAP-specific runtime behavior lives (env var parsing, LDIF bootstrap, `slapd` flag
  assembly in `run.sh`).
- `docker-compose.yml` — local convenience compose file for the image.

The Helm chart lives at the repository root, in `chart/`. It deploys the OpenLDAP 2.7 image on
Kubernetes, follows Bitnami's chart conventions and builds on Bitnami's `common` library chart for
shared template helpers.

OpenLDAP 2.6 is in LTS mode and is no longer maintained by Bitnami. As a result, only the Dockerfile
is likely to need changes, to point at the latest 2.6 patch release.

OpenLDAP 2.7, on the other hand, is still maintained by Bitnami, which means the scripts under
`prebuildfs` and `rootfs` need to be kept in sync with Bitnami's changes to its own repo, just like
the Dockerfile.

## Building locally

```sh
cd 2.7/debian-13   # or 2.6/debian-13
docker build -t openldap:local .
```

Uses the `ARG` defaults baked into that Dockerfile. To build a specific version, pass both args (the
Git tag mirrors the version with underscores):

```sh
docker build \
  --build-arg OPENLDAP_VERSION=2.7.1 \
  --build-arg OPENLDAP_GIT_TAG=OPENLDAP_REL_ENG_2_7_1 \
  -t openldap:local .
```

The `ARG` defaults in each Dockerfile are for local builds only — CI sets `OPENLDAP_VERSION` itself
via the workflow `env:` block and is the source of truth for published versions.

## CI / publishing

`.github/workflows/openldap-2-6.yaml` and `openldap-2-7.yaml` each build and push their respective
image to `ghcr.io/genesary/openldap` on push to `main`, for `linux/amd64` and `linux/arm64`. The
`OPENLDAP_VERSION` env var in each workflow is what actually determines the published version —
bumping a release means editing that value (and the matching `ARG` default in the Dockerfile, to
keep local builds consistent). Images get mutable tags (`latest`, `<minor>`, `<version>`) plus an
immutable `<version>-debian-13-<short-sha>` tag, and a build attestation is published alongside.

Dependabot (`.github/dependabot.yml`) tracks `github-actions` updates at the repo root and `docker`
base-image updates separately for each of `2.6/debian-13` and `2.7/debian-13`.

There is no test suite or lint workflow in CI; the devcontainer includes `shellcheck` for manually
checking the `prebuildfs`/`rootfs` shell scripts.
