FROM ubuntu:22.04

ENV DPDK_VERSION=v22.11.3 \
    ONNX_VERSION=1.16.3

RUN dpkg --add-architecture i386

RUN apt-get update

RUN apt-get install -yy --no-install-recommends \
        software-properties-common \
	dirmngr \
	gnupg-agent

RUN sed -e 's/^deb http/deb [arch=amd64,i386] http/g' /etc/apt/sources.list -i

RUN apt-get update --fix-missing

RUN apt-get install -yy \
  libcli-dev \
  libconfig-dev:i386 \
  libcunit1-dev:i386 \
  libpcap-dev:i386 \
  libpng-dev:i386 \
  libssl-dev:i386 \
  libc6-dev:i386 \
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
  wget \
  unzip

Run gcc --version

RUN apt-get install -yy \
  gcc-11-i686-linux-gnu \
  g++-11-i686-linux-gnu

RUN find / -name 'i686-linux-gnu-g++-11*' 2>/dev/null

Run which i686-linux-gnu-gcc-11
RUN which i686-linux-gnu-g++-11

RUN python3 -m pip install cmake && \
  which cmake && \
  cmake --version

RUN pip3 install \
  meson \
  pyelftools \
  flatbuffers

RUN cd $HOME && \
    wget https://github.com/microsoft/onnxruntime/archive/refs/tags/v${ONNX_VERSION}.zip && \
    unzip v${ONNX_VERSION}.zip && \
    cd onnxruntime-${ONNX_VERSION} && \
    ./build.sh --config RelWithDebInfo --build_shared_lib --parallel --compile_no_warning_as_error --skip_submodule_sync --allow_running_as_root --skip_tests --cmake_extra_defines CMAKE_C_FLAGS="-m32 -msse4.1" CMAKE_CXX_FLAGS="-m32 -msse4.1" CMAKE_C_COMPILER=/usr/bin/i686-linux-gnu-gcc-11 CMAKE_CXX_COMPILER=/usr/bin/i686-linux-gnu-g++-11 CMAKE_FIND_ROOT_PATH=/ CMAKE_SYSTEM_NAME=Linux CMAKE_SYSTEM_PROCESSOR=i386 CMAKE_CROSSCOMPILING=TRUE && \
    tools/ci_build/github/linux/copy_strip_binary.sh -r build/Linux/ -a onnxruntime -l libonnxruntime.so.1.16.3 -c RelWithDebInfo -s . -t 2ac381c55397dffff327cc6efecf6f95a70f90a1 && \
    cp build/Linux/onnxruntime/include/* /usr/local/include/ && \
    cp build/Linux/onnxruntime/lib/* /usr/local/lib/ && \
    ldconfig && \
    cd $HOME && \
    rm -r ./onnxruntime-${ONNX_VERSION}

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
