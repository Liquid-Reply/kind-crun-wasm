FROM ubuntu:21.10 AS builder
WORKDIR /data
RUN DEBIAN_FRONTEND=noninteractive apt update \
    && DEBIAN_FRONTEND=noninteractive apt install -y curl make git gcc build-essential pkgconf libtool libsystemd-dev libprotobuf-c-dev libcap-dev libseccomp-dev libyajl-dev go-md2man libtool autoconf python3 automake \
    && curl https://raw.githubusercontent.com/WasmEdge/WasmEdge/master/utils/install.sh | bash -s -- -p /usr/local \
    && git clone --single-branch --branch feat/handler_per_container https://github.com/liquid-reply/crun \
    && cd crun \
    && ./autogen.sh \
    && ./configure --with-wasmedge \
    && make 

FROM kindest/node:v1.23.0

RUN echo "Installing Packages ..." \
    && DEBIAN_FRONTEND=noninteractive clean-install \
       libyajl-dev git patch gpg \
    && curl https://raw.githubusercontent.com/WasmEdge/WasmEdge/master/utils/install.sh | bash -s -- -p /usr/local \
    && bash -c 'cat <<< $(jq "del(.hooks.createContainer)" /etc/containerd/cri-base.json) > /etc/containerd/cri-base.json' \
    && echo 'deb http://download.opensuse.org/repositories/devel:/tools:/criu/xUbuntu_21.04/ /' > /etc/apt/sources.list.d/devel:tools:criu.list \
    && curl -fsSL https://download.opensuse.org/repositories/devel:tools:criu/xUbuntu_20.10/Release.key | gpg --dearmor > /etc/apt/trusted.gpg.d/devel_tools_criu.gpg \
    && DEBIAN_FRONTEND=noninteractive clean-install criu


COPY config.toml /etc/containerd/config.toml
COPY --from=builder /data/crun/crun /usr/local/sbin/runc