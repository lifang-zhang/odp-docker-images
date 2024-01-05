FROM ubuntu:18.04

ENV DPDK_VERSION=v22.11.3 \
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
  curl \
  gcc \
  gcc-7 \
  gcc-8 \
  git \
  iproute2 \
  kmod \
  libcli-dev \
  libconfig-dev \
  libcunit1-dev \
  libpcap-dev \
  libpng-dev \
  libssl-dev \
  libstdc++-7-dev \
  libstdc++-8-dev \
  libtool \
  make \
  net-tools \
  ninja-build \
  python3-pip \
  sudo \
  wget

RUN pip3 install \
  meson \
  numpy \
  pyelftools

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
