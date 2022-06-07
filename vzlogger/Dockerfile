ARG BUILD_FROM=ghcr.io/home-assistant/amd64-base:3.15
# hadolint ignore=DL3006
FROM ${BUILD_FROM} as builder

ENV LANG C.UTF-8

# hadolint ignore=DL3008
RUN echo $(uname -m) && \
    apk update && apk upgrade &&  apk add \
    make \
    git \
    cmake \
    gcc \
    libc-dev \
    util-linux-dev \
    g++ \
    json-c-dev \
    curl-dev \
    mosquitto-dev \
    openssl-dev \
    gnutls-dev \
    libmicrohttpd-dev \
    libgcrypt-dev \
    autoconf \
    automake \
    libtool \
    libunistring-dev \
    cyrus-sasl-dev

WORKDIR /vzlogger
RUN git clone https://github.com/volkszaehler/libsml.git --depth 1 && \
    git clone https://github.com/rscada/libmbus.git --depth 1 && \
    git clone https://github.com/volkszaehler/vzlogger.git --depth 1

WORKDIR /vzlogger/libsml
RUN make install -C sml

WORKDIR /vzlogger/libmbus
RUN ./build.sh && make install

WORKDIR /vzlogger/vzlogger
RUN cmake -DBUILD_TEST=off && \
    make && make install

# hadolint ignore=DL3006
FROM $BUILD_FROM 

# Labels
LABEL \
    io.hass.name="${BUILD_NAME}" \
    io.hass.description="${BUILD_DESCRIPTION}" \
    io.hass.arch="${BUILD_ARCH}" \
    io.hass.type="addon" \
    io.hass.version=${BUILD_VERSION} \
    org.opencontainers.image.title="${BUILD_NAME}" \
    org.opencontainers.image.description="${BUILD_DESCRIPTION}" \
    org.opencontainers.image.licenses="GNU General Public License v3.0" \
    org.opencontainers.image.url="https://github.com/${BUILD_REPOSITORY}/blob/main/README.md" \
    org.opencontainers.image.source="https://github.com/${BUILD_REPOSITORY}" \
    org.opencontainers.image.documentation="https://github.com/${BUILD_REPOSITORY}/blob/main/README.md" \
    org.opencontainers.image.created=${BUILD_DATE} \
    org.opencontainers.image.revision=${BUILD_REF} \
    org.opencontainers.image.version=${BUILD_VERSION}

# libsml is linked statically => no need to copy
COPY --from=builder /usr/local/bin/vzlogger /usr/local/bin/vzlogger
COPY --from=builder /usr/local/lib/libmbus.so* /usr/local/lib/
COPY rootfs /

# hadolint ignore=DL3008
RUN apk update && apk upgrade && apk add \
    libunistring \
    mosquitto \
    cyrus-sasl \
    libmicrohttpd \
    libuuid \
    gcc \
    g++ \
    eudev

# without running a user context, no exec is possible and without the dialout group no access to usb ir reader possible
# RUN useradd -G dialout vz && mkdir -p /home/vz && chown vz:vz /home/vz

RUN chmod a+x /etc/services.d/vzlogger/listdevices.sh && \
    chmod a+x /etc/services.d/vzlogger/run

#RUN chmod a+x /etc/services.d/vzlogger/listdevices.sh && \
#    usermod -a -G dialout root
