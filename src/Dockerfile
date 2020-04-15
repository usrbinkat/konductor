################################################################################
# ContainerOne | Point of Origin for PlatformOne Devel & Deploy
FROM centos:latest
LABEL Vendor="containercraft.io" \
        maintainer="ContainerCraft.io" \
        License=GPLv3 

################################################################################
# Version Controls 

# OCP Version set in src/ocp
ARG versOCP="4.3.8"
ENV versOCP="${versOCP}"

# Other Versions
ARG versTF="0.12.24"
ARG versJQ="1.6"

ARG urlTF="https://releases.hashicorp.com/terraform/${versTF}/terraform_${versTF}_linux_amd64.zip"
ARG urlJQ="https://github.com/stedolan/jq/releases/download/jq-${versJQ}/jq-linux64"
ARG urlOC="https://mirror.openshift.com/pub/openshift-v4/x86_64/clients/ocp/${versOCP}/openshift-client-linux.tar.gz"
ARG urlOCINST="https://mirror.openshift.com/pub/openshift-v4/x86_64/clients/ocp/${versOCP}/openshift-install-linux.tar.gz"
ARG yumList="\
            tree \
            tmux \
            podman \
            buildah \
            fuse-overlayfs  \
            openssh-server \
            bash-completion \
            "

RUN set -ex                                                       

RUN mkdir -p /tmp/archive/pip                                     \
    && mkdir -p /tmp/archive/yum                                  \
    $$ mkdir -p /tmp/archive/curl 

COPY src/requirements.txt /tmp/requirements.txt

RUN yum update -y --setopt=tsflags=nodocs                             \
    && yum install -y --setopt=tsflags=nodocs python3-pip

WORKDIR /tmp/archive/curl
RUN curl -LO ${urlOCINST}                                          \
    && curl -LO ${urlOC}                                           \
    && curl -LO ${urlTF}                                           \
    && curl -LO ${urlJQ}  

RUN pip3 download -d /tmp/archive/pip -r /tmp/requirements.txt ansible

WORKDIR /tmp/
RUN tar -czvf archive.tar.gz /tmp/archive

RUN yum clean all                                                 \
    && rm -rf /tmp/archive /var/cache /var/log/dnf* /var/log/yum.*\
    && echo

WORKDIR /
ENTRYPOINT ["/bin/bash"]