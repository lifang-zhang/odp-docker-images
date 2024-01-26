FROM ubuntu:18.04

ENV DPDK_VERSION=v22.11.3 \
    ONNX_VERSION=1.16.3

RUN apt-get update

RUN apt-get install -yy --no-install-recommends \
        software-properties-common \
	dirmngr \
	gnupg-agent

RUN sed -e 's/^deb http/deb [arch=amd64,i386] http/g' /etc/apt/sources.list -i

# Required for gcc-9
RUN add-apt-repository ppa:ubuntu-toolchain-r/test

RUN dpkg --add-architecture i386

RUN apt-get update --fix-missing

RUN apt-get install -yy \
  libcli-dev:i386 \
  libconfig-dev:i386 \
  libcunit1-dev:i386 \
  libpcap-dev:i386 \
  libpng-dev:i386 \
  libc6-dev:i386 \
  libssl-dev:i386 \
  autoconf \
  automake \
  ccache \
  clang \
  gcc-9 \
  git \
  iproute2 \
  kmod \
  libtool \
  net-tools \
  ninja-build \
  python3-pip \
  sudo \
  wget \
  unzip \
  gcc-i686-linux-gnu \
  g++-i686-linux-gnu \
  aptitude

# Building onnxruntime requires python3
RUN apt-get install -yy libffi-dev libssl-dev zlib1g-dev liblzma-dev libbz2-dev libreadline-dev libsqlite3-dev

RUN wget https://www.python.org/ftp/python/3.7.9/Python-3.7.9.tgz
RUN tar xzf Python-3.7.9.tgz
RUN cd Python-3.7.9 && ./configure --enable-optimizations && make && make install

RUN rm -rf Python-3.7.9.tgz Python-3.7.9

RUN python3.7 --version

RUN python3 -m pip install cmake

RUN pip3 install \
  meson \
  pyelftools \
  flatbuffers

RUN cd $HOME && \
    wget https://github.com/microsoft/onnxruntime/archive/refs/tags/v${ONNX_VERSION}.zip && \
    unzip v${ONNX_VERSION}.zip && \
    cd onnxruntime-${ONNX_VERSION} && \
    ./build.sh --config RelWithDebInfo --build_shared_lib --parallel --compile_no_warning_as_error --skip_submodule_sync --allow_running_as_root --skip_tests --cmake_extra_defines CMAKE_C_FLAGS="-m32 -msse4.1" CMAKE_CXX_FLAGS="-m32 -msse4.1" CMAKE_C_COMPILER=/usr/bin/i686-linux-gnu-gcc CMAKE_CXX_COMPILER=/usr/bin/i686-linux-gnu-g++ CMAKE_FIND_ROOT_PATH=/ CMAKE_SYSTEM_NAME=Linux CMAKE_SYSTEM_PROCESSOR=i386 CMAKE_CROSSCOMPILING=TRUE && \
    tools/ci_build/github/linux/copy_strip_binary.sh -r build/Linux/ -a onnxruntime -l libonnxruntime.so.1.16.3 -c RelWithDebInfo -s . -t 2ac381c55397dffff327cc6efecf6f95a70f90a1 && \
    cp build/Linux/onnxruntime/include/* /usr/local/include/ && \
    cp build/Linux/onnxruntime/lib/* /usr/local/lib/ && \
    ldconfig && \
    cd $HOME && \
    rm -r ./onnxruntime-${ONNX_VERSION}

# gcc-i686-linux-gnu g++-i686-linux-gnu have conflicts with gcc-multilib
RUN apt-get purge -yy gcc-i686-linux-gnu g++-i686-linux-gnu

RUN apt-get autoremove -yy && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*

RUN apt-get update

RUN aptitude -yy install \
  gcc-multilib \
  g++-multilib

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
