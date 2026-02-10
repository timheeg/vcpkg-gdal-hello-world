ARG BASE_REPO="registry.redhat.io"
ARG BASE_IMAGE="ubi9/ubi"
ARG BASE_TAG="9.7"

################################################
FROM $BASE_REPO/$BASE_IMAGE:$BASE_TAG AS base

# Provide your own RHEL subscription
RUN --mount=type=secret,id=RHEL_ORG \
  --mount=type=secret,id=RHEL_ACTIVATION_KEY \
  subscription-manager register \
  --org=$(cat /run/secrets/RHEL_ORG) \
  --activationkey=$(cat /run/secrets/RHEL_ACTIVATION_KEY)

RUN dnf update -y \
  && dnf -y install \
  unzip \
  && dnf clean all \
  && rm -rf /var/cache/yum

################################################
FROM base AS setup-tools

# Install tools
#
# autoconf for libpq
# autoconf-archive for libspatialite
# automake for freexl
# clang-tools-extra for clang-format
# gcc-toolset-13 compiler
# java-11-openjdk for gdal java bindings
# java-11-openjdk-devel for gdal java bindings
# kernel-headers for openssl build
# libtool for freexl
# perl-FindBin for openssl
# perl-IPC-Cmd for openssl
# perl-Time-Piece for openssl
# vim-enhanced for color vi support in git
# zip/unzip for vcpkg
#
RUN dnf update -y \
  && dnf -y install \
  autoconf \
  autoconf-archive \
  automake \
  clang-tools-extra \
  gcc-toolset-13 \
  git \
  java-11-openjdk \
  java-11-openjdk-devel \
  libtool \
  kernel-headers \
  perl-FindBin \
  perl-IPC-Cmd \
  perl-Time-Piece \
  sudo \
  unzip \
  vim-enhanced \
  zip \
  && dnf clean all \
  && rm -rf /var/cache/yum

# Install tools from centos 9
#
# bison for libpq
# flex for libpq
# kernel-devel for openssl build
# swig 4+ for java bindings
#
COPY centos9.repo /etc/yum.repos.d/
RUN dnf update -y \
  && dnf install -y --enablerepo="centos9*" \
  bison \
  flex \
  kernel-devel \
  swig \
  && dnf clean all \
  && rm -rf /var/cache/yum

FROM setup-tools AS setup-user

# 🧑‍💻 Create the non-root user
ARG USERNAME=dev
ARG USER_UID=1000
ARG USER_GID=$USER_UID

# Create the user and group
RUN groupadd --gid $USER_GID $USERNAME \
  && useradd -s /bin/bash --uid $USER_UID --gid $USER_GID -m $USERNAME \
  && echo $USERNAME ALL=\(ALL\) NOPASSWD: ALL > /etc/sudoers.d/$USER_UID \
  && chmod 0440 /etc/sudoers.d/$USER_UID

################################################
FROM base AS cmake
WORKDIR /tmp
RUN curl --fail -LO https://github.com/Kitware/CMake/releases/download/v4.2.3/cmake-4.2.3-linux-x86_64.tar.gz && \
  tar -xvf cmake-4.2.3-linux-x86_64.tar.gz && \
  rm cmake-4.2.3-linux-x86_64.tar.gz

################################################
FROM base AS ninja
WORKDIR /tmp/ninja-1.13.2
RUN curl --fail -LO https://github.com/ninja-build/ninja/releases/download/v1.13.2/ninja-linux.zip && \
  unzip ninja-linux.zip && \
  rm ninja-linux.zip

################################################
FROM base AS patchelf
WORKDIR /tmp/patchelf-0.18.0
RUN curl --fail -LO https://github.com/NixOS/patchelf/releases/download/0.18.0/patchelf-0.18.0-x86_64.tar.gz && \
  tar -xvf patchelf-0.18.0-x86_64.tar.gz && \
  rm patchelf-0.18.0-x86_64.tar.gz

################################################
FROM base AS ant
WORKDIR /tmp
RUN curl --fail -LO https://downloads.apache.org/ant/binaries/apache-ant-1.10.15-bin.tar.gz.sha512 && \
  curl --fail -LO https://downloads.apache.org/ant/binaries/apache-ant-1.10.15-bin.tar.gz && \
  echo "$(cat apache-ant-1.10.15-bin.tar.gz.sha512)  apache-ant-1.10.15-bin.tar.gz" | sha512sum -c && \
  rm apache-ant-1.10.15-bin.tar.gz.sha512 && \
  tar -xvf apache-ant-1.10.15-bin.tar.gz && \
  rm apache-ant-1.10.15-bin.tar.gz

################################################
FROM setup-user AS vcpkg
# 🧑‍💻 Change to non-root user
USER $USERNAME
WORKDIR /vcpkg
RUN git clone https://github.com/microsoft/vcpkg.git \
  && ./vcpkg/bootstrap-vcpkg.sh -disableMetrics

################################################
FROM setup-user AS build-env

WORKDIR /opt/cmake-4.2.3-linux-x86_64
COPY --from=cmake /tmp/cmake-4.2.3-linux-x86_64 .

WORKDIR /opt/ninja-1.13.2
COPY --from=ninja /tmp/ninja-1.13.2 .

WORKDIR /opt/patchelf-0.18.0
COPY --from=patchelf /tmp/patchelf-0.18.0 .

WORKDIR /opt/apache-ant-1.10.15
COPY --from=ant /tmp/apache-ant-1.10.15 .

WORKDIR /
COPY --from=vcpkg --chown=1000:1000 /vcpkg .

# 🌊 Setup the container environment

# Add gcc to path
ENV PATH="/opt/rh/gcc-toolset-13/root/usr/bin:$PATH"

# Add cmake to path
ENV CMAKE_HOME="/opt/cmake-4.2.3-linux-x86_64"
ENV PATH="$CMAKE_HOME/bin:$PATH"

# Add ninja to path
ENV PATH="/opt/ninja-1.13.2:$PATH"

# Add patchelf to path
ENV PATH="/opt/patchelf-0.18.0/bin:$PATH"

# Add ant to path
ENV ANT_HOME="/opt/apache-ant-1.10.15"
ENV PATH="$ANT_HOME/bin:$PATH"

# Define VCPKG_ROOT and add to path
ENV VCPKG_ROOT="/vcpkg"
ENV PATH="$VCPKG_ROOT:$PATH"

# 🧑‍💻 Change to non-root user
USER $USERNAME

# Install bashrc configuration
WORKDIR /home/$USERNAME/.bashrc.d
COPY --chown=$USER_UID:$USER_GID git.bashrc .
COPY --chown=$USER_UID:$USER_GID user.bashrc .

# Install inputrc with prefix history search
WORKDIR /home/$USERNAME
COPY --chown=$USER_UID:$USER_GID .inputrc .

CMD [ "bash" ]
