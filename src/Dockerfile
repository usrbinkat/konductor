FROM centos:latest
#################################################################################
# Package Lists
ARG varListRpms="\
             git \
             vim \
             sudo \
             tree \
             tmux \
             rsync \
             bsdtar \
             podman \
             openssl \
             buildah \
             ncurses \
             python3-pip \
             fuse-overlayfs \
             openssh-server \
             openssh-clients \
             bash-completion \
             "
ARG varListPip="\
             boto3 \
             awscli \
             ansible \
             "

#################################################################################
# version tracking
ARG varVerOpenshift="${varVerOpenshift}"
ARG varVerTerraform="${varVerTerraform}"
ARG varVerJq="${varVerJq}"

# Build Variables
ARG varDirDependencies="/tmp/dependencies"
ARG varUrlOsInst="https://mirror.openshift.com/pub/openshift-v4/x86_64/clients/ocp/${varVerOpenshift}/openshift-install-linux.tar.gz"
ARG varUrlOpenshift="https://mirror.openshift.com/pub/openshift-v4/x86_64/clients/ocp/${varVerOpenshift}/openshift-client-linux.tar.gz"
ARG varUrlTerraform="https://releases.hashicorp.com/terraform/${varVerTerraform}/terraform_${varVerTerraform}_linux_amd64.zip"
ARG varUrlJq="https://github.com/stedolan/jq/releases/download/jq-${varVerJq}/jq-linux64"

#################################################################################
RUN set -ex                                                                     \
     && mkdir -p ${varDirDependencies}/pip                                      \
     && mkdir -p ${varDirDependencies}/rpm                                      \
     && mkdir -p ${varDirDependencies}/curl                                     \
    && echo

RUN set -ex                                                                     \
     && yum update  -y --setopt=tsflags=nodocs                                  \
     && yum install -y --setopt=tsflags=nodocs python3-pip                      \
    && echo

# Gather Dependencies | rpm
WORKDIR /tmp/builder/artifacts/rpm
RUN set -ex                                                                     \
     && yum install -y                                                          \
          --downloadonly                                                        \
          --downloaddir=${varDirDependencies}/rpm                               \
          ${varListRpms}                                                        \
    && echo

# Gather Dependencies | curl
WORKDIR ${varDirDependencies}/curl
RUN set -ex                                                                     \
     && curl -LO ${varUrlOpenshift}                                             \
     && curl -LO ${varUrlTerraform}                                             \
     && curl -LO ${varUrlOsInst}                                                \
     && curl -LO ${varUrlJq}                                                    \
    && echo

# Gather Dependencies | Pip
WORKDIR ${varDirDependencies}/pip
RUN set -ex                                                                     \
     && pip3 download                                                           \
            -d ${varDirDependencies}/pip                                        \
          ${varListPip}                                                         \
    && echo

# Create Unified Tarball
WORKDIR /tmp/
RUN set -ex                                                                     \
     && tar -czvf dependencies.tar.gz dependencies                              \
     && du -sh dependencies.tar.gz                                              \
    && echo

# Cleanup container
RUN yum clean all                                                               \
    && history -c                                                               \
    && rm -rf                                                                   \
        /tmp/dependencies                                                       \
        /var/cache/*                                                            \
        /var/log/dnf*                                                           \
        /var/log/yum*                                                           \
   ; echo

#################################################################################
# ContainerOne | Cloud Orchestration Tools - Point of Origin
ENV versOCP="${varVerOpenshift}"
LABEL Vendor="containercraft.io"                                                \
      OpenShift="${versOCP}"                                                    \
      BuildDate="${varRunDate}"                                                 \
      maintainer="ContainerCraft.io"                                            \
      License=GPLv3 
CMD ["bash"]
