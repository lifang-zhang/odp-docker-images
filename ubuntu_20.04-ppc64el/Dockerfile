FROM ubuntu:20.04

ENV DPDK_VERSION=v22.11.3 \
    ONNX_VERSION=1.16.3

RUN apt-get update

RUN apt-get install -y --no-install-recommends \
  software-properties-common \
  dirmngr \
  gnupg-agent

RUN sed -e 's/^deb http/deb [arch=amd64] http/g' /etc/apt/sources.list -i

RUN dpkg --add-architecture ppc64el && \
  apt-add-repository -y "deb [arch=ppc64el] http://ports.ubuntu.com focal main universe" && \
  apt-add-repository -y "deb [arch=ppc64el] http://ports.ubuntu.com focal-updates main universe"

RUN apt-get update --fix-missing

# Install fails if done together with other packages
RUN apt-get install -o APT::Immediate-Configure=false -y libc6:ppc64el

RUN apt-get install -y \
  crossbuild-essential-ppc64el \
  libcli-dev:ppc64el \
  libconfig-dev:ppc64el \
  libcunit1-dev:ppc64el \
  libpcap-dev:ppc64el \
  libpng-dev:ppc64el \
  libssl-dev:ppc64el \
  libstdc++-10-dev:ppc64el \
  pkg-config-powerpc64le-linux-gnu \
  autoconf \
  automake \
  ccache \
  clang \
  gcc \
  gcc-10 \
  git \
  iproute2 \
  kmod \
  libtool \
  meson \
  net-tools \
  ninja-build \
  python3-pip \
  sudo \
  wget \
  unzip \
  g++-powerpc64le-linux-gnu

RUN python3 -m pip install cmake && \
  which cmake && \
  cmake --version

RUN pip3 install \
  numpy \
  onnx \
  pyelftools

RUN cd $HOME && \
    wget https://github.com/microsoft/onnxruntime/archive/refs/tags/v${ONNX_VERSION}.zip && \
    unzip v${ONNX_VERSION}.zip && \
    cd onnxruntime-${ONNX_VERSION} && \
    ./build.sh --config RelWithDebInfo --build_shared_lib --parallel --compile_no_warning_as_error --skip_submodule_sync --skip_tests --allow_running_as_root --cmake_extra_defines CMAKE_C_COMPILER=/usr/bin/powerpc64le-linux-gnu-gcc CMAKE_ASM_COMPILER=/usr/bin/powerpc64le-linux-gnu-gcc CMAKE_CXX_COMPILER=/usr/bin/powerpc64le-linux-gnu-g++ CMAKE_CXX_FLAGS=-Wno-psabi CMAKE_SYSTEM_NAME=Linux CMAKE_CROSSCOMPILING=TRUE CMAKE_SYSTEM_PROCESSOR=ppc64le && \
    tools/ci_build/github/linux/copy_strip_binary.sh -r build/Linux/ -a onnxruntime -l libonnxruntime.so.1.16.3 -c RelWithDebInfo -s . -t 2ac381c55397dffff327cc6efecf6f95a70f90a1 && \
    cp build/Linux/onnxruntime/include/* /usr/local/include/ && \
    cp build/Linux/onnxruntime/lib/* /usr/local/lib/ && \
    ldconfig && \
    cd $HOME && \
    rm -r ./onnxruntime-${ONNX_VERSION}

RUN cd $HOME && \
    git clone http://dpdk.org/git/dpdk-stable --branch ${DPDK_VERSION} --depth 1 ./dpdk && \
    cd dpdk && \
    meson setup build --cross-file config/ppc/ppc64le-power8-linux-gcc-ubuntu && \
    cd build && \
    meson configure -Dlibdir=lib/powerpc64le-linux-gnu && \
    meson configure -Dprefix=/usr && \
    meson configure -Ddisable_apps=* && \
    meson configure -Dtests=false && \
    meson configure -Denable_drivers=crypto/*,dma/*,net/pcap && \
    ninja && \
    ninja install && \
    cd $HOME && \
    rm -r ./dpdk
