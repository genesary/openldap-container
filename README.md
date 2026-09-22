# Bitnami-compatible OpenLDAP image

A drop-in replacement for the [Bitnami OpenLDAP image](https://github.com/bitnami/containers/blob/main/bitnami/openldap/README.md), built entirely from open-source components.

**Key differences from the official Bitnami image:**
- OpenLDAP is compiled directly from the [upstream sources](https://github.com/openldap/openldap) (no Bitnami-built tarball)
- Base image is `debian:13-slim` instead of Bitnami's `minideb`

## Quick start

```yaml
# docker-compose.yml
services:
  openldap:
    image: ghcr.io/genesary/openldap:latest
    ports:
      - '1389:1389'
      - '1636:1636'
    environment:
      - LDAP_ADMIN_USERNAME=admin
      - LDAP_ADMIN_PASSWORD=adminpassword
      - LDAP_USERS=user01,user02
      - LDAP_PASSWORDS=password1,password2
    volumes:
      - openldap_data:/bitnami/openldap

volumes:
  openldap_data:
```

## Available tags

| Tag                                          | OpenLDAP version | Base OS        |
|----------------------------------------------|------------------|----------------|
| `latest`, `2.7`, `2.7.1`, `2.7.1-debian-13`  | 2.7.1            | Debian 13 slim |
| `2.6`, `2.6.15`, `2.6.15-debian-13`          | 2.6.15           | Debian 13 slim |

`latest` follows the most recent release line, currently 2.7. Pin `2.6` to stay on the previous line.

Both lines are built from the same base image and ship the same modules, entrypoint scripts, and configuration variables; only the OpenLDAP sources differ.

Every build also publishes an immutable tag `<version>-debian-13-<short-sha>`, where `<short-sha>` is the 7-character Git commit the image was built from. The tags above are mutable and always point to the most recent build; use the commit-suffixed tag to pin a specific one. The same value is exposed inside the image as `IMAGE_REVISION`.

Images are published to [ghcr.io/genesary/openldap](https://github.com/genesary/openldap/pkgs/container/openldap) for `linux/amd64` and `linux/arm64`.

## Compiled modules

In addition to the standard overlays, the following contrib modules are included:

| Module      | Description                                                |
|-------------|------------------------------------------------------------|
| `autogroup` | Automatic group membership management                      |
| `lastbind`  | Records the last successful bind time                      |
| `pbkdf2`    | PBKDF2 password hashing scheme                             |
| `sha2`      | SHA-2 password hashing schemes (SHA-256, SHA-384, SHA-512) |
| `smbk5pwd`  | Samba and Kerberos password synchronization                |

## Configuration

This image is fully compatible with the Bitnami OpenLDAP image configuration. All environment variables, volumes, and entrypoint scripts behave identically.

See the [Bitnami OpenLDAP documentation](https://github.com/bitnami/containers/blob/main/bitnami/openldap/README.md) for the full list of configuration options.

## Building locally

Each release line has its own directory, `2.6/debian-13` and `2.7/debian-13`:

```sh
cd 2.7/debian-13
docker build -t openldap:local .
```

This uses the OpenLDAP version set as the `ARG` defaults in that Dockerfile. To build a different release, pass both arguments — the Git tag mirrors the version with underscores:

```sh
docker build \
  --build-arg OPENLDAP_VERSION=2.7.1 \
  --build-arg OPENLDAP_GIT_TAG=OPENLDAP_REL_ENG_2_7_1 \
  -t openldap:local .
```

Published images are built by the CI workflow, which sets `OPENLDAP_VERSION` itself and is the source of truth for the released version.

## License

Apache 2.0 — see [LICENSE](LICENSE).
