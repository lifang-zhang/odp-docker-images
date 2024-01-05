FROM ubuntu:20.04

ARG COVERITY_TOKEN

ENV COVERITY_PROJECT=ODP \
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
  gcc-10 \
  git \
  libcli-dev \
  libconfig-dev \
  libcunit1-dev \
  libpcap-dev \
  libpng-dev \
  libssl-dev \
  libtool \
  libstdc++-10-dev \
  make \
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

RUN curl -d "token=${COVERITY_TOKEN}&project=${COVERITY_PROJECT}" -o /coverity_tool.tgz https://scan.coverity.com/download/linux64 && \
  mkdir /coverity_tool && \
  tar xf /coverity_tool.tgz --strip 1 -C /coverity_tool

ENV PATH="${PATH}:/coverity_tool/bin"
