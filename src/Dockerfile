FROM registry:2 as registry
FROM centos:latest as rpm
#FROM registry.fedoraproject.org/fedora as rpm
FROM registry.access.redhat.com/ubi8/ubi:latest
#################################################################################
# OCP Version set in src/ocp
#ARG versOCP="$(curl -sL https://mirror.openshift.com/pub/openshift-v4/clients/ocp/latest/release.txt | awk '/ Version:/{print $2}')"
ARG versOCP="4.3.12"
ARG varVerOpenshift="${versOCP}"
ENV versOCP="${versOCP}"

# Binary URL Version Locks
ARG varVerTerraform="0.12.27"
ARG varVerHelm="3.2.4"
ARG varVerJq="1.6"

# OC Download Urls
ARG urlOC="https://mirror.openshift.com/pub/openshift-v4/x86_64/clients/ocp/${versOCP}/openshift-client-linux.tar.gz"
ARG urlOCINST="https://mirror.openshift.com/pub/openshift-v4/x86_64/clients/ocp/${versOCP}/openshift-install-linux.tar.gz"

# Binary Artifact URLS
ARG varUrlGcloud="https://sdk.cloud.google.com"
ARG varUrlHelm="https://get.helm.sh/helm-v${varVerHelm}-linux-amd64.tar.gz"
ARG varUrlTerraform="https://releases.hashicorp.com/terraform/${varVerTerraform}/terraform_${varVerTerraform}_linux_amd64.zip"
ARG varUrlOsInst="https://mirror.openshift.com/pub/openshift-v4/x86_64/clients/ocp/${varVerOpenshift}/openshift-install-linux.tar.gz"
ARG varUrlOpenshift="https://mirror.openshift.com/pub/openshift-v4/x86_64/clients/ocp/${varVerOpenshift}/openshift-client-linux.tar.gz"
ARG varUrlJq="https://github.com/stedolan/jq/releases/download/jq-${varVerJq}/jq-linux64"

#################################################################################
# Package Lists
ARG varListRpms="\
             git \
             vim \
             file \
             sudo \
             tree \
             tmux \
             rsync \
             skopeo \
             bsdtar \
             podman \
             iputils \
             iproute \
             ncurses \
             openssl \
             buildah \
             rpm-libs \
             nmap-ncat \
             libselinux \
             audit-libs \
             httpd-tools \
             libsemanage \
             python3-pip \
             podman-remote \
             iptables-libs \
             fuse-overlayfs \
             openssh-server \
             libvarlink-util \
             openssh-clients \
             bash-completion \
             "
ARG varListPip="\
             boto3 \
             awscli \
             ansible \
             "

# Build Variables
ARG varDirDependencies="/tmp/dependencies"

# Create Artifact Directories
RUN set -ex                                                                     \
     && mkdir -p ${varDirDependencies}/bin                                      \
     && mkdir -p ${varDirDependencies}/pip                                      \
     && mkdir -p ${varDirDependencies}/rpm                                      \
     && mkdir -p ${varDirDependencies}/gcloud                                   \
    && echo

#################################################################################
# Install Local Build Requirements
RUN set -ex                                                                     \
     && yum update  -y --setopt=tsflags=nodocs                                  \
     && yum install -y --setopt=tsflags=nodocs                                  \
                python3-pip                                                     \
                bsdtar                                                          \
                which                                                           \
    && echo

#################################################################################

# Gather Dependencies | rpm
WORKDIR /tmp/builder/artifacts/rpm
COPY --from=rpm /etc/yum.repos.d /etc/yum.repos.d
COPY --from=rpm /etc/pki /etc/pki
RUN set -ex                                                                     \
     && yum install -y                                                          \
          --downloadonly                                                        \
          --downloaddir=${varDirDependencies}/rpm                               \
          ${varListRpms}                                                        \
    && echo

# Gather Dependencies | binaries
WORKDIR ${varDirDependencies}/bin 
COPY --from=registry /bin/registry  ${varDirDependencies}/bin/registry
RUN set -ex                                                                     \
     && curl -L ${varUrlOpenshift}                                              \
          | tar xzvf - --directory ${varDirDependencies}/bin kubectl oc         \
     && curl -L ${varUrlOsInst}                                                 \
          | tar xzvf - --directory ${varDirDependencies}/bin openshift-install  \
     && curl -L ${varUrlHelm}                                                   \
          | tar xzvf - --directory /tmp linux-amd64/helm                        \
     && mv /tmp/linux-amd64/helm   ${varDirDependencies}/bin/                   \
     && curl -L ${varUrlTerraform}                                              \
          | bsdtar -xvf- -C ${varDirDependencies}/bin                           \
     && curl -L ${varUrlJq}                                                     \
             -o ${varDirDependencies}/bin/jq                                    \
    && echo

# Gather Dependencies | Pip
WORKDIR ${varDirDependencies}/pip
RUN set -ex                                                                     \
     && pip3 download                                                           \
            -d ${varDirDependencies}/pip                                        \
          ${varListPip}                                                         \
    && echo

# Gather Dependencies | gcloud
#WORKDIR ${varDirDependencies}/gcloud
#RUN set -ex                                                                     \
#     && curl -L ${varUrlGcloud}                                                 \
#          --output /tmp/install-gcloud.sh                                       \
#     && /bin/bash -x /tmp/install-gcloud.sh --disable-prompts                   \
#          --install-dir=${varDirDependencies}/gcloud                            \
#    && echo

# Create Unified Tarball
WORKDIR /tmp/
RUN set -ex                                                                     \
     && tar -czvf /root/dependencies.tar.gz dependencies                        \
     && du -sh /root/dependencies.tar.gz                                        \
    && echo

# Cleanup container
RUN yum clean all                                                               \
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
