FROM rockylinux:8

ENV DPDK_VERSION=v22.11.3 \
    ONNX_VERSION=1.16.2

RUN dnf install -y 'dnf-command(config-manager)'
RUN dnf config-manager --set-enabled powertools

RUN yum update -y

RUN yum install -y \
	https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm

RUN yum install -y \
	autoconf \
	automake \
	clang \
	CUnit-devel \
	curl \
	diffutils \
	gcc \
	gcc-c++ \
	gcc-toolset-12-libatomic-devel \
	git-core \
	kmod \
	libatomic \
	libconfig-devel \
	libibverbs-devel \
	libpcap-devel \
	libpng-devel \
	libtool \
	make \
	meson \
	numactl-devel \
	net-tools \
	ninja-build \
	openssl-devel \
	pkgconfig \
	python3-pip \
	procps \
	sudo \
	wget

RUN pip3 install \
	pyelftools

RUN cd $HOME && \
    wget https://github.com/microsoft/onnxruntime/releases/download/v${ONNX_VERSION}/onnxruntime-linux-x64-${ONNX_VERSION}.tgz && \
    tar xvf onnxruntime-linux-x64-${ONNX_VERSION}.tgz && \
    cd onnxruntime-linux-x64-${ONNX_VERSION} && \
    cp include/* /usr/include/ && \
    cp lib/* /usr/lib/ && \
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
