# Copyright Broadcom, Inc. All Rights Reserved.
# SPDX-License-Identifier: APACHE-2.0

FROM debian:12 AS build

RUN apt-get update -y && \
  DEBIAN_FRONTEND="noninteractive" apt-get install -y --no-install-recommends \
    build-essential \
    ca-certificates \
    check \
    cmake \
    curl \
    git \
    gnutls-dev \
    libaio-dev \
    libasio-dev \
    libboost-dev \
    libboost-program-options-dev \
    libjemalloc-dev \
    libncurses-dev \
    libpam0g-dev \
    scons
RUN \
  curl https://archive.mariadb.org/mariadb-11.4.7/source/mariadb-11.4.7.tar.gz --output /tmp/mariadb-11.4.7.tar.gz && \
  tar --no-same-owner -C /tmp -xf /tmp/mariadb-11.4.7.tar.gz && \
  cd /tmp/mariadb-11.4.7 && \
  cmake \
    -DCMAKE_INSTALL_PREFIX=/opt/bitnami/mariadb \
    -DDEFAULT_SYSCONFDIR=/opt/bitnami/mariadb/conf \
    -DSYSCONFDIR=/opt/bitnami/mariadb/conf \
    -DINSTALL_SBINDIR=sbin \
    -DINSTALL_SCRIPTDIR=/opt/bitnami/mariadb/bin \
    -DINSTALL_PLUGINDIR=plugin \
    -DINSTALL_INFODIR=/opt/bitnami/mariadb/info \
    -DINSTALL_SUPPORTFILESDIR=/opt/bitnami/mariadb/share \
    -DMYSQL_DATADIR=/opt/bitnami/mariadb/data \
    -DCURSES_INCLUDE_PATH=/usr/include/ \
    -DMYSQL_UNIX_ADDR=/opt/bitnami/mariadb/tmp/mysql.sock \
    -DENABLED_LOCAL_INFILE=ON \
    -DWITH_INNOBASE_STORAGE_ENGINE=1 \
    -DWITH_ARCHIVE_STORAGE_ENGINE=1 \
    -DWITH_BLACKHOLE_STORAGE_ENGINE=1 \
    -DWITH_PERFSCHEMA_STORAGE_ENGINE=1 \
    -DWITH_FEDERATED_STORAGE_ENGINE=1 \
    -DWITH_PARTITION_STORAGE_ENGINE=1 \
    -DPLUGIN_AUTH_PAM=YES && \
  make --jobs=5 && \
  make install --jobs=5 && \
  git clone --branch release_26.4.23 https://github.com/codership/galera.git /tmp/libgalera && \
  cd /tmp/libgalera && \
  git submodule update --init --recursive && \
  ./scripts/build.sh

FROM docker.io/bitnami/minideb:bookworm

ARG DOWNLOADS_URL="downloads.bitnami.com/files/stacksmith"
ARG TARGETARCH

LABEL com.vmware.cp.artifact.flavor="sha256:c50c90cfd9d12b445b011e6ad529f1ad3daea45c26d20b00732fae3cd71f6a83" \
      org.opencontainers.image.base.name="docker.io/bitnami/minideb:bookworm" \
      org.opencontainers.image.created="2025-06-10T12:47:02Z" \
      org.opencontainers.image.description="Application packaged by Broadcom, Inc." \
      org.opencontainers.image.documentation="https://github.com/bitnami/containers/tree/main/bitnami/mariadb-galera/README.md" \
      org.opencontainers.image.ref.name="11.4.7-debian-12-r3" \
      org.opencontainers.image.source="https://github.com/bitnami/containers/tree/main/bitnami/mariadb-galera" \
      org.opencontainers.image.title="mariadb-galera" \
      org.opencontainers.image.vendor="Broadcom, Inc." \
      org.opencontainers.image.version="11.4.7"

ENV HOME="/" \
    OS_ARCH="${TARGETARCH:-amd64}" \
    OS_FLAVOUR="debian-12" \
    OS_NAME="linux"

COPY prebuildfs /
SHELL ["/bin/bash", "-o", "errexit", "-o", "nounset", "-o", "pipefail", "-c"]
# Install required system packages and dependencies
RUN install_packages ca-certificates curl iproute2 ldap-utils libaio1 libaudit1 libcap-ng0 libcrypt1 libgcc-s1 libicu72 libldap-common liblzma5 libncurses6 libpam-ldapd libpam0g libssl3 libstdc++6 libtinfo6 libxml2 nslcd procps psmisc rsync socat zlib1g
RUN mkdir -p /tmp/bitnami/pkg/cache/ ; cd /tmp/bitnami/pkg/cache/ || exit 1 ; \
    COMPONENTS=( \
      "ini-file-1.4.7-17-linux-${OS_ARCH}-debian-12" \
    ) ; \
    for COMPONENT in "${COMPONENTS[@]}"; do \
      if [ ! -f "${COMPONENT}.tar.gz" ]; then \
        curl -SsLf "https://${DOWNLOADS_URL}/${COMPONENT}.tar.gz" -O ; \
        curl -SsLf "https://${DOWNLOADS_URL}/${COMPONENT}.tar.gz.sha256" -O ; \
      fi ; \
      sha256sum -c "${COMPONENT}.tar.gz.sha256" ; \
      tar -zxf "${COMPONENT}.tar.gz" -C /opt/bitnami --strip-components=2 --no-same-owner ; \
      rm -rf "${COMPONENT}".tar.gz{,.sha256} ; \
    done
COPY --from=build /opt/bitnami/mariadb /opt/bitnami/mariadb
RUN apt-get update && apt-get upgrade -y && \
    apt-get clean && rm -rf /var/lib/apt/lists /var/cache/apt/archives
RUN chmod g+rwX /opt/bitnami
RUN find / -perm /6000 -type f -exec chmod a-s {} \; || true
RUN mkdir /docker-entrypoint-initdb.d
RUN uninstall_packages curl

COPY rootfs /
RUN /opt/bitnami/scripts/mariadb-galera/postunpack.sh
ENV APP_VERSION="11.4.7" \
    BITNAMI_APP_NAME="mariadb-galera" \
    PATH="/opt/bitnami/common/bin:/opt/bitnami/common/sbin:/opt/bitnami/mariadb/bin:/opt/bitnami/mariadb/sbin:$PATH"

EXPOSE 3306 4444 4567 4568

USER 1001
ENTRYPOINT [ "/opt/bitnami/scripts/mariadb-galera/entrypoint.sh" ]
CMD [ "/opt/bitnami/scripts/mariadb-galera/run.sh" ]
