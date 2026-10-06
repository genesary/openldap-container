# SPDX-License-Identifier: APACHE-2.0

# Defaults for local builds only. The CI workflow overrides both values and is
# the source of truth for published images.
ARG OPENLDAP_VERSION=2.6.14
ARG OPENLDAP_GIT_TAG=OPENLDAP_REL_ENG_2_6_14

FROM mcr.microsoft.com/vscode/devcontainers/cpp:dev-debian13@sha256:c53c8c707da7fa4a374ef2551ad3d64560929c39d0293c279464b63bec5d71e6 AS build

ARG OPENLDAP_GIT_TAG

ADD https://github.com/openldap/openldap.git#${OPENLDAP_GIT_TAG} /tmp/openldap

COPY customize.sh /tmp/openldap/customize.sh

# Build and install OpenLDAP from source
WORKDIR /tmp/openldap

ENV PREFIX="/opt/bitnami/openldap" \
    SUBDIR="/openldap"

RUN apt-get update && \
    apt-get install -y \
        libargon2-dev \
        libsasl2-dev \
        libtool && \
    ./configure \
        --enable-argon2 \
        --enable-overlays=mod \
        --enable-modules \
        --enable-spasswd \
        --prefix=${PREFIX} \
        --with-argon2=libargon2 \
        --with-cyrus-sasl \
        --with-subdir=${SUBDIR} && \ 
    make depend && \
    make && \
    make install

RUN ./customize.sh ${PREFIX} ${SUBDIR}

# Build and install additional OpenLDAP autogroup module
WORKDIR /tmp/openldap/contrib/slapd-modules/autogroup

RUN make prefix=${PREFIX} && \
    make install prefix=${PREFIX}

# Build and install additional OpenLDAP lastbind module
WORKDIR /tmp/openldap/contrib/slapd-modules/lastbind

RUN make prefix=${PREFIX} && \
    make install prefix=${PREFIX}

# Build and install additional OpenLDAP pbkdf2 modules
WORKDIR /tmp/openldap/contrib/slapd-modules/passwd/pbkdf2

RUN make prefix=${PREFIX} && \
    make install prefix=${PREFIX}

# Build and install additional OpenLDAP sha2 modules
WORKDIR /tmp/openldap/contrib/slapd-modules/passwd/sha2

RUN make prefix=${PREFIX} && \
    make install prefix=${PREFIX}

# Build and install additional OpenLDAP smbk5pwd module
WORKDIR /tmp/openldap/contrib/slapd-modules/smbk5pwd

RUN apt-get install -y heimdal-multidev && \
    make \
        prefix=${PREFIX} \
        HEIMDAL_INC="$(/usr/bin/krb5-config.heimdal --cflags)" \
        HEIMDAL_LIB="$(/usr/bin/krb5-config.heimdal --libs)" && \
    make install prefix=${PREFIX}

# Clean up unnecessary files to reduce image size
RUN rm -rf ${PREFIX}/share/man && \
    find ${PREFIX} -name "*.la" -delete && \
    find ${PREFIX} -name "*.a" -delete

FROM docker.io/debian:13-slim@sha256:a29215f6a35e51e22adffa17f89e9d2ef06214e64a2bad10d765c46aea49f11f

ENV HOME="/" \
    OS_FLAVOUR="debian-13" \
    OS_NAME="linux"

COPY prebuildfs /
SHELL ["/bin/bash", "-o", "errexit", "-o", "nounset", "-o", "pipefail", "-c"]
# Install required system packages and dependencies
RUN install_packages ca-certificates libargon2-1 libcap2-bin libcom-err2 libcrypt1 libgssapi-krb5-2 libk5crypto3 libkeyutils1 libkrb5-3 libkrb5support0 libltdl7 libnsl2 libnss3-tools libodbc2 libperl5.40 libsasl2-2 libssl3 libtirpc3 libwrap0 mdbtools procps psmisc

COPY --from=build /opt/bitnami/openldap /opt/bitnami/openldap

RUN apt-get update && apt-get upgrade -y && \
    apt-get clean && rm -rf /var/lib/apt/lists /var/cache/apt/archives
RUN chmod g+rwX /opt/bitnami
RUN find / -perm /6000 -type f -exec chmod a-s {} \; || true

COPY rootfs /
RUN /opt/bitnami/scripts/openldap/postunpack.sh
ARG OPENLDAP_VERSION
ARG IMAGE_REVISION=0
ENV APP_VERSION="${OPENLDAP_VERSION}" \
    BITNAMI_APP_NAME="openldap" \
    IMAGE_REVISION="${IMAGE_REVISION}" \
    PATH="/opt/bitnami/openldap/bin:/opt/bitnami/openldap/sbin:$PATH"

EXPOSE 1389 1636

USER 1001
ENTRYPOINT [ "/opt/bitnami/scripts/openldap/entrypoint.sh" ]
CMD [ "/opt/bitnami/scripts/openldap/run.sh" ]