#FROM registry.fedoraproject.org/fedora as rpm
FROM centos:latest as rpm
FROM registry:2 as registry
FROM quay.io/containercraft/openshift-install-mod:latest as oops
FROM registry.access.redhat.com/ubi8/ubi-minimal:latest as ubi8
FROM registry.access.redhat.com/ubi8/ubi-minimal:latest as ubi
FROM ubi
#################################################################################
# ContainerOne | Cloud Orchestration Tools - Point of Origin
LABEL Vendor="containercraft.io"                                                \
      BuildDate="${varRunDate}"                                                 \
      OpenShift="${varVerOpenshift}"                                            \
      maintainer="ContainerCraft.io"                                            \
      License=GPLv3 
WORKDIR /root
#################################################################################
# OCP Version set in src/ocp
ARG varVerOpenshift="${varVerOpenshift}"
ARG varVerTerraform="${varVerTerraform}"
ARG varVerHelm="${varVerHelm}"
ARG varVerJq="${varVerJq}"

ARG varUrlGcloud="https://sdk.cloud.google.com"
ARG varUrlHelm="https://get.helm.sh/helm-v${varVerHelm}-linux-amd64.tar.gz"
ARG varUrlJq="https://github.com/stedolan/jq/releases/download/jq-${varVerJq}/jq-linux64"
ARG urlRelease="https://mirror.openshift.com/pub/openshift-v4/clients/ocp/candidate/release.txt"
ARG urlOC="https://mirror.openshift.com/pub/openshift-v4/x86_64/clients/ocp/${varVerOpenshift}/openshift-client-linux.tar.gz"
ARG urlOCINST="https://mirror.openshift.com/pub/openshift-v4/x86_64/clients/ocp/${varVerOpenshift}/openshift-install-linux.tar.gz"
ARG varUrlTerraform="https://releases.hashicorp.com/terraform/${varVerTerraform}/terraform_${varVerTerraform}_linux_amd64.zip"
ARG varUrlOsInst="https://mirror.openshift.com/pub/openshift-v4/x86_64/clients/ocp/${varVerOpenshift}/openshift-install-linux.tar.gz"
ARG varUrlOpenshift="https://mirror.openshift.com/pub/openshift-v4/x86_64/clients/ocp/${varVerOpenshift}/openshift-client-linux.tar.gz"

ARG varDirDependencies="/root/build/dependencies"
ARG listManifest="/var/lib/koffer/release.list"

#################################################################################
# Package Lists
ARG varListRpms="\
             xz \
             git \
             vim \
             tar \
             file \
             sudo \
             tree \
             tmux \
             rsync \
             which \
             skopeo \
             bsdtar \
             iputils \
             ncurses \
             openssl \
             rpm-libs \
             procps-ng \
             net-tools \
             nmap-ncat \
             bind-utils \
             audit-libs \
             httpd-tools \
             python3-pip \
             openssh-server \
             libvarlink-util \
             openssh-clients \
             bash-completion \
             "
#            podman-remote \
#            iproute \
#            buildah \
#            fuse-overlayfs \
#            podman \
#            libsemanage \
#            iptables-libs \
#            libselinux \
ARG varListPip="\
             boto3 \
             awscli \
             ansible \
             "
COPY --from=rpm /etc/pki                      /etc/pki
COPY --from=rpm /etc/yum.repos.d              /etc/yum.repos.d
COPY --from=registry /bin/registry            ${varDirDependencies}/bin/registry
COPY --from=oops /root/openshift-install-mod  ${varDirDependencies}/bin/openshift-install-mod
#################################################################################
RUN set -ex                                                                     \
     && mkdir -p /root/versions                                                 \
     && mkdir -p /root/build/rpm                                                \
     && mkdir -p ${varDirDependencies}/bin                                      \
     && mkdir -p ${varDirDependencies}/pip                                      \
     && mkdir -p ${varDirDependencies}/config                                   \
     && mkdir -p ${varDirDependencies}/gcloud                                   \
     && mkdir -p /var/lib/koffer/                                               \
     && curl -sL ${urlRelease}                                                  \
      | awk -F'[ ]' '/Pull From:/{print $3}'                                    \
      | tee -a ${listManifest}                                                  \
      | sed 's/quay.io\///g'                                                    \
     && curl -sL ${urlRelease}                                                  \
      | grep -v 'Pull From'                                                     \
      | awk '/quay.io\/openshift-release/{print $2}'                            \
      | tee -a ${listManifest}                                                  \
      | sed 's/quay.io\///g'                                                    \
    && echo

# Gather Dependencies | rpm
COPY --from=rpm /etc/yum.repos.d /etc/yum.repos.d
COPY --from=rpm /etc/pki /etc/pki
RUN set -ex \
     && mkdir /tmp/rpm                                                          \
     && microdnf install dnf -y                                                 \
     && dnf update  -y --setopt=tsflags=nodocs                                  \
     && dnf install -y                                                          \
              --downloadonly                                                    \
              --downloaddir=/root/build/rpm/                                    \
            bsdtar tar                                                          \
     && dnf install -y                                                          \
              --downloadonly                                                    \
              --downloaddir=/tmp/rpm/                                           \
            ${varListRpms}                                                      \
     && cp -r /tmp/rpm ${varDirDependencies}/rpm                                \
     && ls -l ${varDirDependencies}/rpm/*.rpm                                   \
     && microdnf install -y                                                     \
                bsdtar tar                                                      \
     && curl -L ${varUrlOpenshift}                                              \
          | tar xzvf - --directory ${varDirDependencies}/bin kubectl oc         \
     && ${varDirDependencies}/bin/oc version | tee /root/versions/oc.release    \
     && curl -L ${varUrlOsInst}                                                 \
          | tar xzvf - --directory ${varDirDependencies}/bin openshift-install  \
     && curl -L ${varUrlHelm}                                                   \
          | tar xzvf - --directory /tmp linux-amd64/helm                        \
     && mv /tmp/linux-amd64/helm   ${varDirDependencies}/bin/                   \
     && curl -L ${varUrlTerraform}                                              \
          | bsdtar -xvf- -C ${varDirDependencies}/bin                           \
     && curl -L ${varUrlJq}                                                     \
             -o ${varDirDependencies}/bin/jq                                    \
     && dnf install -y --setopt=tsflags=nodocs                                  \
                python3-pip                                                     \
     && pip3 download                                                           \
                -d ${varDirDependencies}/pip                                    \
          ${varListPip}                                                         \
     && tar -czvf /root/build/dependencies.tar.gz -C /root/build dependencies   \
     && du  -sh   /root/build/dependencies.tar.gz                               \
     && rm  -rf   /root/build/dependencies                                      \
    && rm -rf                                                                   \
        /root/dependencies                                                      \
        /var/cache/*                                                            \
        /var/log/dnf*                                                           \
        /var/log/yum*                                                           \
   ; echo

COPY --from=ubi8 /etc/pki         ${varDirDependencies}/config/etc-pki
COPY --from=ubi8 /etc/yum.repos.d ${varDirDependencies}/config/etc-yum.repos.d

################################################################################
# Need to re-incorporate
# Gather Dependencies | gcloud
#RUN set -ex                                                                     \
#     && dnfinstall -y --setopt=tsflags=nodocs                                  \
#                which                                                           \
#     && curl -L ${varUrlGcloud}                                                 \
#          --output /tmp/install-gcloud.sh                                       \
#     && /bin/bash -x /tmp/install-gcloud.sh --disable-prompts                   \
#          --install-dir=${varDirDependencies}/gcloud                            \
#    && echo
