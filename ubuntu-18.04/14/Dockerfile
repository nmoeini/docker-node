FROM ubuntu:bionic-20200807

LABEL name="nmoeini/node:14-bionic-20200807" mainteiner="Navid Moeini <nmoeiny@yahoo.com>" version="0.0"

ARG DEBIAN_FRONTEND
ARG NODE_VERSION
ARG DISTRO
ARG SVC_USER
ARG UID
ARG NPM_CONFIG_LOGLEVEL

ENV NPM_CONFIG_LOGLEVEL=${NPM_CONFIG_LOGLEVEL:-info} \
    NODE_VERSION=${NODE_VERSION:-14} \
    DISTRO=${DISTRO:-bionic} \
    SVC_USER=${SVC_USER:-node} \
    UID=${UID:-999} \
    DEBIAN_FRONTEND=${DEBIAN_FRONTEND:-noninteractive}

RUN \
    \
##  Create the node.js system user \
##  ------------------------------------------------------------------------  ## \
    groupadd --system --force --gid "${UID}" "${SVC_USER}" \
    && useradd --system --uid "${UID}" --gid "${SVC_USER}" --shell /bin/bash --create-home "${SVC_USER}"

RUN set -ex \
    && apt-get update -q -y \
    \
##  Tools Setup \
##  ------------------------------------------------------------------------  ## \
    && apt-get install -y --no-install-recommends \
        ca-certificates \
        curl \
        mc \
        wget \
        git \
        htop \
        zip \
        gnupg \
    \
##  Cleanup \
##  -----------------------------------------------------------------------  ## \
    && apt-get purge -y --auto-remove -o APT::AutoRemove::RecommendsImportant=false \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*

RUN set -ex \
    \
##  Add Node Repository \
##  ------------------------------------------------------------------------  ## \
    && curl -fsSL -k https://deb.nodesource.com/gpgkey/nodesource.gpg.key | apt-key add - \
    && echo "deb https://deb.nodesource.com/node_"${NODE_VERSION}".x $DISTRO main" | tee /etc/apt/sources.list.d/nodesource.list \
    \
##  Add Node \
##  ------------------------------------------------------------------------  ## \
    && apt-get update -q -y \
    && apt-get install -y --no-install-recommends nodejs \
    && node --version \
    && npm --version

RUN set -ex \
    \
##  Add utills \
## ------------------------------------------------------------------------  ## \
    && apt-get install -y --no-install-recommends \
        gcc \
        g++ \
        make \
    \
##  Add Yarn \
## ------------------------------------------------------------------------  ## \
    && curl -sL -k https://dl.yarnpkg.com/debian/pubkey.gpg | apt-key add - \
    && echo "deb https://dl.yarnpkg.com/debian/ stable main" | tee /etc/apt/sources.list.d/yarn.list \
    && apt-get update \
    && apt-get install -y --no-install-recommends yarn \
    && yarn --version \
    \
##  Cleanup \
##  -----------------------------------------------------------------------  ## \
    && apt-get purge -y --auto-remove -o APT::AutoRemove::RecommendsImportant=false \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*

ADD docker-node-entrypoint /usr/local/bin/

RUN chmod +x /usr/local/bin/docker-node-entrypoint

ENTRYPOINT ["docker-node-entrypoint"]

CMD [ "node" ]