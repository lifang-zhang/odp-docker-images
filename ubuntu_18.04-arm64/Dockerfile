FROM ubuntu:18.04

ENV DPDK_VERSION=v22.11.3 \
    AARCH64_CRYPTOLIB_TAG=2445f0e \
    ONNX_VERSION=1.16.3

RUN apt-get update

RUN apt-get install -yy --no-install-recommends \
        software-properties-common \
	dirmngr \
	gnupg-agent

RUN sed -e 's/^deb http/deb [arch=amd64] http/g' /etc/apt/sources.list -i

RUN dpkg --add-architecture arm64 && \
 apt-add-repository -y "deb [arch=arm64] http://ports.ubuntu.com bionic main universe" && \
 apt-add-repository -y "deb [arch=arm64] http://ports.ubuntu.com bionic-updates main universe"

RUN apt-get update --fix-missing

RUN apt-get install -yy \
  crossbuild-essential-arm64 \
  libcli-dev:arm64 \
  libconfig-dev:arm64 \
  libcunit1-dev:arm64 \
  libpcap-dev:arm64 \
  libpng-dev:arm64 \
  libssl-dev:arm64 \
  libstdc++-7-dev:arm64 \
  pkg-config-aarch64-linux-gnu \
  autoconf \
  automake \
  ccache \
  clang \
  gcc \
  git \
  iproute2 \
  kmod \
  libtool \
  net-tools \
  ninja-build \
  python3-pip \
  sudo \
  wget

RUN pip3 install \
  meson \
  pyelftools

RUN cd $HOME && \
    wget https://github.com/microsoft/onnxruntime/releases/download/v${ONNX_VERSION}/onnxruntime-linux-aarch64-${ONNX_VERSION}.tgz && \
    tar xvf onnxruntime-linux-aarch64-${ONNX_VERSION}.tgz && \
    cd onnxruntime-linux-aarch64-${ONNX_VERSION} && \
    cp include/* /usr/local/include/ && \
    cp lib/* /usr/local/lib/ && \
    ldconfig && \
    cd $HOME && \
    rm -r ./onnxruntime-linux-aarch64-${ONNX_VERSION}


RUN cd $HOME && \
    git clone http://dpdk.org/git/dpdk-stable --branch ${DPDK_VERSION} --depth 1 ./dpdk && \
    cd dpdk && \
    meson setup build --cross-file config/arm/arm64_armv8_linux_gcc && \
    cd build && \
    meson configure -Dlibdir=lib/aarch64-linux-gnu && \
    meson configure -Dprefix=/usr && \
    meson configure -Ddisable_apps=* && \
    meson configure -Dtests=false && \
    meson configure -Denable_drivers=crypto/*,dma/*,net/pcap && \
    ninja && \
    ninja install && \
    cd $HOME && \
    rm -r ./dpdk

RUN cd $HOME && \
    git clone https://github.com/ARM-software/AArch64cryptolib --depth 1 ./aarch64cryptolib && \
    cd aarch64cryptolib && \
    git checkout ${AARCH64_CRYPTOLIB_TAG} && \
    make OPT=big CROSS=aarch64-linux-gnu- && \
    cd -
