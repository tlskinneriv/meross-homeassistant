ARG BUILD_FROM=homeassistant/amd64-base-debian
FROM ${BUILD_FROM}

# Build arguments
ARG BUILD_ARCH
ARG BUILD_DATE
ARG BUILD_REF
ARG BUILD_VERSION

# Versions
ENV HIVEMQ='b2043e7fcbd5897a3799869d42c6202122221fbc' \
    LIBWEBSOCKETS='3.1.0' \
    MOSQUITTO='2.0.10' \
    MOSQUITTO_GO_AUTH='1.5.0' \
    MOSQUTTO_DISCONNECT='0.0.1'

# Set shell
SHELL ["/bin/bash", "-o", "pipefail", "-c"]

# Set workdir
WORKDIR /usr/src

# Copy the requirements.txt file
COPY rootfs/opt/custom_broker/requirements.txt requirements.txt

# Setup base
# hadolint ignore=DL3003
RUN apt-get update \
  && apt-get install --no-install-recommends -y gnupg gpg-agent \
  && apt-get --no-install-recommends install -y \
    avahi-utils \
    build-essential \
    git \
    cmake \
    libssl-dev \
    libffi-dev \
    python3-dev \
    libavahi-client-dev \
    libavahi-compat-libdnssd-dev \
    avahi-daemon \
    dbus \
    bash \
    sqlite \
    curl \
    socat \
    openssl \
    python3 \
    python3-setuptools \
    python3-pip \
    nginx \
    software-properties-common \
    xsltproc \
    logrotate \
  \
  && pip3 install wheel \
  && mkdir -p /tmp/go \
  && \
    if [ "${BUILD_ARCH}" = "armhf" ]; then \
    export GOOS=linux ; export GOARCH=arm ; \
    elif [ "${BUILD_ARCH}" = "armv7" ]; then \
    export GOOS=linux ; export GOARM=7 ; export GOARCH=arm ; \
    elif [ "${BUILD_ARCH}" = "aarch64" ]; then \
    export GOOS=linux ; export GOARCH=arm64 ; \
    elif [ "${BUILD_ARCH}" = "i386" ]; then \
    export GOOS=linux ; export GOARCH=386 ; \
    elif [ "${BUILD_ARCH}" = "amd64" ]; then \
    export GOOS=linux ; export GOARCH=amd64 ; \
    else \
    echo 'NOT VALID BUILD'; exit 1; \
    fi \
  && curl https://dl.google.com/go/go1.16.3.${GOOS}-${GOARCH}.tar.gz --output /tmp/go/go.tar.gz \
  && tar -C /usr/local -xzf /tmp/go/go.tar.gz \
  && export PATH=$PATH:/usr/local/go/bin \
  \
  && pip3 install -r requirements.txt \
  \
  && git clone --branch "v${LIBWEBSOCKETS}" --depth=1 \
    https://github.com/warmcat/libwebsockets.git /tmp/libwebsockets \
  \
  && mkdir -p /tmp/libwebsockets/build \
  && cd /tmp/libwebsockets/build \
  && cmake .. \
    -DCMAKE_BUILD_TYPE=MinSizeRel \
    -DCMAKE_INSTALL_PREFIX=/usr \
    -DLWS_IPV6=OFF \
    -DLWS_WITHOUT_BUILTIN_GETIFADDRS=ON \
    -DLWS_WITHOUT_CLIENT=ON \
    -DLWS_WITHOUT_EXTENSIONS=ON \
    -DLWS_WITHOUT_TESTAPPS=ON \
    -DLWS_WITH_SHARED=OFF \
    -DLWS_WITH_ZIP_FOPS=OFF \
    -DLWS_WITH_ZLIB=OFF \
  && make \
  && make install \
  \
  && git clone --branch "v${MOSQUITTO}" --depth=1 \
      https://github.com/eclipse/mosquitto.git /tmp/mosquitto \
  && git clone --branch "v${MOSQUTTO_DISCONNECT}" --depth=1 \
      https://github.com/albertogeniola/mosquitto-disconnections-plugin.git /tmp/mosquitto-disconnections-plugin \
  && mkdir -p /tmp/mosquitto/plugins/disconnections \
  && cp -R /tmp/mosquitto-disconnections-plugin/src/* /tmp/mosquitto/plugins/disconnections \
  && echo "add_subdirectory(disconnections)" >> /tmp/mosquitto/plugins/CMakeLists.txt \
  && cd /tmp/mosquitto \
  && cmake . \
    -D WITH_ADNS=OFF \
    -D WITH_DOCS=OFF \
    -D WITH_MEMORY_TRACKING=OFF \
    -D WITH_TLS_PSK=OFF \
    -D WITH_WEBSOCKETS=ON \
    -D WITH_CJSON=OFF \
    -D WITH_CLIENTS=OFF \
    -D WITH_BROKER=ON \
    -D WITH_PLUGINS=ON \
    -D DOCUMENTATION=OFF \
  && make install \
  && mkdir -p /usr/share/mosquitto \
  && cp /usr/local/lib/mosquitto_disconnections.so /usr/share/mosquitto/mosquitto_disconnections.so \
  \
  && cd /tmp \
  && git clone --depth 1 --branch "${MOSQUITTO_GO_AUTH}" \
        https://github.com/iegomez/mosquitto-go-auth.git \
  && cd mosquitto-go-auth \
  && make \
  && cp /tmp/mosquitto-go-auth/pw /usr/share/mosquitto/pw \
  && cp /tmp/mosquitto-go-auth/go-auth.so /usr/share/mosquitto/go-auth.so \
  && chmod +x /usr/share/mosquitto/pw \
  \
  && addgroup --system mosquitto \
  && adduser --system --no-create-home --disabled-login --group mosquitto \
  \
  && apt-get clean \
  && rm -fr \
    /etc/nginx \
    /opt/mosquitto.conf \
    /opt/acl \
    /tmp/*

WORKDIR /
COPY rootfs /
RUN chmod +x /usr/local/bin/*.sh

LABEL io.hass.version="VERSION" io.hass.type="addon" io.hass.arch="armhf|aarch64|i386|amd64"
