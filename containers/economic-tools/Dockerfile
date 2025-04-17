# Copyright (C) 2025 Jaremy Hatler
# SPDX-License-Identifier: MIT

##
# Inspired by https://github.com/qdm12/latexdevcontainer
##

#checkov:skip=CKV_DOCKER_2: No healthcheck needed
#checkov:skip=CKV_DOCKER_3: User handled by devcontainer

# Build entroy tool for use in final stage
FROM mcr.microsoft.com/devcontainers/python:bookworm AS entropy_build
ARG ENTROPY_VERSION=1.1
ENV ENTROPY_FILE=v${ENTROPY_VERSION}.tar.gz
ENV ENTROPY_URL=https://github.com/merces/entropy/archive/refs/tags/${ENTROPY_FILE}
ENV DEBIAN_FRONTEND=noninteractive

WORKDIR /usr/local/src
# hadolint ignore=DL3003
RUN apt-get update \
 && apt-get install -y --no-install-recommends \
        build-essential=12.9 \
        cmake=3.25.1-1 \
 && curl -sSLO "${ENTROPY_URL}" \
 && tar -xzf "${ENTROPY_FILE}" \
 && cd "entropy-${ENTROPY_VERSION}" \
 && mkdir build \
 && cd build \
 && cmake .. \
 && make \
 && mv entropy /usr/local/bin

# Functional stage
FROM mcr.microsoft.com/devcontainers/python:bookworm AS functional
SHELL ["/bin/bash", "-o", "pipefail", "-e", "-c"]
ENV SHELL=/bin/bash
ENV DEBIAN_FRONTEND=noninteractive

# hadolint ignore=DL3003
RUN apt-get update \
 && apt-get install -y --no-install-recommends \
        build-essential=12.9 \
        wget=1.21.3-1+deb12u1 \
        gnupg=2.2.40-1.1 \
        bash-completion=1:2.11-6 \
 && apt-get autoremove --purge -y \
 && apt-get clean \
 && rm -rf /var/lib/apt/lists/* \
 && find /var/cache/apt -type f -print0 | xargs -0 rm -f

COPY --from=entropy_build /usr/local/bin/entropy /usr/local/bin/entropy

# Testing stage - verify binaries work and have the right permissions
FROM functional AS testing
RUN entropy

# Final stage
FROM functional AS final
