FROM ubuntu:22.04 AS builder
WORKDIR /data
RUN DEBIAN_FRONTEND=noninteractive apt update \
    && DEBIAN_FRONTEND=noninteractive apt install -y curl make git gcc build-essential pkgconf libtool libsystemd-dev libprotobuf-c-dev libcap-dev libseccomp-dev libyajl-dev go-md2man libtool autoconf python3 automake \
    && curl https://raw.githubusercontent.com/WasmEdge/WasmEdge/master/utils/install.sh | bash -s -- -p /usr/local --version=0.11.0 \
    && git clone --depth 1 --branch 1.6 https://github.com/containers/crun.git \
    && cd crun \
    && ./autogen.sh \
    && ./configure --with-wasmedge --enable-embedded-yajl\
    && make 

FROM kindest/node:v1.25.0

COPY config.toml /etc/containerd/config.toml
COPY --from=builder /data/crun/crun /usr/local/sbin/crun
COPY --from=builder /usr/local/lib/libwasmedge.so /usr/local/lib/libwasmedge.so
RUN ln -s /usr/local/lib/libwasmedge.so /usr/local/lib/libwasmedge.so.0 && \
    ln -s /usr/local/lib/libwasmedge.so /usr/local/lib/libwasmedge.so.0.0.0 && \
    ldconfig
