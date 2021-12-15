FROM kindest/node:v1.23.0

RUN echo "Installing Packages ..." \
    && DEBIAN_FRONTEND=noninteractive clean-install \
       libyajl-dev git patch \
    && curl https://raw.githubusercontent.com/WasmEdge/WasmEdge/master/utils/install.sh | bash -s -- -p /usr/local

COPY config.toml /etc/containerd/config.toml
COPY crun /usr/local/sbin/runc