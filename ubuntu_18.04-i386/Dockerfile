FROM ubuntu:18.04

ENV DPDK_VERSION=v22.11.3 \
    ONNX_VERSION=1.16.2

RUN apt-get update

RUN apt-get install -yy --no-install-recommends \
        software-properties-common \
	dirmngr \
	gnupg-agent

RUN sed -e 's/^deb http/deb [arch=amd64,i386] http/g' /etc/apt/sources.list -i

RUN dpkg --add-architecture i386

RUN apt-get update --fix-missing

RUN apt-get install -yy \
  libcli-dev:i386 \
  libconfig-dev:i386 \
  libcunit1-dev:i386 \
  libpcap-dev:i386 \
  libpng-dev:i386 \
  libssl-dev:i386 \
  lib32stdc++-7-dev \
  autoconf \
  automake \
  ccache \
  clang \
  gcc \
  gcc-multilib \
  g++-multilib \
  git \
  iproute2 \
  kmod \
  libtool \
  net-tools \
  ninja-build \
  python3-pip \
  sudo \
  wget \
  unzip

RUN pip3 install \
  meson \
  pyelftools

RUN cd $HOME && \
    wget https://github.com/microsoft/onnxruntime/archive/refs/tags/v${ONNX_VERSION}.zip && \
    unzip v${ONNX_VERSION}.zip && \
    cd onnxruntime-${ONNX_VERSION} && \
    ./build.sh --config RelWithDebInfo --build_shared_lib --parallel --compile_no_warning_as_error --skip_submodule_sync --cmake_extra_defines CMAKE_C_FLAGS=-m32 CMAKE_CXX_FLAGS=-m32 && \
    ls && \
    rm -r ./onnxruntime-linux-x64-${ONNX_VERSION}

RUN cd $HOME && \
    git clone http://dpdk.org/git/dpdk-stable --branch ${DPDK_VERSION} --depth 1 ./dpdk && \
    cd dpdk && \
    PKG_CONFIG_LIBDIR=/usr/lib/i386-linux-gnu/pkgconfig \
    meson setup -Dc_args='-m32' -Dc_link_args='-m32' build && \
    cd build && \
    meson configure -Dlibdir=lib/i386-linux-gnu && \
    meson configure -Dmachine=default && \
    meson configure -Dprefix=/usr && \
    meson configure -Ddisable_apps=* && \
    meson configure -Dtests=false && \
    meson configure -Denable_drivers=crypto/*,dma/*,net/pcap && \
    ninja && \
    ninja install && \
    cd $HOME && \
    rm -r ./dpdk
