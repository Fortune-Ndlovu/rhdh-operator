# THIS IS USED BY OSBS BUILDS <=1.3
# FOR KONFLUX >=1.4, see ../../docker/Dockerfile

#@follow_tag(registry.redhat.io/rhel9/go-toolset:latest)
# https://registry.access.redhat.com/ubi9/go-toolset
FROM registry.access.redhat.com/ubi9/go-toolset:9.5-1731639025@sha256:45170b6e45114849b5d2c0e55d730ffa4a709ddf5f58b9e810548097b085e78f AS builder
# hadolint ignore=DL3002
USER 0
ENV GOPATH=/go/
# update RPMs
RUN dnf -q -y update

# Upstream sources
# Downstream comment
# ENV EXTERNAL_SOURCE=.
# ENV CONTAINER_SOURCE=/opt/app-root/src
# WORKDIR /workspace
#/ Downstream comment

# Downstream sources
# Downstream uncomment
ENV EXTERNAL_SOURCE=$REMOTE_SOURCES/upstream1/app/distgit/containers/rhdh-operator
ENV CONTAINER_SOURCE=$REMOTE_SOURCES_DIR
WORKDIR $CONTAINER_SOURCE/
#/ Downstream uncomment

COPY $EXTERNAL_SOURCE ./

# cache deps before building and copying source so that we don't need to re-download as much
# and so that source changes don't invalidate our downloaded layer
# Downstream comment
# RUN go mod download
#/ Downstream comment

# Downstream uncomment
COPY $REMOTE_SOURCES/upstream1/cachito.env ./
RUN source ./cachito.env && rm -f ./cachito.env && mkdir -p /workspace
#/ Downstream uncomment

# Build
# hadolint ignore=SC3010
RUN export ARCH="$(uname -m)" && if [[ ${ARCH} == "x86_64" ]]; then export ARCH="amd64"; elif [[ ${ARCH} == "aarch64" ]]; then export ARCH="arm64"; fi && \
    CGO_ENABLED=1 GOOS=linux GOARCH=${ARCH} go build -a -o manager main.go

# Install openssl for FIPS support
#@follow_tag(registry.redhat.io/ubi9/ubi-minimal:latest)
# https://registry.access.redhat.com/ubi9/ubi-minimal
FROM registry.access.redhat.com/ubi9-minimal:9.5-1731604394@sha256:46f77b7dfba47b041de4c9d8516c24081fc92cc7743fca4a353e7f1c2a4beb19 AS runtime
RUN microdnf update --setopt=install_weak_deps=0 -y && microdnf install -y openssl; microdnf clean -y all

# RHIDP-4220 - make Konflux preflight and EC checks happy - [check-container] Create a directory named /licenses and include all relevant licensing
COPY $EXTERNAL_SOURCE/LICENSE /licenses/

# Upstream sources
# Downstream comment
# ENV CONTAINER_SOURCE=/workspace
#/ Downstream comment

# Downstream sources
# Downstream uncomment
ENV CONTAINER_SOURCE=$REMOTE_SOURCES_DIR
#/ Downstream uncomment

ENV HOME=/ \
    USER_NAME=backstage \
    USER_UID=1001

RUN echo "${USER_NAME}:x:${USER_UID}:0:${USER_NAME} user:${HOME}:/sbin/nologin" >> /etc/passwd

# Copy manager binary
COPY --from=builder $CONTAINER_SOURCE/manager .

USER ${USER_UID}

WORKDIR ${HOME}

ENTRYPOINT ["/manager"]

# append Brew metadata here
