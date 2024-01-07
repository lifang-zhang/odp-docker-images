FROM centos:7

ENV DPDK_VERSION=v19.11.14 \
    RTE_ARCH=x86_64 \
    RTE_TARGET=x86_64-native-linuxapp-gcc \
    CUNIT_VERSION=2.1-3 \
    ONNX_VERSION=1.16.2 \
    LIBCONFIG_VERSION=v1.5

RUN yum update -y

RUN yum install -y \
	https://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm

RUN yum install -y \
	autoconf \
	automake \
	bzip2 \
	clang \
	curl \
	flex \
	gcc \
	gcc-c++ \
	git-core \
	kmod \
	libatomic \
	libcli-devel \
	libpcap-devel \
	libtool \
	make \
	numactl-devel \
	net-tools \
	openssl-devel \
	pkgconfig \
	sudo \
    wget

RUN cd $HOME && \
    wget https://download.sourceforge.net/libpng/libpng-1.6.37.tar.gz && \
    tar xvzf libpng-1.6.37.tar.gz && \
    cd libpng-1.6.37 && \
    ./configure && \
    make && \
    make install && \
    cd $HOME && \
    rm -r ./libpng-1.6.37

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
    curl -sSOL http://sourceforge.net/projects/cunit/files/CUnit/${CUNIT_VERSION}/CUnit-${CUNIT_VERSION}.tar.bz2 && \
    tar -jxf *.bz2 && \
    cd CUnit* && \
    ./bootstrap && \
    ./configure && \
    make && \
    make install && \
    cd $HOME && \
    rm -r ./CUnit*

RUN cd $HOME && \
    git clone https://github.com/hyperrealm/libconfig.git --branch ${LIBCONFIG_VERSION} --depth 1 ./libconfig && \
    cd libconfig && \
    autoreconf -i && \
    ./configure --prefix=/usr/ && \
    make && \
    make install && \
    ldconfig /usr/lib/ && \
    cd $HOME && \
    rm -r ./libconfig

RUN cd $HOME && \
    git clone http://dpdk.org/git/dpdk-stable --branch ${DPDK_VERSION} --depth 1 ./dpdk && \
    cd dpdk && \
    make config T=x86_64-native-linuxapp-gcc O=x86_64-native-linuxapp-gcc && \
    cd x86_64-native-linuxapp-gcc/ && \
    sed -ri 's,(CONFIG_RTE_MACHINE=).*,\1"default",' .config && \
    sed -ri 's,(CONFIG_RTE_LIBRTE_PMD_PCAP=).*,\1y,' .config && \
    sed -ri 's,(CONFIG_RTE_LIBRTE_PMD_OPENSSL=).*,\1y,' .config && \
    sed -ri 's,(CONFIG_RTE_EAL_IGB_UIO=).*,\1n,' .config && \
    sed -ri 's,(CONFIG_RTE_KNI_KMOD=).*,\1n,' .config && \
    cd .. && \
    make -j $(nproc) install T=x86_64-native-linuxapp-gcc DESTDIR=/usr EXTRA_CFLAGS="-fPIC" && \
    cd $HOME && \
    rm -r ./dpdk
