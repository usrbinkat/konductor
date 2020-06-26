#FROM registry.fedoraproject.org/fedora as rpm
FROM centos:latest as rpm
FROM registry:2 as registry
FROM registry.access.redhat.com/ubi8/ubi:latest as ubi8
FROM registry.access.redhat.com/ubi8/ubi:latest as ubi
FROM ubi
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
             which \
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
WORKDIR /root
ARG varDirDependencies="/root/dependencies"

# Create Artifact Directories
RUN set -ex                                                                     \
     && mkdir -p ${varDirDependencies}/bin                                      \
     && mkdir -p ${varDirDependencies}/pip                                      \
     && mkdir -p ${varDirDependencies}/config                                   \
     && mkdir -p ${varDirDependencies}/gcloud                                   \
    && echo

#################################################################################

# Gather Dependencies | rpm
COPY --from=rpm /etc/yum.repos.d /etc/yum.repos.d
COPY --from=rpm /etc/pki /etc/pki
RUN set -ex \
     && mkdir /tmp/rpm                                                          \
     && yum update  -y --setopt=tsflags=nodocs                                  \
     && yum install -y                                                          \
              --downloadonly                                                    \
              --downloaddir=/tmp/rpm/                                           \
            ${varListRpms}                                                      \
     && cp -r /tmp/rpm /root/dependencies/rpm                                   \
     && ls -l ${varDirDependencies}/rpm/*.rpm                                   \
    && echo

# Gather Dependencies | binaries
COPY --from=registry /bin/registry  ${varDirDependencies}/bin/registry
RUN set -ex                                                                     \
     && yum install -y --setopt=tsflags=nodocs                                  \
                bsdtar                                                          \
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
RUN set -ex                                                                     \
     && yum install -y --setopt=tsflags=nodocs                                  \
                python3-pip                                                     \
     && pip3 download                                                           \
                -d ${varDirDependencies}/pip                                    \
          ${varListPip}                                                         \
    && echo

# Gather Dependencies | gcloud
#RUN set -ex                                                                     \
#     && yum install -y --setopt=tsflags=nodocs                                  \
#                which                                                           \
#     && curl -L ${varUrlGcloud}                                                 \
#          --output /tmp/install-gcloud.sh                                       \
#     && /bin/bash -x /tmp/install-gcloud.sh --disable-prompts                   \
#          --install-dir=${varDirDependencies}/gcloud                            \
#    && echo

COPY --from=ubi8 /etc/pki         ${varDirDependencies}/config/etc-pki
COPY --from=ubi8 /etc/yum.repos.d ${varDirDependencies}/config/etc-yum.repos.d

# Create Unified Tarball
RUN set -ex                                                                     \
     && tar -czvf /root/dependencies.tar.gz dependencies                        \
     && du  -sh   /root/dependencies.tar.gz                                     \
    && echo

# Cleanup container
RUN yum clean all                                                               \
    && rm -rf                                                                   \
        /root/dependencies                                                      \
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
