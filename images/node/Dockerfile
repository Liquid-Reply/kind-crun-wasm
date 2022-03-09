FROM ubuntu:21.10 AS builder
WORKDIR /data
RUN DEBIAN_FRONTEND=noninteractive apt update \
    && DEBIAN_FRONTEND=noninteractive apt install -y curl make git gcc build-essential pkgconf libtool libsystemd-dev libprotobuf-c-dev libcap-dev libseccomp-dev libyajl-dev go-md2man libtool autoconf python3 automake \
    && curl https://raw.githubusercontent.com/WasmEdge/WasmEdge/master/utils/install.sh | bash -s -- -p /usr/local --version=0.9.1 \
    && git clone --single-branch --branch main https://github.com/containers/crun.git \
    && cd crun \
    && ./autogen.sh \
    && ./configure --with-wasmedge --enable-embedded-yajl\
    && make 

FROM kindest/node:v1.21.10

COPY config.toml /etc/containerd/config.toml
COPY --from=builder /data/crun/crun /usr/local/sbin/runc
COPY --from=builder /usr/local/lib/libwasmedge_c.so /usr/local/lib/libwasmedge_c.so

RUN echo "Installing Packages ..." \
    && bash -c 'cat <<< $(jq "del(.hooks.createContainer)" /etc/containerd/cri-base.json) > /etc/containerd/cri-base.json' \
    && ldconfig
