FROM centos:latest
################################################################################
# ContainerOne | Cloud Orchestration Tools - Point of Origin
LABEL Vendor="containercraft.io" \
      maintainer="ContainerCraft.io" \
      License=GPLv3 

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
          ${rpmList}
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
RUN tar -czvf artifacts.tar.gz artifacts

RUN yum clean all \
    && rm -rf \
        /tmp/builder/artifacts \
        /var/cache \
        /var/log/dnf* \
        /var/log/yum.* \
   ; echo

ENV versOCP="${varVerOpenshift}"
ENTRYPOINT ["/bin/bash"]
