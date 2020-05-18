FROM centos:latest
################################################################################
# ContainerOne | Cloud Orchestration Tools - Point of Origin
LABEL Vendor="containercraft.io" \
      maintainer="ContainerCraft.io" \
      License=GPLv3 

################################################################################
# version tracking
#varRunDate="$(date +%y%m%d%I%M%S)"
varVerOpenshift="4.3.8"
varVerTerraform="0.12.25"
varVerJq="1.6"
varUrlOsInst="https://mirror.openshift.com/pub/openshift-v4/x86_64/clients/ocp/${versOCP}/openshift-install-linux.tar.gz"
varUrlOpenshift="https://mirror.openshift.com/pub/openshift-v4/x86_64/clients/ocp/${versOCP}/openshift-client-linux.tar.gz"
varUrlTerraform="https://releases.hashicorp.com/terraform/${varVerTerraform}/terraform_${varVerTerraform}_linux_amd64.zip"
varUrlJq="https://github.com/stedolan/jq/releases/download/jq-${varVerJq}/jq-linux64"
rpmList="\
        tree \
        tmux \
        podman \
        buildah \
        fuse-overlayfs  \
        openssh-server \
        bash-completion \
        "
pipList="\
        boto3 \
        awscli \
        ansible \
        "

################################################################################
RUN set -ex \
     && mkdir -p /tmp/builder/artifacts/pip  \
     && mkdir -p /tmp/builder/artifacts/rpm  \
     && mkdir -p /tmp/builder/artifacts/curl \
    && echo

RUN set -ex \
     && yum update  -y --setopt=tsflags=nodocs \
     && yum install -y --setopt=tsflags=nodocs python3-pip \
    && echo

# Change base workdir
WORKDIR /tmp/builder/artifacts/rpm

RUN set -ex \
     && yum install \
          --downloadonly \
          --downloaddir=/tmp/builder/artifacts/rpm \
          ${rpmList} \
    && echo

# Change base workdir
WORKDIR /tmp/builder/artifacts/curl

RUN set -ex \
     && curl -LO ${varUrlOpenshift} \
     && curl -LO ${varVerTerraform} \
     && curl -LO ${varUrlOsInst}    \
     && curl -LO ${varVerJq}        \
    && echo

# Change base workdir
WORKDIR /tmp/builder/artifacts/pip

RUN set -ex \
     && pip3 download \
            -d /builder/artifacts/pip \
          ${pipList}                  \
    && echo

# Change base workdir
WORKDIR /tmp/
RUN tar -czvf artifacts.tar.gz builder/artifacts

RUN yum clean all \
    && rm -rf \
        /tmp/builder/artifacts \
        /var/cache \
        /var/log/dnf* \
        /var/log/yum.* \
   ; echo

ENV versOCP="${varVerOpenshift}"
ENTRYPOINT ["/bin/bash"]
