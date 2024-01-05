FROM ubuntu:23.04

ENV DPDK_VERSION=v22.11.3 \
    XDP_VERSION=v1.2.3 \
    ONNX_VERSION=1.16.2

RUN apt-get update

RUN apt-get install -yy --no-install-recommends \
      software-properties-common \
      dirmngr \
      gnupg-agent

RUN apt-get update --fix-missing

RUN apt-get install -yy \
  autoconf \
  automake \
  ccache \
  clang \
  gcc \
  gcc-13 \
  gcc-multilib \
  git \
  iproute2 \
  kmod \
  libbpf-dev \
  libconfig-dev \
  libcunit1-dev \
  libpcap-dev \
  libpng-dev \
  libssl-dev \
  libstdc++-13-dev \
  libsystemd-dev \
  libtool \
  llvm-dev \
  meson \
  net-tools \
  ninja-build \
  python3-pip \
  python3-pyelftools \
  sudo \
  wget

RUN cd $HOME && \
    wget https://github.com/microsoft/onnxruntime/releases/download/v${ONNX_VERSION}/onnxruntime-linux-x64-${ONNX_VERSION}.tgz && \
    tar xvf onnxruntime-linux-x64-${ONNX_VERSION}.tgz && \
    cd onnxruntime-linux-x64-${ONNX_VERSION} && \
    cp include/* /usr/local/include/ && \
    cp lib/* /usr/local/lib/ && \
    ldconfig && \
    cd $HOME && \
    rm -r ./onnxruntime-linux-x64-${ONNX_VERSION}

RUN cd $HOME && \
    git clone http://dpdk.org/git/dpdk-stable --branch ${DPDK_VERSION} --depth 1 ./dpdk && \
    cd dpdk && \
    meson setup build && \
    cd build && \
    meson configure -Dmachine=default && \
    meson configure -Ddisable_apps=* && \
    meson configure -Dtests=false && \
    meson configure -Denable_drivers=crypto/*,dma/*,net/pcap && \
    ninja && \
    ninja install && \
    cd $HOME && \
    rm -r ./dpdk

RUN cd $HOME && \
    git clone --recurse-submodules https://github.com/xdp-project/xdp-tools.git --branch ${XDP_VERSION} --depth 1 ./xdp && \
    cd xdp && \
    ./configure && \
    make && \
    make install && \
    ldconfig /usr/local/lib/ && \
    cd $HOME && \
    rm -r ./xdp
