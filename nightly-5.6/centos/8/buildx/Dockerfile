FROM centos:8 AS base
LABEL maintainer="Swift Infrastructure <swift-infrastructure@forums.swift.org>"
LABEL description="Docker Container for the Swift programming language"

RUN yum install -y https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm

RUN yum install --enablerepo=powertools -y \
  binutils \
  gcc \
  git \
  glibc-static \
  libbsd-devel \
  libcurl-devel \
  libedit \
  libedit-devel \
  libicu-devel \
  libstdc++-static \
  libxml2-devel \
  pkg-config \
  python2 \
  sqlite \
  zlib-devel

RUN ln -s /usr/bin/python2 /usr/bin/python

# Everything up to here should cache nicely between Swift versions, assuming dev dependencies change little

# gpg --keyid-format LONG -k FAF6989E1BC16FEA
# pub   rsa4096/FAF6989E1BC16FEA 2019-11-07 [SC] [expires: 2021-11-06]
#       8A7495662C3CD4AE18D95637FAF6989E1BC16FEA
# uid                 [ unknown] Swift Automatic Signing Key #3 <swift-infrastructure@swift.org>
ARG SWIFT_SIGNING_KEY=8A7495662C3CD4AE18D95637FAF6989E1BC16FEA
ARG SWIFT_PLATFORM=centos
ARG OS_MAJOR_VER=8
ARG SWIFT_WEBROOT=https://download.swift.org/swift-5.6-branch

# This is a small trick to enable if/else for arm64 and amd64.
# Because of https://bugs.swift.org/browse/SR-14872 we need adjust tar options.
FROM base AS base-amd64
ARG OS_ARCH_SUFFIX=
ARG ADDITIONAL_TAR_OPTIONS="--strip-components=1"

FROM base AS base-arm64
ARG OS_ARCH_SUFFIX=-aarch64
ARG ADDITIONAL_TAR_OPTIONS=

FROM base-$TARGETARCH AS final

ARG OS_VER=$SWIFT_PLATFORM$OS_MAJOR_VER$OS_ARCH_SUFFIX
ARG PLATFORM_WEBROOT="$SWIFT_WEBROOT/$SWIFT_PLATFORM$OS_MAJOR_VER$OS_ARCH_SUFFIX"

RUN echo "${PLATFORM_WEBROOT}/latest-build.yml"

RUN set -e; \
    # - Latest Toolchain info
    export $(curl -s ${PLATFORM_WEBROOT}/latest-build.yml | grep 'download:' | sed 's/:[^:\/\/]/=/g')  \
    && export $(curl -s ${PLATFORM_WEBROOT}/latest-build.yml | grep 'download_signature:' | sed 's/:[^:\/\/]/=/g')  \
    && export DOWNLOAD_DIR=$(echo $download | sed "s/-${OS_VER}.tar.gz//g") \
    && echo $DOWNLOAD_DIR > .swift_tag \
    # - Download the GPG keys, Swift toolchain, and toolchain signature, and verify.
    && export GNUPGHOME="$(mktemp -d)" \
    && curl -fsSL ${PLATFORM_WEBROOT}/${DOWNLOAD_DIR}/${download} -o latest_toolchain.tar.gz \
    ${PLATFORM_WEBROOT}/${DOWNLOAD_DIR}/${download_signature} -o latest_toolchain.tar.gz.sig \
    && curl -fSsL https://swift.org/keys/all-keys.asc | gpg --import -  \
    && gpg --batch --verify latest_toolchain.tar.gz.sig latest_toolchain.tar.gz \
    # - Unpack the toolchain, set libs permissions, and clean up.
    && tar -xzf latest_toolchain.tar.gz --directory / ${ADDITIONAL_TAR_OPTIONS} \
    && chmod -R o+r /usr/lib/swift \
    && rm -rf "$GNUPGHOME" latest_toolchain.tar.gz.sig latest_toolchain.tar.gz

# Print Installed Swift Version
RUN swift --version

RUN echo '[ ! -z "$TERM" -a -r /etc/motd ] && cat /etc/motd' \
    >> /etc/bashrc; \
    echo -e " ################################################################\n" \
    "#\t\t\t\t\t\t\t\t#\n" \
    "# Swift Nightly Docker Image\t\t\t\t\t#\n" \
    "# Tag: $(cat .swift_tag)\t\t\t#\n" \
    "#\t\t\t\t\t\t\t\t#\n"  \
    "################################################################\n" > /etc/motd
