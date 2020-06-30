#FROM registry.fedoraproject.org/fedora as rpm
FROM centos:latest as rpm
FROM registry:2 as registry
FROM registry.access.redhat.com/ubi8/ubi:latest as ubi8
FROM registry.access.redhat.com/ubi8/ubi:latest as ubi
FROM ubi
#################################################################################
# OCP Version set in src/ocp
WORKDIR /root
ARG \
  varVerOpenshift="${varVerOpenshift}" \
  varVerTerraform="${varVerTerraform}" \
  varVerHelm="${varVerHelm}" \
  varVerJq="${varVerJq}" \
  urlOC="https://mirror.openshift.com/pub/openshift-v4/x86_64/clients/ocp/${varVerOpenshift}/openshift-client-linux.tar.gz" \
  urlOCINST="https://mirror.openshift.com/pub/openshift-v4/x86_64/clients/ocp/${varVerOpenshift}/openshift-install-linux.tar.gz" \
  varUrlGcloud="https://sdk.cloud.google.com" \
  varUrlHelm="https://get.helm.sh/helm-v${varVerHelm}-linux-amd64.tar.gz" \
  urlRelease="https://mirror.openshift.com/pub/openshift-v4/clients/ocp/candidate/release.txt" \
  varUrlTerraform="https://releases.hashicorp.com/terraform/${varVerTerraform}/terraform_${varVerTerraform}_linux_amd64.zip" \
  varUrlOsInst="https://mirror.openshift.com/pub/openshift-v4/x86_64/clients/ocp/${varVerOpenshift}/openshift-install-linux.tar.gz" \
  varUrlOpenshift="https://mirror.openshift.com/pub/openshift-v4/x86_64/clients/ocp/${varVerOpenshift}/openshift-client-linux.tar.gz" \
  varUrlJq="https://github.com/stedolan/jq/releases/download/jq-${varVerJq}/jq-linux64" \
  varDirDependencies="/root/dependencies" \
  listManifest="/var/lib/trno/release.list"

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
             podman-remote \
             openssh-server \
             libvarlink-util \
             openssh-clients \
             bash-completion \
             "
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

# Create Artifact Directories
RUN set -ex                                                                     \
     && mkdir -p /root/versions                                                 \
     && mkdir -p ${varDirDependencies}/bin                                      \
     && mkdir -p ${varDirDependencies}/pip                                      \
     && mkdir -p ${varDirDependencies}/config                                   \
     && mkdir -p ${varDirDependencies}/gcloud                                   \
     && mkdir -p /var/lib/trno/                                                 \
    && echo

#################################################################################

# Generate OCP Release Images List Manifest

RUN set -ex \
     && curl -sL ${urlRelease}                                                  \
      | awk -F'[ ]' '/Pull From:/{print $3}'                                    \
      | tee -a ${listManifest}                                                  \
     && curl -sL ${urlRelease}                                                  \
      | grep -v 'Pull From'                                                     \
      | awk '/quay.io\/openshift-release/{print $2}'                            \
      | tee -a ${listManifest}                                                  \
    && echo

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
     && rm  -rf   /root/dependencies                                            \
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
ENV varVerOpenshift="${varVerOpenshift}"
LABEL Vendor="containercraft.io"                                                \
      BuildDate="${varRunDate}"                                                 \
      OpenShift="${varVerOpenshift}"                                            \
      maintainer="ContainerCraft.io"                                            \
      License=GPLv3 
