FROM ubuntu:24.04

ENV DEBIAN_FRONTEND=noninteractive \
    TZ=Asia/Shanghai \
    LANG=C.UTF-8

RUN apt-get update && apt-get install -y --no-install-recommends \
    build-essential \
    g++ \
    clang \
    clang-format \
    clang-tidy \
    lldb \
    gdb \
    cmake \
    ninja-build \
    make \
    pkg-config \
    git \
    curl \
    wget \
    ca-certificates \
    lsb-release \
    gnupg \
    vim \
    less \
    unzip \
    python3 \
    python3-pip \
    libboost-all-dev \
    libssl-dev \
    libcurl4-openssl-dev \
    zlib1g-dev \
    libzstd-dev \
    liblz4-dev \
    libsnappy-dev \
    libbz2-dev \
    libfmt-dev \
    libspdlog-dev \
    libgtest-dev \
    libgmock-dev \
    libbenchmark-dev \
    libprotobuf-dev \
    protobuf-compiler \
    nlohmann-json3-dev \
    libtbb-dev \
    libzmq3-dev \
    cppzmq-dev \
    clangd \
    && rm -rf /var/lib/apt/lists/*

# Apache Arrow / Parquet
RUN wget -O /tmp/arrow-keyring.deb \
    https://packages.apache.org/artifactory/arrow/$(lsb_release --id --short | tr 'A-Z' 'a-z')/apache-arrow-apt-source-latest-$(lsb_release --codename --short).deb \
    && apt-get install -y /tmp/arrow-keyring.deb \
    && rm /tmp/arrow-keyring.deb \
    && apt-get update \
    && apt-get install -y --no-install-recommends \
    libarrow-dev \
    libarrow-dataset-dev \
    libarrow-acero-dev \
    libarrow-flight-dev \
    libparquet-dev \
    && rm -rf /var/lib/apt/lists/*

WORKDIR /work

CMD ["/bin/bash"]
